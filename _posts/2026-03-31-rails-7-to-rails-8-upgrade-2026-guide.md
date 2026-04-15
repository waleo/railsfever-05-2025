---
layout: post
title: "Rails 7 to Rails 8 Upgrade: The 2026 Technical Guide"
subtitle: "Scope, sequence, and the real gotchas you will hit - from someone who has run this migration across a lot of codebases"
description: "A practical, field-tested 2026 guide to the Rails 7 to Rails 8 upgrade. Covers scoping, sequencing, Solid Cache and Solid Queue, Propshaft, authentication, and the gotchas that actually break production."
author: "Wale Olaleye"
categories: ["Rails Upgrades", "Developer Guide"]
tags: [rails 7 to rails 8 upgrade, rails 8, propshaft, solid queue, solid cache, rails upgrade]
featured_image: "/assets/images/blog/upgrade-on-chalk-board-800x600.webp"
preview_image: "/assets/images/blog/upgrade-on-chalk-board-400x300.webp"
---

Rails 8 has been out long enough now that "we're planning our Rails 7 to Rails 8 upgrade" is no longer early-adopter talk. It is overdue for a lot of teams. If you're on Rails 7.0 or 7.1, your official support window is closing. If you're on Rails 7.2, you still have runway, but not for long.

This is a 2026 field guide to the Rails 7 to Rails 8 upgrade, written from the perspective of someone who has shepherded this migration through many codebases. Not a marketing brochure for Rails 8's new features - a practical walkthrough of what to plan, what to sequence, and what actually breaks.

If you want an even more specific focus on Rails 7.2 only, see [Rails 7.2 to Rails 8 upgrade: step-by-step](/2026/04/14/rails-7-2-to-rails-8-upgrade-step-by-step.html).

## Before You Touch Anything: Scope the Work

The single biggest mistake teams make on a Rails 7 to Rails 8 upgrade is treating it like a framework bump. It is not. It is a compound migration that touches the asset pipeline, caching, background jobs, and potentially authentication all at once.

Before writing any code, answer these honestly:

- **What Rails point release are we on?** 7.0, 7.1, and 7.2 each have a different distance to 8.
- **What Ruby version?** Rails 8 requires Ruby 3.2 at minimum. 3.3 or 3.4 strongly preferred.
- **Asset pipeline?** Sprockets, Propshaft, Webpacker, jsbundling-rails, shakapacker?
- **Background jobs?** Sidekiq, GoodJob, Delayed Job, Resque?
- **Cache store?** Redis, Memcache, Dalli, custom?
- **Authentication?** Devise, custom, Sorcery, OmniAuth-only?
- **Database?** Postgres (version?), MySQL, SQLite.
- **Deploy?** Heroku, Kamal, AWS, custom.

Write this down. Each answer constrains the upgrade path. An app on Rails 7.0 with Webpacker, Sidekiq, Devise, and Heroku is a fundamentally different project than Rails 7.2 with Propshaft, GoodJob, custom auth, and Kamal. Do not let anyone give you a timeline without knowing these answers.

## Sequence the Upgrade: Do Not Do It All at Once

The other mistake: treating "upgrade to Rails 8" as a single PR. It is three or four separate projects that should ship in order. Batching them multiplies the risk.

A good default sequence:

1. **Get Ruby current** (3.3.x or 3.4.x).
2. **Get to the latest Rails 7.2.x** if you are not there already.
3. **Clear upgrade blockers**: retire unused gems, update authentication if needed, migrate off Webpacker to jsbundling-rails or importmap.
4. **Bump to Rails 8.0**.
5. **Adopt Rails 8 features deliberately** (Solid Cache, Solid Queue, built-in auth) only after you're stable on 8.

Each of these 1-5 items is ideally a separate deploy, possibly with weeks between them.

## Step 1: Get Ruby Current

Rails 8 requires Ruby 3.2+. Ruby 3.0 and 3.1 are EOL. If you are on Ruby 3.0 or earlier, this is your first milestone - and it is its own project.

```bash
# .ruby-version
3.3.6
```

```bash
# Dockerfile (if applicable)
FROM ruby:3.3.6-slim
```

Then:

```bash
bundle install
bundle exec rspec
```

Common breakage points moving to Ruby 3.2 / 3.3:

- Gems that called `.exists?` on `nil` or relied on frozen-string quirks.
- `Data.define` collisions if you have a model or class named `Data`.
- Keyword argument strictness - if you didn't finish the 2.7 → 3.0 transition, you'll catch it now.

Ship Ruby as a separate deploy. Let it bake in production for a week before touching Rails.

## Step 2: Move to Latest Rails 7.2.x

Whatever Rails 7 point release you are on, move to the latest 7.2.x first. Do **not** jump from 7.0 straight to 8.

```ruby
# Gemfile
gem "rails", "~> 7.2.2"
```

```bash
bundle update rails
bundle exec rails app:update
```

`rails app:update` will walk you through changed configs interactively. Read every diff. Accept, reject, or merge with care. Don't blindly accept - some of these defaults will bite you.

Watch for:

- `config/initializers/new_framework_defaults_*.rb` files.
- `config.load_defaults` value in `config/application.rb`.
- Changes to `config/puma.rb` if you haven't updated it in years.

Ship Rails 7.2 to production. Let it bake for at least a week.

## Step 3: Clear the Rails 8 Blockers

Some changes are easier to make on Rails 7 before the 8 upgrade than during it. Tackle these as independent PRs:

### Move off Webpacker

Webpacker is retired. If your app still uses it, migrate to `jsbundling-rails` with esbuild or rollup, or move to `importmap-rails` if your JS is modest. This is a meaningful project on its own - do not pretend it's part of the Rails upgrade.

### Decide Sprockets vs Propshaft

Rails 8 defaults to Propshaft, but Sprockets still works. If your Sprockets setup is complex (custom transforms, Sass via sprockets, image fingerprinting assumptions), stay on Sprockets through the Rails 8 upgrade and migrate to Propshaft as a separate project later. Do **not** do both at once.

### Review authentication

Rails 8 ships a built-in authentication generator. You do not have to use it. Devise still works on Rails 8. Do not rip out working auth to adopt a new framework feature in the same upgrade - that combination is a test suite nightmare.

### Clean up your Gemfile

```bash
bundle outdated --only-explicit
bundle exec bundle-audit check --update
```

Delete gems you don't use. Update gems with known CVEs. Upgrade any gem that has a known-to-block-Rails-8 older version.

## Step 4: Actually Upgrade to Rails 8

Now the bump itself. This should be the smallest PR in the whole project if you sequenced well.

```ruby
# Gemfile
gem "rails", "~> 8.0.0"
```

```bash
bundle update rails
bundle exec rails app:update
bundle exec rspec
```

Work through `rails app:update` deliberately. Expect to touch:

- `config/application.rb` - `load_defaults 8.0`.
- `config/environments/*.rb` - new Rails 8 defaults in the framework_defaults initializer.
- `config/cache.yml` - if adopting Solid Cache.
- `config/queue.yml` - if adopting Solid Queue.
- `config/puma.rb` - default may have shifted.

### The Solid Cache / Solid Queue decision

Rails 8 introduces Solid Cache (DB-backed caching) and Solid Queue (DB-backed jobs). These are optional. You do not have to adopt them during the upgrade.

- If you have a working Redis + Sidekiq stack, keep it. Adopt Solid* later as a separate project if at all.
- If you are over-paying for Redis for caching alone, Solid Cache is a real cost saver.
- If you are on Heroku and tired of Redis add-on complexity, Solid Queue is attractive.

Adopt Solid Cache and Solid Queue only after you are stable on Rails 8 vanilla.

### Database compatibility

Rails 8 drops support for older database versions. Verify:

- Postgres 13+ (14+ strongly recommended)
- MySQL 8.0.19+
- SQLite 3.8+

If your production DB is older, you have a separate infrastructure project before Rails 8 will run.

## The Real Gotchas

Here are the ones that keep showing up across migrations, beyond the obvious:

### Deprecated `ActiveRecord::Base.connection`

In Rails 8 the multi-db API has evolved. Calls to `ActiveRecord::Base.connection` in places that should be `.lease_connection` or the per-role handler can raise. Audit any explicit `.connection` use in `lib/` and `app/`.

### Zeitwerk strictness

Rails 8 is stricter about constant autoloading. If you relied on implicit autoload paths or weird file naming, you will find out during boot. Test `Rails.application.eager_load!` locally.

### `find_or_create_by` race conditions

Rails 8 hardens `find_or_create_by` / `find_or_initialize_by` behavior around unique constraints. Review any high-throughput `find_or_create_by` calls for subtle semantic changes.

### Initializer ordering

Rails 8 shifts a handful of framework initializer orderings. If you have initializers that read or mutate framework state during boot, verify they still run when you expect.

### ActionMailer `deliver_later` with Solid Queue

If you adopt Solid Queue, your mail delivery moves to the DB queue by default. If your transactional emails depend on Sidekiq retries or a specific dead-letter behavior, you need to reconfigure.

### JS build differences

If you moved off Webpacker, verify that every page's JS actually loads in production. Importmap-based apps in particular can look fine in dev and break in prod over CDN caching.

### Cookie / session compatibility

Rails 8 changed some default session cookie settings. If you run multiple services behind a shared domain or share cookies across subdomains, test the login flow end-to-end.

## Testing Strategy

The test suite is your upgrade's co-pilot. A few things to do:

```bash
bundle exec rspec --format documentation
```

- Run the entire suite multiple times to catch nondeterminism introduced by new defaults.
- Re-run system specs explicitly - browser-based specs catch the class of issues unit tests miss.
- Run your full suite with `RAILS_ENV=production` in staging, not just `test`.

A staging environment that truly mirrors production is not optional for this upgrade.

## Deployment

Do not deploy Rails 8 to production on a Friday. Do deploy it behind whatever rollout mechanism you have:

- Percentage rollouts if you have them.
- Canary deploys to one instance first.
- Feature-flagged new behavior (e.g., a trial of Solid Cache on a read-only surface) before full adoption.

Have a rollback plan. For a Rails major upgrade, "rollback" means redeploying the previous image with the previous Gemfile.lock - not a `git revert`. Make sure your deploy tool can do that under stress.

## Post-Upgrade: Adopt New Features Deliberately

Once you are stable on Rails 8, then evaluate:

- **Solid Cache**: significant cost savings if you were paying for Redis just for caching. Migrate gradually.
- **Solid Queue**: attractive if you want to drop Redis entirely; test throughput honestly on your actual workload before committing.
- **Built-in auth**: for new apps, compelling. For existing apps on Devise, rarely worth the swap.
- **Kamal 2**: if you are already on Kamal, the 2.x upgrade is straightforward. If you are on Heroku and happy, Kamal is a whole separate project.

None of these belong in the "upgrade to Rails 8" PR.

## Timeline Expectations

Rough numbers for a well-tested SaaS app:

- Well-maintained Rails 7.2 app: 2-4 weeks elapsed, 1-2 weeks of focused work.
- Rails 7.1 with modest debt: 4-8 weeks.
- Rails 7.0 with Webpacker and dated gems: 8-16 weeks, and the Webpacker migration is half of it.

If anyone quotes you under a week for the full project, they are underscoping. If anyone quotes over 6 months, they are either overscoping or telling you that you have a bigger problem than a Rails upgrade.

## Related Reading

- [Rails 7.2 to Rails 8 upgrade: step-by-step](/2026/04/14/rails-7-2-to-rails-8-upgrade-step-by-step.html)
- [When to upgrade Rails](/2025/10/20/when-to-upgrade-rails.html)
- [Stop wasting money on one-off Rails upgrades](/2025/11/02/stop-wasting-money-one-off-rails-upgrades.html)
- [How to maintain an existing Rails app](/2026/03/03/how-to-maintain-an-existing-rails-app.html)

---

Planning a Rails 7 to Rails 8 upgrade and want an honest scope? We offer [Rails Upgrade Express](/services/rails_upgrade_express/) for incremental Rails 7 to 8 migrations, [Rails technical audits](/services/rails_tech_audit/) to scope the work realistically before you commit, and [Rails Care Plans](/services/rails_care_plan/) so the next upgrade is routine instead of heroic.

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to talk through your Rails 8 upgrade.
