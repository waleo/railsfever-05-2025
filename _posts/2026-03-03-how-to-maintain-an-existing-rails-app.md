---
layout: post
title: "How to Maintain an Existing Rails App: A Practical Playbook"
subtitle: "The habits, tools, and rhythms that keep a production Rails app healthy year after year"
description: "A hands-on guide to maintaining an existing Rails app. Covers dependency management, monitoring, incremental upgrades, test health, and the weekly and monthly habits that prevent emergencies."
author: "Wale Olaleye"
categories: ["Rails Maintenance", "Developer Guide"]
tags: [maintain existing rails app, rails maintenance, dependency management, rails upgrades, technical debt]
featured_image: "/assets/images/blog/maintenance-800x600.webp"
preview_image: "/assets/images/blog/maintenance-400x300.webp"
---

Most Rails apps in production are not new. They are 3, 5, 8 years old. They carry scar tissue from the founder who wrote the first version, the agency that built v2, and the four engineers who passed through since. The question for those apps is not "how do we build this?" It is "how do we keep this alive, without losing a week every time we touch it?"

This is a playbook for maintaining an existing Rails app. Not greenfield advice. Not "rewrite in Hotwire." Practical habits that keep a real codebase shippable and upgradable.

If you are feeling the pain of an app that has been neglected for years, start with [what to do when your Rails app crashes in production](/2025/10/19/what-to-do-when-your-rails-app-crashes-in-production.html) first. This post assumes you are not currently on fire.

## What "Maintenance" Actually Means

Maintenance is not "keep the lights on." Keeping the lights on is operations. Maintenance is the deliberate, ongoing work that keeps the app:

- **Upgradable**: you can bump Rails, Ruby, and gems without a two-month project.
- **Auditable**: you know what is running, why, and who changed it.
- **Secure**: you know your CVE exposure and patch quickly.
- **Observable**: you find out about problems from your monitors, not your customers.
- **Changeable**: a new engineer can ship a feature this week.

If any one of those is broken, you are accumulating technical debt faster than you are paying it down. That debt has compound interest - see [the hidden ROI of regular Rails maintenance](/2025/10/13/hidden-roi-of-regular-rails-maintenance.html).

## The Four Layers of a Rails App You Have to Maintain

Most engineers think "Rails maintenance" and picture bumping `gem "rails"` in the Gemfile. That is one layer of four. Miss any of the others and the app decays.

### 1. The runtime

Ruby version, OS, base Docker image, Node / Yarn if you have JS.

### 2. The framework

Rails itself and its direct dependencies (`actionpack`, `activerecord`, etc.).

### 3. The gems

Everything in `Gemfile.lock` that is not Rails. This is usually where the real technical debt lives.

### 4. Your own code

The models, controllers, jobs, views, and initializers your team has written. Especially the initializers. Oh, the initializers.

A good maintenance program touches all four on a predictable rhythm.

## The Weekly, Monthly, Quarterly Rhythm

The single biggest upgrade for most Rails teams is moving from "we maintain reactively" to "we maintain on a schedule." Here is a default rhythm that works for small teams.

### Weekly (30-60 minutes)

- Read dependency alerts (Dependabot, Snyk, or `bundler-audit`).
- Triage any new CVEs: patch now, patch this month, ignore with justification.
- Check Sentry / error monitor for new error signatures.
- Skim production logs for 500s and slow queries.

### Monthly (half a day)

- Bump all patch-level gems.
- Bump Ruby patch versions if available.
- Update one or two minor-version gems.
- Run the full test suite and deploy to staging with the bumps.

### Quarterly (1-2 days)

- Bump a Rails minor version if one is available and you are behind.
- Run `bundle outdated --only-explicit` and pick 3-5 gems to move forward meaningfully.
- Spike the next Rails major upgrade, even if you do not ship it.
- Review initializers and middleware - anything stale or unused?

### Annually (1-2 weeks)

- Complete at least one Rails minor or major upgrade.
- Refresh test coverage around critical flows.
- Re-evaluate every pinned-with-no-comment gem version.
- Retire dead code that nobody has touched in 18+ months.

That is it. Apps that do this never call me in a panic. Apps that do not, eventually do.

## Dependency Management: The Core Skill

Dependency management is the single most important maintenance skill in Rails. It is also the most neglected.

### Know what you have

```bash
bundle outdated --only-explicit
bundle exec bundle-audit check --update
```

The first command shows you what is out of date among gems you explicitly declared. The second flags known CVEs. Both should run in CI. Neither should scream at your team - if they do, fix the signal before you fix the bug.

### Upgrade in small, boring steps

The pattern that works:

```bash
git checkout -b upgrade/redis-5
# bump Gemfile
bundle update redis
bundle exec rspec
# if green, ship. if not, debug this one gem.
```

One gem per branch. One branch per PR. If the test suite fails you know exactly who to blame. This is unglamorous and far faster than "let's upgrade all the gems" marathons.

### Kill dead gems

Every quarter, look at `Gemfile` and ask "does anything in the app still use this?"

```bash
grep -r "GemClass" app lib spec
```

Remove unused gems aggressively. Every gem is a future CVE and a future upgrade friction point.

### Pin deliberately, comment always

```ruby
# Pinned to 1.12.x because 1.13 drops Ruby 3.0 support.
# Unpin when we are on Ruby 3.2+.
gem "some_gem", "~> 1.12.0"
```

Pins without comments are booby traps for the next engineer.

## Test Health Is Maintenance Health

You cannot maintain an app you cannot test. Most upgrade pain is test pain wearing a costume.

A few baseline requirements for a maintainable Rails app:

- CI runs the full test suite on every PR.
- The test suite finishes in under 15 minutes (ideally under 5).
- The test suite is deterministic. Flaky tests get fixed or deleted, not retried.
- Critical user flows have end-to-end coverage (system specs or equivalent).
- New features come with tests. No exceptions.

If your tests are red, skipped, or flaky, maintenance becomes guesswork. Fix the tests first. Everything downstream gets easier.

## Observability: Find Out Before Customers Do

A maintainable Rails app has:

- **Error monitoring** (Sentry, Honeybadger, Rollbar). You should know about a new exception signature within minutes.
- **Performance monitoring** (Skylight, Scout, New Relic, Datadog). You should see slow endpoints, slow queries, and memory growth.
- **Structured logs** that are actually searchable.
- **Alerting that routes to a human** for critical incidents.

This is not optional. The cost of *not* having this is your next production outage.

See [what to do when your Rails app crashes in production](/2025/10/19/what-to-do-when-your-rails-app-crashes-in-production.html) for the incident side.

## The Initializer Graveyard

Every old Rails app has an `config/initializers` directory full of files nobody has opened in years. `monkey_patches.rb`. `custom_marshalling.rb`. `rack_patches.rb`. Each one is a silent veto on every future upgrade.

On a quarterly cadence, walk through `config/initializers` one file at a time and ask:

- Why is this here?
- Is it still needed?
- Can it be deleted now that Rails / the gem has upstreamed the fix?
- If it stays, is there a comment explaining why?

Deleting an initializer is one of the highest-leverage maintenance actions in Rails. Each one you remove makes future upgrades measurably easier.

## Migrations and the Schema

A healthy Rails app:

- Has a `schema.rb` (or `structure.sql`) that matches the database.
- Can rebuild the dev database from scratch in under a minute.
- Runs migrations reversibly or, for destructive changes, with an explicit rollback plan.
- Has no "temporary" columns from 2019.

Audit the schema once a year. Drop unused columns, indexes, and tables. Big schemas slow down everything - migrations, specs, data backups.

## Security Maintenance

Some baseline habits that catch 80% of real Rails security incidents:

- `bundler-audit` in CI, not just locally.
- Rails, Ruby, and Nokogiri on their latest supported releases.
- Brakeman in CI for static analysis.
- Credentials in `config/credentials.yml.enc` or a secret manager, not ENV files in Git.
- Force SSL, strict CSP, secure cookie flags in production.

If you handle customer data you should also read [security best practices for web apps](/2025/08/20/security-best-practices-web-apps-lessons-coderabbit-exploit.html).

## When Maintenance Cannot Fix It Alone

Some apps have decayed past the point where weekly patching will save them. Signs:

- You are more than two Rails majors behind.
- Ruby is past EOL.
- Critical gems have no upgrade path on your version.
- The test suite is mostly skipped or broken.
- Deploys require a specific person's laptop.

In those cases, maintenance is not the answer. A [Rails rescue](/services/rails_rescue_kit/) or structured [upgrade project](/services/rails_upgrade_express/) is. Once you are stable, then you go back to maintenance rhythms.

## The Team Cost of Not Maintaining

The real cost of skipping maintenance is not the eventual upgrade bill. It is the gradual slowdown in everything else. Features take longer because the test suite is scary. Bugs take longer to fix because nobody trusts deploys. New engineers ramp up slower because nothing is documented. Every quarter the app gets a little harder to change.

Apps on regular maintenance do not have that slope. They ship faster in year five than apps that "saved time" by skipping it in year two.

---

Need help setting up a sustainable maintenance rhythm for your Rails app? That is exactly what our [Rails Care Plan](/services/rails_care_plan/) delivers - monthly patching, incremental upgrades, and a team that actually knows your codebase. If you are starting from a bad place, a [Rails technical audit](/services/rails_tech_audit/) gives you an honest baseline first.

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to talk through your Rails maintenance plan.
