---
layout: post
title: "Upgrading From Rails 6 to Rails 8: The Full Guide"
subtitle: "The complete roadmap for jumping four Rails versions — the intermediate stops, the Ruby bumps, and the specific landmines at each stage"
description: "A step-by-step guide to upgrading a Ruby on Rails application from Rails 6 all the way to Rails 8. The intermediate version stops, Ruby requirements, gem compatibility, and the specific landmines at each major step."
author: "Wale Olaleye"
categories: ["Rails Upgrades", "Developer Guide"]
tags: [rails 6 to rails 8 upgrade, rails upgrade guide, rails 8, ruby on rails modernization, legacy rails]
image: "/assets/images/blog/upgrade-on-chalk-board-800x600.webp"
preview_image: "/assets/images/blog/upgrade-on-chalk-board-400x300.webp"
---

Rails 6 shipped in August 2019. Rails 8 shipped in late 2024. Between them sit four years of changes: Zeitwerk, Trilogy, Propshaft, Solid Cache, Solid Queue, the deprecation of Sprockets, shifts in default configuration, and three Ruby major-minor bumps.

You cannot jump Rails 6 to Rails 8 in one step. The official Rails upgrade guides are unambiguous: move one minor at a time. That is not advice you can skip — the internal deprecation warnings only fire at each version, and skipping a minor means silently carrying forward broken assumptions that will surface as weird runtime errors two releases later.

This guide lays out the full path, version by version, with the specific things to check at each step.

## The Path

From Rails 6.0 or 6.1, the route to Rails 8 goes:

**6.0 → 6.1 → 7.0 → 7.1 → 7.2 → 8.0**

That is five upgrade steps if you are starting on 6.0, four if you are starting on 6.1. Each step should be its own branch, its own PR, and its own deploy to production. Do not batch them.

Ruby version bumps are separate steps and should happen *between* Rails upgrades, not alongside them.

## Before You Start: Baseline Health

Nothing about this guide works if your current Rails 6 app is in bad shape. Before step one:

- `bundle exec rspec` (or `bin/rails test`) passes, no flakes, in CI.
- `bundle outdated` has been reviewed in the last month.
- `bundle exec bundle-audit check --update` is clean.
- You have a staging environment that mirrors production.
- Deploys are a single command with a working rollback path.
- You can spin up a new branch and run the test suite against a clean database in under five minutes.

If any of those are false, fix them first. The Rails 6-to-8 jump has enough surface area on its own; attempting it on a brittle base turns it into a multi-month disaster.

## Stage 0: Rails 6.0 to Rails 6.1

Skip this stage if you are already on 6.1.

Rails 6.1 is the easier half of the 6.x line. The key changes:

- **Zeitwerk is the default autoloader** in new apps but Rails 6.1 still supports Classic. Switch to Zeitwerk now if you have not — Rails 7.0 will force the issue. The dedicated [Zeitwerk transition guide](https://github.com/fxn/zeitwerk) covers the common fixes.
- **Strict loading associations** can surface N+1 queries earlier. Worth enabling in development.
- **Per-database switching** for multi-database setups was refined. If you use horizontal sharding, re-read the guide.

**Ruby requirement:** Ruby 2.5+, but you should be on Ruby 3.0 or newer before moving to Rails 7.

**The common landmines:**

- Classic autoloader files with naming that breaks under Zeitwerk (constants that do not match file paths).
- Enum behavior changes around `validates` on enums.
- Deprecations around `Rails.application.routes.default_url_options`.

Update `Gemfile`, run `bundle update rails`, run tests, fix what breaks, run `rails app:update`, review the config diff, deploy.

## Stage 1: Rails 6.1 to Rails 7.0

This is the largest single jump on the path and the one most likely to eat a week of developer time. Rails 7.0 introduced a major shift in frontend tooling and forced Zeitwerk adoption.

**Ruby requirement:** Ruby 2.7.0 minimum, but upgrade to Ruby 3.0 or 3.1 before this stage. Rails 7.0 on Ruby 2.7 is possible but awkward — many gems have moved on.

**The major surface areas:**

- **Asset pipeline rewrite.** Rails 7.0 introduces Propshaft as an alternative to Sprockets, and the default for new apps shifts away from Webpacker toward import maps or jsbundling-rails/cssbundling-rails. You do not have to migrate immediately — keep Sprockets and Webpacker if you want — but understand where the project is heading.
- **Zeitwerk is mandatory.** Classic autoloader support is removed. Any remaining classic-style files will break.
- **Encrypted attributes** become a framework feature, replacing some third-party gems.
- **Async query loading** is available for long-running queries in parallel.

**The common landmines:**

- Webpacker deprecation. Webpacker is officially retired in Rails 7. You can keep it working, but the clock is ticking.
- `ActiveRecord::Base.connection_config` is deprecated in favor of `connection_db_config`.
- Spring is removed from new Rails 7 apps. Keep it in your Gemfile if you were relying on it.
- Minor behavior changes in `delegate_missing_to` and `attr_accessor` in controllers.

Run `rails app:update` carefully. Diff the generated changes against your existing configs. Reject the asset pipeline changes if you are not ready to migrate Webpacker or Sprockets yet.

Expect this stage to take 2-5 engineer-days on a mid-sized app.

## Stage 2: Rails 7.0 to Rails 7.1

Rails 7.1 is a polish release. Most apps make this jump in a day or two.

**Ruby requirement:** Ruby 2.7+, but 3.1 or later is strongly recommended.

**The major surface areas:**

- **Default config bumps.** Several configs that were deprecation-warning only in 7.0 get stricter defaults in 7.1.
- **Composite primary keys** are finally supported in ActiveRecord.
- **Async query consolidation**, better Trilogy support.

**The common landmines:**

- `secrets.yml` handling — some edge cases changed. If you still use `Rails.application.secrets` anywhere, audit.
- Autoloader edge cases around namespaced constants in engines.
- `ActionController::Parameters#each` behavior tweak.

`rails app:update`, review config diffs, run tests, deploy. Short stage.

## Stage 3: Rails 7.1 to Rails 7.2

Rails 7.2 was specifically designed as the on-ramp to Rails 8. Almost everything that will land in 8 has a migration path in 7.2.

**Ruby requirement:** Ruby 3.1.0 minimum. If you are still on Ruby 3.0 here, upgrade Ruby first.

**The major surface areas:**

- **Trilogy becomes the default MySQL adapter** in generators (mysql2 still works).
- **Dev container support** shipped for Rails apps.
- **PWA manifest and service worker** scaffolded in new apps.
- **YJIT enabled by default** on supported Ruby versions.

**The common landmines:**

- If you have custom rack middleware, re-verify the middleware stack order.
- `require` statements in initializers that relied on autoloading semantics — Zeitwerk's 7.2 behavior is stricter.
- Production asset compilation flags — several defaults were tightened.

This is a short stage on most apps. A day or two.

**Critical checkpoint:** at the end of Stage 3, you are on Rails 7.2 with Ruby 3.1+. This is the known-good platform to launch Rails 8 from. Do not skip it even if you are tempted to fast-forward.

## Stage 4: Ruby Upgrade Before Rails 8

Rails 8 requires Ruby 3.2.0 minimum, and recommends Ruby 3.3 or newer. (See our post on [the minimum Ruby version for Rails 8](/blog/minimum-ruby-version-for-rails-8/) for the source citations.)

Before attempting the Rails 8 bump, upgrade Ruby:

- Target Ruby 3.3 or 3.4 — not just the minimum 3.2.
- Do this as its own PR, on Rails 7.2, with no framework changes.
- Run the full test suite, deploy to staging, let it bake at least a few days.

Skipping this — bumping Ruby and Rails in the same PR — is the single most common way to turn a boring upgrade into a bisecting nightmare.

## Stage 5: Rails 7.2 to Rails 8.0

If you have been disciplined through the previous stages, this is usually the shortest stage of the whole project.

**Ruby requirement:** Ruby 3.2.0 minimum, 3.3 or 3.4 recommended.

**The major surface areas:**

- **Solid Cache, Solid Queue, Solid Cable** replace Redis and Sidekiq in new apps (old apps can keep Redis-backed workers; Solid Queue is optional).
- **Kamal 2** is the default deploy tool in scaffolded Dockerfiles.
- **Authentication generator** ships in the framework.
- **Propshaft becomes the default asset pipeline** in new apps.
- **Rails 8 generators** produce slimmer, more opinionated apps.

**The common landmines:**

- If you still depend on Sprockets, pin it explicitly — it is no longer a default.
- If you use Devise or another auth gem, the new built-in `authenticate_by` helpers may conflict if you do not name things carefully.
- The Docker generator assumes Kamal; if you deploy to Heroku or another PaaS, override.
- Minor behavior changes in ActiveRecord transaction callbacks.

For the detailed step-by-step of this specific stage, see our [Rails 7.2 to Rails 8 upgrade guide](/blog/rails-7-2-to-rails-8-upgrade-step-by-step/).

## Gem Ecosystem: Audit at Every Stage

Major Rails versions break gems. At each stage, expect to:

- Update any gem with a Rails version dependency.
- Remove gems that have been absorbed into Rails (e.g., `byebug` after Ruby debug stdlib matures, `webpacker` after you migrate assets).
- Watch for gems that are no longer maintained. A gem with no commits since Rails 6 is a gem you should be nervous about.

Bundler's error messages during `bundle update rails` are usually informative. Read them carefully rather than reflexively running `bundle update --all` to "fix" them.

## Test Strategy Across the Upgrade

A four-stage Rails upgrade is a test-suite-intensive project. A few patterns that pay off:

- **Run the full test suite at every stage.** Not just a sampler. Rails upgrades frequently surface existing bugs that were always there.
- **Watch for silenced deprecations.** Search the codebase for `ActiveSupport::Deprecation.silence` or `ActiveSupport::Deprecation.behavior = :silence`. Each silenced deprecation is a landmine at the next stage.
- **Run in both development and production modes in CI.** Several Rails issues only appear with `config.eager_load = true`.
- **Exercise background jobs.** Run at least one end-to-end test per job class.

## Expected Timeline

On a mid-sized Rails 6 app (50-100 models, a few thousand specs, moderate gem complexity):

- Stage 0 (6.0 → 6.1): 1-3 days
- Stage 1 (6.1 → 7.0): 3-7 days (Zeitwerk, asset pipeline)
- Stage 2 (7.0 → 7.1): 1-2 days
- Stage 3 (7.1 → 7.2): 1-2 days
- Stage 4 (Ruby upgrade to 3.3/3.4): 1-2 days
- Stage 5 (7.2 → 8.0): 1-3 days

Total: 2-4 engineer-weeks of focused work, plus staging bake time between each stage. Spread over a couple months with normal feature work, that is a quarter.

Apps with unusual gem stacks, heavy custom middleware, or hand-rolled autoloading can easily double that. Apps with excellent test coverage and clean conventions can finish faster.

## When to Bring in Help

Rails 6 to Rails 8 is a real project. Teams that succeed at it usually have one of:

- A dedicated engineer with prior Rails upgrade experience running point for the duration.
- A consultancy that has done this upgrade many times and knows the landmines in advance.
- A lot of calendar time and appetite to learn as you go.

If none of those are true, the upgrade tends to stall out at Stage 1 or Stage 2, the app sits in a half-upgraded state on a branch, and the project quietly dies. We have seen this many times.

---

Looking at a Rails 6 to Rails 8 upgrade and want a partner who has done it repeatedly? Our [Rails Upgrade Express](/services/rails_upgrade_express/) handles the full path on a fixed timeline, including the intermediate Ruby bumps. For apps that need a structural assessment first, the [Rails Tech Audit](/services/rails_tech_audit/) produces a stage-by-stage plan you can execute internally or hand to us. See also our [Rails 7.2 to Rails 8 step-by-step guide](/blog/rails-7-2-to-rails-8-upgrade-step-by-step/) for the last leg of the journey.

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to plan your Rails 6 to Rails 8 upgrade.
