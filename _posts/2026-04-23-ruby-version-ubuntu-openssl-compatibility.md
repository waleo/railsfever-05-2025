---
layout: post
title: "Ruby Version and Ubuntu Compatibility: It's All About OpenSSL"
subtitle: "Why older Rubies fail to build on new Ubuntu releases, which combinations actually work, and what to install when"
description: "A practical compatibility matrix for Ruby versions and Ubuntu releases, centered on the OpenSSL dependency that determines whether a given Ruby will actually install and run."
author: "Wale Olaleye"
categories: ["Rails Maintenance", "Developer Guide"]
tags: [ruby ubuntu compatibility, ruby openssl, ruby install ubuntu, ruby 2.7 ubuntu 22, ruby version matrix]
---

Every Rails team hits this eventually. You provision a new Ubuntu server. You try to install Ruby — usually an older version, because that is what the app uses. It fails somewhere deep in the build with a cryptic OpenSSL error. An hour later you are deep in StackOverflow reading about `OPENSSL_ROOT_DIR`, libssl1.1 PPAs, and people recommending you just "switch to Docker."

The underlying issue is almost always the same: **the Ruby you are trying to install does not support the OpenSSL that shipped with the Ubuntu you are trying to install it on.** Once you understand the compatibility matrix, the fix is usually straightforward.

This post lays out the matrix and the practical decision tree.

## Why OpenSSL Is the Pivot

Ruby links against OpenSSL at build time for its `openssl`, `net/https`, `digest`, and related standard library modules. Every Ruby release supports a specific range of OpenSSL versions — and OpenSSL has had two major transitions in the relevant window: the move from 1.0.x to 1.1.x, and the move from 1.1.x to 3.0.x.

Ubuntu ships whichever OpenSSL was current when the Ubuntu release was cut. If you install a Ruby that predates that OpenSSL version, the build either fails or produces a Ruby that cannot make HTTPS connections.

The Ubuntu maintainers do not update OpenSSL versions within a release. So a Ruby that worked on Ubuntu 20.04 may not build on Ubuntu 22.04 without extra work.

## The Ruby / OpenSSL Matrix

Per the [Ruby on Mac OpenSSL compatibility reference](https://www.rubyonmac.dev/openssl-versions-supported-by-ruby):

| Ruby Version | OpenSSL Required | Notes                                  |
|--------------|------------------|----------------------------------------|
| 4.0.x        | >= 1.1           | Prefers 3.0+                           |
| 3.4.x        | >= 1.1           | Prefers 3.0+                           |
| 3.3.x        | >= 1.1           | Prefers 3.0+                           |
| 3.2.x        | >= 1.1           | Prefers 3.0+ (end-of-life)             |
| 3.1.x        | >= 1.1           | Prefers 3.0+ (end-of-life)             |
| 3.0.x        | >= 1.1, < 3.0    | End-of-life                            |
| 2.7.x        | >= 1.1, < 3.0    | End-of-life                            |
| 2.6.x        | >= 1.1, < 3.0    | End-of-life                            |
| 2.5.x        | >= 1.1, < 3.0    | End-of-life                            |
| 2.4.x        | >= 1.1, < 3.0    | End-of-life                            |
| 2.3.x        | >= 1.0, < 1.1    | End-of-life                            |
| 2.2.x        | >= 1.0, < 1.1    | End-of-life                            |

The critical breakpoint: **Ruby versions older than 3.1 do not support OpenSSL 3.0.** They were released before OpenSSL 3.0 existed, and the C API changes were not backported.

## The Ubuntu / OpenSSL Matrix

And here is what each Ubuntu LTS ships with by default:

| Ubuntu Release         | OpenSSL Default    |
|------------------------|--------------------|
| Ubuntu 18.04 (Bionic)  | 1.1.1              |
| Ubuntu 20.04 (Focal)   | 1.1.1              |
| Ubuntu 22.04 (Jammy)   | 3.0.2              |
| Ubuntu 24.04 (Noble)   | 3.0.13             |

The big jump is between 20.04 and 22.04 — that is where OpenSSL moved from 1.1.x to 3.0.x. That jump is the source of most "my Ruby build just broke" support tickets.

## The Combined Compatibility Table

Putting the two together, here is what actually works out of the box on each Ubuntu release:

| Ubuntu Release | Works Cleanly                    | Requires Extra Work             | Will Not Work         |
|----------------|----------------------------------|---------------------------------|-----------------------|
| 18.04          | Ruby 2.4 – 3.4                   | (Ubuntu is EOL; see notes)      | Ruby 2.3 and older    |
| 20.04          | Ruby 2.4 – 3.4                   | Ruby 4.0 (may need newer OpenSSL) | Ruby 2.3 and older  |
| 22.04          | Ruby 3.1 – 4.0                   | Ruby 2.4 – 3.0 (needs OpenSSL 1.1) | Ruby 2.3 and older |
| 24.04          | Ruby 3.1 – 4.0                   | Ruby 2.4 – 3.0 (needs OpenSSL 1.1) | Ruby 2.3 and older |

The pattern: if you are on Ubuntu 22.04 or 24.04 and your app uses Ruby 3.0 or older, installing it is not a native-apt-packages job. You need to either install OpenSSL 1.1 alongside the system OpenSSL 3, or containerize.

## The "Works Cleanly" Path

If your Ruby and Ubuntu are in the green zone, the install just works:

```bash
# On Ubuntu 22.04 or 24.04, installing Ruby 3.3
sudo apt-get install -y build-essential libssl-dev libffi-dev \
  libyaml-dev libreadline-dev zlib1g-dev libgmp-dev \
  libncurses5-dev libgdbm-dev

# Using rbenv:
rbenv install 3.3.9
rbenv global 3.3.9
```

That path is boring and reliable when the versions align.

## The "Requires Extra Work" Path: Old Ruby on New Ubuntu

This is the common pain case: your Rails app is on Ruby 2.7 or 3.0, and your ops team is migrating to Ubuntu 22.04 or 24.04. The Ruby build will fail on OpenSSL.

Three paths, in order of preference:

### Option A: Upgrade Ruby

If you can, this is the right move. Getting onto Ruby 3.1+ removes the OpenSSL problem entirely and unblocks future Rails upgrades. For most apps, the Ruby jump from 2.7 or 3.0 to 3.1+ is modest — harder than a patch, easier than a Rails minor.

If your Rails version supports it (Rails 6.1+ runs on Ruby 3.1), upgrade Ruby first, *then* provision the new Ubuntu box. See our [Rails 6 to Rails 8 upgrade guide](/2026/04/22/rails-6-to-rails-8-upgrade-guide.html) for the surrounding context.

### Option B: Build Ruby Against OpenSSL 1.1

If you genuinely cannot upgrade Ruby in the short term, install OpenSSL 1.1 on the Ubuntu box and build Ruby against it.

```bash
# Install OpenSSL 1.1 from a trusted source (not the default apt repo on 22.04/24.04).
# One common approach: build from source.
cd /tmp
wget https://www.openssl.org/source/openssl-1.1.1w.tar.gz
tar xzf openssl-1.1.1w.tar.gz
cd openssl-1.1.1w
./config --prefix=/opt/openssl-1.1 --openssldir=/opt/openssl-1.1
make -j$(nproc)
sudo make install

# Then build Ruby against it:
RUBY_CONFIGURE_OPTS="--with-openssl-dir=/opt/openssl-1.1" \
  rbenv install 2.7.8
```

Caveat: **OpenSSL 1.1 reached end-of-life in September 2023.** You are running crypto that is no longer receiving security patches. This is acceptable as a short-term bridge during an upgrade; it is not a long-term production posture.

### Option C: Containerize

Run the old Ruby in a Docker container based on an older Ubuntu (20.04) or a Ruby-official base image. The host Ubuntu's OpenSSL version becomes irrelevant inside the container.

This is the pragmatic answer for teams with infrastructure already leaning on containers. The drawback is that you have not actually solved the Ruby-end-of-life problem — you have just insulated it from the host OS.

## Ubuntu 18.04 Is EOL; Don't Build New Infrastructure There

Ubuntu 18.04 LTS reached end of standard support in May 2023 and extended security maintenance in April 2028. Do not provision new servers on it. If you have old infrastructure still on 18.04 and your Rails app only runs there because of Ruby/OpenSSL alignment, you are carrying compounding debt on both axes. Plan a joint Ruby + Ubuntu upgrade rather than extending the life of either.

## The Right Target Today

For new Ubuntu infrastructure running Rails in 2026:

- **Ubuntu 24.04 LTS** — supported through 2029.
- **Ruby 3.3 or 3.4** — matches OpenSSL 3.0+ cleanly and is supported through 2027-2028.
- **Rails 7.2 or 8.0** — both work on this pairing.

That combination installs with no special flags, receives security updates from all three upstreams, and has runway measured in years, not months.

## A Quick Diagnostic Checklist

When a Ruby install fails with a confusing error, work through this list:

1. `lsb_release -a` — which Ubuntu?
2. `openssl version` — which OpenSSL is installed on the host?
3. Which Ruby version are you trying to install?
4. Cross-reference against the tables above. Is your Ruby compatible with that OpenSSL?

If the answer is no, you have found the issue. The fix is one of the three options above.

## Further Reading

- [OpenSSL versions supported by Ruby](https://www.rubyonmac.dev/openssl-versions-supported-by-ruby) — the authoritative matrix
- [Understanding Ruby versioning for founders](/2026/04/07/understanding-ruby-versioning-for-founders.html) — the business-side view of Ruby versions
- [Minimum Ruby version for Rails 8](/2026/04/20/minimum-ruby-version-for-rails-8.html) — the forward-looking Ruby target

---

Struggling to install an old Ruby on a new Ubuntu, or planning infrastructure changes that bump into Ruby version constraints? Our [Rails Care Plan](/services/rails_care_plan/) and [Rails Upgrade Express](/services/rails_upgrade_express/) services regularly handle these Ruby-and-OS alignment projects — so the server migration and Ruby upgrade happen once, together, correctly.

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to plan your Ruby and OS upgrade.
