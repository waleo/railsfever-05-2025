---
layout: post
title: "Deploying Rails 8 with Kamal on a Single Server"
subtitle: "A senior engineer's guide to hardening a fresh VPS and shipping a production Rails 8 app with Kamal, kamal-proxy, and Let's Encrypt"
description: "How to deploy a Rails 8 app with Kamal on a single server. Server hardening script, config/deploy.yml, kamal-proxy SSL, host Postgres vs accessories, the Docker-bridge networking, first deploy, and ongoing operations."
slug: "deploying-rails-8-with-kamal-single-server"
author: "Wale Olaleye"
categories: ["Developer Guide", "Rails Deployment"]
tags: [rails 8 deployment, kamal, kamal-proxy, single server rails, vps hardening, docker, lets encrypt, devops for rails]
image: "/assets/images/blog/scrabble-tiles-spelling-launch-800x600.webp"
preview_image: "/assets/images/blog/scrabble-tiles-spelling-launch-400x300.webp"
---

A single server will take a Rails 8 app a long way. One reasonably-sized VPS — a few dedicated vCPUs and 8–16 GB of RAM from any commodity host — comfortably runs the web app, a background worker, Postgres, and Redis for a product doing real revenue. You do not need Kubernetes, a managed control plane, or a five-service cloud architecture to serve thousands of users. You need a box, a hardening pass, and a deploy tool that does zero-downtime releases without ceremony.

That deploy tool now ships in the framework. Rails 8 includes [Kamal](https://kamal-deploy.org/) out of the box, and Kamal 2 brought `kamal-proxy` — its own reverse proxy with automatic Let's Encrypt certificates — so a single-server deploy no longer needs Traefik, nginx, or a separate load balancer in front of it.

This guide is the path I actually use: harden the server first, then layer Kamal on top. It is platform-agnostic — everything here works the same whether your box is on Hetzner, Linode, DigitalOcean, Vultr, or bare metal in a closet. The only assumption is a fresh **Ubuntu 24.04 LTS** server you can SSH into as root.

## The Shape of the Setup

Before any commands, here is the target architecture on the single host:

| Layer | What runs it | Exposed to the internet? |
|-------|--------------|--------------------------|
| TLS termination + routing | `kamal-proxy` (container) | Yes — ports 80/443 |
| Rails web | Puma in your app container | No — proxy routes to it |
| Background jobs | Solid Queue / Sidekiq container | No |
| Database | Postgres on the host | No — Docker bridge only |
| Cache / queue backend | Redis on the host | No — Docker bridge only |

The app and proxy run as Docker containers managed by Kamal. Postgres and Redis run as **host services** reachable only over the Docker bridge network. Nothing but 22, 80, and 443 is reachable from outside.

## Step 1: Harden the Server First

The single biggest mistake I see is deploying the app first and securing the box "later." A fresh VPS with a public IP is being scanned for weak SSH credentials within minutes. Do the hardening pass before anything else, and make it a **script** so it is repeatable and reviewable — not a sequence of ad-hoc SSH commands you can never reproduce.

What a good bootstrap script does, in order:

1. Update the system and install core packages
2. Create an unprivileged `deploy` user with SSH-key access
3. Install Docker (Kamal needs it on the host)
4. Configure host Postgres and Redis to listen on the Docker bridge
5. Harden SSH (no root login, no passwords)
6. Lock down the firewall
7. Add fail2ban and automatic security upgrades
8. Raise file descriptor limits

Run it as root on the fresh box: `bash setup-server.sh`. Below are the pieces that matter, adapted from a script I run on production single-server deploys. Make it idempotent — you will re-run it. The fragments in this section are assembled into a complete, runnable script in [this GitHub gist](https://gist.github.com/waleo/c649651cccabb1b8d060567857f1e2ca) — read it before running it, and adjust the config block at the top for your app.

> **Tip:** You do not have to copy the script onto the server first. Pipe it over SSH and run it in one shot, keeping the file version-controlled in your repo:
>
> ```bash
> ssh deploy@<server-ip> 'sudo bash -s' < bin/setup-server.sh
> ```
>
> On the very first run of a brand-new box the `deploy` user does not exist yet, so connect as root that one time — `ssh root@<server-ip> 'bash -s' < bin/setup-server.sh` — then use the `deploy` form for every re-run after that.

### Packages and the deploy user

Start with a strict shell and install the essentials:

```bash
#!/usr/bin/env bash
set -euo pipefail

export DEBIAN_FRONTEND=noninteractive
apt-get update -qq
apt-get upgrade -y -qq

apt-get install -y -qq \
  fail2ban ufw unattended-upgrades \
  curl git htop unzip \
  postgresql postgresql-contrib libpq-dev \
  redis-server logrotate
```

Create a `deploy` user with passwordless sudo and copy root's SSH keys to it. Kamal connects as this user, never as root:

```bash
if ! id deploy &>/dev/null; then
  useradd --create-home --shell /bin/bash deploy
fi
usermod -aG sudo deploy

echo "deploy ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/deploy
chmod 440 /etc/sudoers.d/deploy

# Reuse the key you already used to reach root
install -d -m 700 -o deploy -g deploy /home/deploy/.ssh
cp /root/.ssh/authorized_keys /home/deploy/.ssh/authorized_keys
chown deploy:deploy /home/deploy/.ssh/authorized_keys
chmod 600 /home/deploy/.ssh/authorized_keys
```

### Install Docker

Install Docker CE from the official repository and add `deploy` to the `docker` group so Kamal can drive it without sudo:

```bash
apt-get install -y -qq ca-certificates gnupg
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  > /etc/apt/sources.list.d/docker.list
apt-get update -qq
apt-get install -y -qq docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

usermod -aG docker deploy
systemctl enable --now docker
```

One non-obvious detail: give the Docker daemon an explicit public DNS resolver. Without it, containers sometimes cannot resolve external hosts — and `kamal-proxy` needs to reach Let's Encrypt to issue your TLS certificate:

```bash
mkdir -p /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
EOF
systemctl restart docker
```

### Harden SSH

Use a drop-in file rather than `sed`-editing the main `sshd_config` — it is cleaner and survives package upgrades. Disable root login and password auth entirely:

```bash
cat > /etc/ssh/sshd_config.d/99-hardening.conf <<EOF
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
EOF

# Validate BEFORE restarting — a bad config can lock you out
if sshd -t; then
  systemctl restart ssh
else
  echo "sshd config invalid, not restarting" >&2
  exit 1
fi
```

> **Keep your current session open** until you have confirmed, from a second terminal, that you can `ssh deploy@your-server-ip`. Locking yourself out of a fresh box is an annoying way to start.

### Firewall, fail2ban, and automatic upgrades

Default-deny everything inbound, then allow only SSH and HTTP(S):

```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw --force enable
```

fail2ban to throttle SSH brute-force attempts:

```bash
cat > /etc/fail2ban/jail.local <<EOF
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5

[sshd]
enabled = true
port = ssh
backend = systemd
EOF
systemctl enable --now fail2ban
```

Security-only unattended upgrades so the kernel and OpenSSL stay patched without you babysitting them:

```bash
cat > /etc/apt/apt.conf.d/20auto-upgrades <<EOF
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
EOF
```

Finally, raise the open-file limit for the `deploy` user — Puma plus background workers plus DB connections will blow past the default 1024:

```bash
sed -i '/^deploy\s\+\(soft\|hard\)\s\+nofile/d' /etc/security/limits.conf
echo "deploy soft nofile 65536" >> /etc/security/limits.conf
echo "deploy hard nofile 65536" >> /etc/security/limits.conf
```

That is the whole security baseline: an unprivileged deploy user, key-only SSH, a default-deny firewall, brute-force protection, and automatic patching. If you want a second set of eyes on a setup like this before it faces the internet, that is exactly what a [Rails technical audit](/services/rails_tech_audit/) covers.

## Step 2: Decide — Host Services or Kamal Accessories

Kamal can run your database and Redis as **accessories** (containers it manages), or you can run them as **host services** (installed directly on the box). For a single server, I default to host services for the stateful pieces, because:

- Postgres on the host uses the OS package, gets security updates through `unattended-upgrades`, and its data lives in a normal directory you can back up with `pg_dump` and a cron job.
- There is no container restart that can disrupt a database that should essentially never move.

The cost is that you wire up the networking yourself. The app container has to reach Postgres and Redis on the host, and it does that across the **Docker bridge** (`172.17.0.1` by default).

### Wiring host Postgres to the Docker bridge

Make Postgres listen on the bridge interface in addition to localhost, and allow password auth from Docker's subnet:

```bash
PG_CONF="/etc/postgresql/18/main/postgresql.conf"
PG_HBA="/etc/postgresql/18/main/pg_hba.conf"

sed -i "s/^#\?listen_addresses.*/listen_addresses = 'localhost,172.17.0.1'/" "$PG_CONF"

# Allow connections from any Docker network (covers the default bridge and Kamal's)
grep -q "172.16.0.0/12" "$PG_HBA" || \
  echo "host  all  all  172.16.0.0/12  scram-sha-256" >> "$PG_HBA"

systemctl restart postgresql
```

Redis needs the same treatment — bind to the bridge and, because UFW already blocks external access, turn off protected mode so bridge connections are accepted:

```bash
REDIS_CONF="/etc/redis/redis.conf"
sed -i 's/^bind .*/bind 127.0.0.1 ::1 172.17.0.1/' "$REDIS_CONF"
sed -i 's/^protected-mode yes/protected-mode no/' "$REDIS_CONF"
sed -i 's/^appendonly no/appendonly yes/' "$REDIS_CONF"
systemctl restart redis-server
```

Then — and this is the step people forget — explicitly allow the Docker subnet through UFW to reach those ports. Otherwise the firewall silently drops your app's database connections:

```bash
ufw allow from 172.16.0.0/12 to any port 5432
ufw allow from 172.16.0.0/12 to any port 6379
ufw reload
```

Your app then connects with a `DATABASE_URL` pointed at the bridge IP:

```
DATABASE_URL=postgres://myapp:PASSWORD@172.17.0.1:5432/myapp_production
REDIS_URL=redis://172.17.0.1:6379/0
```

### When to use an accessory instead

If you prefer Postgres-in-a-container — for parity with development, or to keep the host minimal — Kamal accessories are clean. The critical detail is the port binding: **publish to `127.0.0.1`, never `0.0.0.0`**, or you have just exposed your database to the internet (more on why in Pitfalls).

```yaml
accessories:
  db:
    image: postgres:18
    host: 203.0.113.10
    port: "127.0.0.1:5432:5432"   # bound to loopback, not the public IP
    env:
      clear:
        POSTGRES_DB: myapp_production
        POSTGRES_USER: myapp
      secret:
        - POSTGRES_PASSWORD
    directories:
      - data:/var/lib/postgresql/data
```

Either approach is valid. Host services trade a little setup for operational simplicity; accessories trade a little exposure risk for dev/prod parity. Pick one and be deliberate about it.

## Step 3: Configure Kamal

Rails 8 generates `config/deploy.yml` and `.kamal/secrets` for you. Here is a complete single-server config:

```yaml
# config/deploy.yml
service: myapp
image: myuser/myapp

servers:
  web:
    - 203.0.113.10
  job:
    hosts:
      - 203.0.113.10
    cmd: bin/jobs

proxy:
  ssl: true
  host: app.example.com
  app_port: 3000
  healthcheck:
    path: /up
    interval: 3

registry:
  server: ghcr.io
  username: myuser
  password:
    - KAMAL_REGISTRY_PASSWORD

builder:
  arch: amd64

env:
  clear:
    RAILS_ENV: production
    SOLID_QUEUE_IN_PUMA: false
  secret:
    - RAILS_MASTER_KEY
    - DATABASE_URL
    - REDIS_URL

aliases:
  console: app exec --interactive --reuse "bin/rails console"
  shell:   app exec --interactive --reuse "bash"
  logs:    app logs -f
  dbc:     app exec --interactive --reuse "bin/rails dbconsole"
```

A few things worth calling out:

- **`proxy.ssl: true` with a `host`** is all it takes for `kamal-proxy` to request and renew a Let's Encrypt certificate. Point your domain's A record at the server IP first, or the ACME challenge fails.
- **`healthcheck.path: /up`** uses Rails 8's built-in health endpoint. Kamal will not route traffic to a new container until it returns 200, which is what makes deploys zero-downtime.
- **`builder.arch: amd64`** matters if you develop on an Apple Silicon Mac. Without it, Kamal builds an arm64 image your amd64 server cannot run.

### Choosing a Container Registry

Kamal builds your image locally (or in CI), pushes it to a registry, then pulls it down on the server. You need a registry both ends can reach — but you do **not** need a paid plan. The config above uses GitHub's, which is the option I reach for first:

| Registry | `server:` value | Private images | Cost |
|----------|-----------------|----------------|------|
| GitHub Container Registry (GHCR) | `ghcr.io` | Yes | **Free** — unlimited public, and private images are free for personal accounts and included in GitHub plans |
| Docker Hub | *(omit — it's the default)* | 1 free private repo | Free tier with pull rate limits |
| GitLab Container Registry | `registry.gitlab.com` | Yes | Free with a GitLab repo |
| Self-hosted (`registry:2`) | your host/IP | Yes | Free, but it's one more thing to run and secure |

**GitHub Container Registry (`ghcr.io`) is the pragmatic default for most teams**, especially if your code already lives on GitHub. It is free for both public and private images, has no Docker Hub-style pull rate limits, and ties access to credentials you already manage. Authenticate with a [personal access token](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry) (classic) that has the `write:packages` scope — that token becomes your `KAMAL_REGISTRY_PASSWORD`:

```yaml
registry:
  server: ghcr.io
  username: myuser
  password:
    - KAMAL_REGISTRY_PASSWORD   # a PAT with write:packages scope
```

For **Docker Hub**, drop the `server:` line entirely (it's Kamal's default) and use your Docker Hub username plus an access token. Be aware of the anonymous/free-tier pull rate limits — on a single server that re-pulls on every deploy, a busy day can bump into them. For an **air-gapped or fully self-owned** setup, you can run a `registry:2` container, but for a single-server app that is usually more operational surface than it saves.

Whichever you pick, the image name in `deploy.yml` must match: `myuser/myapp` for GHCR/Docker Hub, or `your-registry-host:5000/myapp` for a self-hosted registry.

### Secrets

Secrets are resolved from `.kamal/secrets`, which should pull from your environment or a secrets manager — never commit real values:

```bash
# .kamal/secrets
KAMAL_REGISTRY_PASSWORD=$KAMAL_REGISTRY_PASSWORD
RAILS_MASTER_KEY=$(cat config/master.key)
DATABASE_URL=$DATABASE_URL
REDIS_URL=$REDIS_URL
```

In CI, set those as environment variables (GitHub Actions secrets, for example). Locally, a tool like `1password` CLI or a git-ignored `.env` feeds them in. The point is that `deploy.yml` is safe to commit and `secrets` only references names.

## Step 4: First Deploy

With the server hardened and `deploy.yml` in place, the first deploy is two commands. `kamal setup` installs `kamal-proxy`, logs in to your registry, builds and pushes the image, and boots the app:

```bash
kamal setup
```

Once it is up, prepare the database. Because Postgres lives on the host, run migrations through the app container:

```bash
kamal app exec "bin/rails db:prepare"
```

Every subsequent release is just:

```bash
kamal deploy
```

Kamal builds the image, pushes it, boots a new container alongside the old one, waits for `/up` to pass, switches `kamal-proxy` to the new container, and retires the old one. No downtime, no manual proxy juggling.

### A sane first-deploy checklist

- [ ] DNS A record for `app.example.com` points at the server IP
- [ ] `ssh deploy@server` works with your key (root login already disabled)
- [ ] `DATABASE_URL` / `REDIS_URL` point at `172.17.0.1` (host services) or the accessory
- [ ] `RAILS_MASTER_KEY` is available to `.kamal/secrets`
- [ ] `builder.arch` matches your server's CPU architecture
- [ ] `/up` returns 200 locally before you rely on it as a healthcheck
- [ ] You can reach `https://app.example.com` and the certificate is valid

## Step 5: Backups and Ongoing Operations

A deploy is not done until the data is backed up. For host Postgres, a nightly `pg_dump` piped to object storage via cron is enough for most single-server apps:

```bash
# /home/deploy/bin/backup-db.sh (sketch)
set -euo pipefail
STAMP=$(date -u +%Y%m%d-%H%M%S)
pg_dump myapp_production | gzip > "/tmp/myapp-${STAMP}.sql.gz"
# upload to your object store of choice, then clean up
aws s3 cp "/tmp/myapp-${STAMP}.sql.gz" "s3://your-bucket/db/" --endpoint-url "$S3_ENDPOINT"
rm -f "/tmp/myapp-${STAMP}.sql.gz"
```

```cron
0 2 * * * /home/deploy/bin/backup-db.sh 2>> /var/log/db-backup.log
```

Pair it with a `logrotate` rule so the log does not grow forever, and — importantly — **test a restore** at least once. A backup you have never restored is a hypothesis, not a backup.

Day-to-day, Kamal gives you the operations you need without SSHing into the box:

| Task | Command |
|------|---------|
| Tail logs | `kamal app logs -f` |
| Open a console | `kamal console` (the alias above) |
| Run a one-off task | `kamal app exec "bin/rails some:task"` |
| Roll back to previous release | `kamal rollback` |
| Restart the app | `kamal app boot` |
| Restart the proxy | `kamal proxy reboot` |

`kamal rollback` is the one to remember. If a deploy ships a bad release, you are one command from the previous image, because Kamal keeps it around.

## Common Pitfalls

A short list of the things that actually bite people on single-server Kamal deploys:

### Docker punches through UFW

This is the big one. Docker writes its own `iptables` rules, and **published container ports bypass UFW**. Your `ufw default deny incoming` does *not* protect a container port published to `0.0.0.0`. This is precisely why an accessory database must bind to `127.0.0.1:5432:5432` and not `0.0.0.0`. `kamal-proxy` publishing 80/443 is fine — you want those open — but never assume UFW is shielding a published port.

### Forgetting the Docker subnet firewall rule

If host Postgres listens on the bridge but you never run `ufw allow from 172.16.0.0/12 to any port 5432`, the app container's connections are silently dropped and you will chase a "database unreachable" ghost.

### Architecture mismatch

Building on Apple Silicon and deploying to an amd64 VPS without `builder.arch: amd64` produces an image that will not boot. The error is rarely obvious.

### Let's Encrypt can't validate

If DNS is not pointed at the server yet, or the Docker daemon has no DNS resolver, the ACME challenge fails and you get no certificate. Point DNS first; set `daemon.json` DNS during hardening.

### Treating the master key as optional

No `RAILS_MASTER_KEY` in the container means encrypted credentials do not decrypt, and the app boots into a confusing half-broken state. Make sure it flows through `.kamal/secrets`.

## When a Single Server Is Enough — and When It Isn't

A hardened single server with Kamal is the right answer for the overwhelming majority of Rails SaaS apps: predictable cost, simple mental model, real zero-downtime deploys. It scales vertically a long way, and when you do outgrow it, Kamal already supports multiple hosts — you add IPs to the `servers` block and put a load balancer in front.

The time to rethink is when you need high availability that survives a single box dying, multi-region latency, or a managed database with automated failover. Until then, resist the pull toward complexity you do not yet need.

If you would rather hand the whole pipeline — server hardening, Kamal config, backups, and monitoring — to a team that does this every week, that is the core of our [Rails Care Plans](/services/rails_care_plan/). And if your app is still on an older Rails version, getting to Rails 8 first is what [Rails Upgrade Express](/services/rails_upgrade_express/) is for.

## Frequently Asked Questions

### Do I need Kamal, or can I just use Capistrano or a PaaS?

Kamal targets a different model than Capistrano: it deploys your app as a Docker container rather than running code directly on the host, which gives you reproducible images and trivial rollbacks. Compared to a PaaS like Heroku or Render, Kamal on your own VPS is dramatically cheaper at scale and keeps you off proprietary infrastructure — at the cost of owning server hardening and backups yourself. If you want zero ops and are happy paying for it, a PaaS is fine. If you have a senior engineer and want control plus low cost, Kamal on a single server is the sweet spot.

### Is a single server really enough for a production SaaS?

For the large majority of Rails apps, yes. One VPS with a few dedicated vCPUs and 8–16 GB of RAM handles the web app, a worker, Postgres, and Redis for thousands of users. You scale vertically (a bigger box) for a long time before you need horizontal scaling. The honest limiter is not throughput — it is availability. A single server means a single point of failure, so the question to ask is whether you can tolerate occasional minutes of downtime, not whether the box is fast enough.

### What happens to my app during a deploy?

Nothing visible, if your healthcheck is set up. Kamal boots the new container alongside the old one, waits for `/up` to return 200, then switches `kamal-proxy` to the new container and retires the old one. In-flight requests finish on the old container. That is what makes `kamal deploy` zero-downtime.

### How do I roll back a bad release?

`kamal rollback`. Kamal keeps the previous image on the server, so rollback is near-instant — it just points the proxy back at the prior container. This is one of the strongest reasons to deploy containers rather than mutating a host in place.

### Should I run Postgres on the host or as a Kamal accessory?

On a single server, I default to **host Postgres**: it gets OS security updates automatically, its data lives in a normal directory you can `pg_dump`, and no container restart can ever disrupt it. Use an accessory if you want dev/prod parity or to keep the host minimal — just bind its port to `127.0.0.1`, never `0.0.0.0`, or you expose your database to the internet.

### Does the firewall actually protect my Docker containers?

Not automatically — this trips up almost everyone. Docker writes its own `iptables` rules, so a container port published to `0.0.0.0` **bypasses UFW entirely**. UFW protects host services (SSH, host Postgres on the bridge), but for containers you control exposure through the port binding. Publish internal services to loopback or the Docker bridge, and only let `kamal-proxy` bind 80/443 publicly.

### Which container registry should I use?

GitHub Container Registry (`ghcr.io`) is the free default I reach for — unlimited public images, free private images, and no Docker Hub-style pull rate limits. Docker Hub works too but watch its rate limits on a server that re-pulls each deploy. See the [registry section](#choosing-a-container-registry) above for the full comparison.

### How do I handle database migrations?

There are two common places to run migrations, and the choice matters more than it looks.

**Option 1 — in `bin/docker-entrypoint`.** The Rails 8 generated entrypoint can run `./bin/rails db:prepare` as each container boots. It is the simplest setup and fine for small apps. The catch: migrations now run *inside the container's startup*, and they count against the time Kamal waits for the healthcheck to pass. A long migration on a growing table can exceed the container-up timeout, Kamal marks the boot as failed, and your deploy aborts — sometimes with the migration half-applied.

**Option 2 — as a Kamal hook (recommended).** Run migrations from a `.kamal/hooks/pre-deploy` (or `post-deploy`) hook instead, decoupled from container boot:

```bash
#!/usr/bin/env bash
# .kamal/hooks/pre-deploy
set -euo pipefail
kamal app exec "bin/rails db:migrate"
```

This runs the migration once, in its own step, with no bearing on the healthcheck timeout — so it scales as your migrations get longer and slower. Either way, migrations run inside the deployed image, so they use the exact code being shipped. Use the entrypoint for first-deploy convenience (`db:prepare`), but move recurring migrations to a hook before they grow long enough to start timing out your deploys.

---
Need help with Rails maintenance? We offer comprehensive [Rails Care Plans](/services/rails_care_plan/) for ongoing support, [technical audits](/services/rails_tech_audit/) to assess your current state, and [Rails upgrades](/services/rails_upgrade_express/) to keep you current. View our [pricing plans](/pricing/) to find the right fit for your needs. [Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to discuss your Rails needs.
