---
layout: post
title: "What Ruby Version Does Rails 8 Actually Require?"
subtitle: "The precise minimum, the recommended version, and why the answer you find on the internet is often one version off"
description: "The official Ruby version requirements for Rails 8 — minimum and recommended — with links to the authoritative Rails sources and a practical recommendation for which Ruby to actually run."
author: "Wale Olaleye"
categories: ["Rails Upgrades", "Developer Guide"]
tags: [rails 8, ruby version rails 8, ruby 3.2, ruby 3.3, rails 8 requirements]
---

Every time a new Rails major ships, the same question churns through blog posts, Stack Overflow answers, and internal Slack channels: what is the minimum Ruby version? And the internet, being the internet, produces several confident but contradictory answers.

For Rails 8, this is the short version:

- **Minimum Ruby version: 3.2.0**
- **Recommended Ruby version: 3.3 or later**

The rest of this post shows where those numbers come from, why the Rails 7.2 requirement often gets confused with Rails 8, and what you should actually install on your server today.

## Where the Minimum Comes From

The authoritative source for a Rails version's minimum Ruby is the `rails.gemspec` file in the corresponding stable branch. For Rails 8.0, that file specifies:

```ruby
s.required_ruby_version = ">= 3.2.0"
```

This is the constraint that `bundle install` will enforce. If your system Ruby is older than 3.2.0, Rails 8 will refuse to install. Full stop.

The [official Rails upgrade guide](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html) says the same thing: "Rails 8.0 and 8.1 require Ruby 3.2.0 or newer."

## Why Ruby 3.1 Keeps Getting Cited

There is a widely circulated claim that Rails 8 requires Ruby 3.1. This claim is wrong, but it has a plausible origin.

The [Rails PR that bumped the minimum Ruby to 3.1](https://github.com/rails/rails/pull/50491) landed in late 2023 and was titled "Bump the required Ruby version to 3.1.0." That PR was merged before Rails 8 released and became the basis for Rails 7.2's minimum — **not** Rails 8's.

Between Rails 7.2 and Rails 8, the Rails team raised the minimum again to 3.2. So:

- **Rails 6.1** requires Ruby 2.5+
- **Rails 7.0 and 7.1** require Ruby 2.7+
- **Rails 7.2** requires Ruby 3.1+
- **Rails 8.0 and 8.1** require Ruby 3.2+

If you see a source confidently saying Rails 8 runs on Ruby 3.1, it is almost always a reader who found PR 50491, read "Rails" and "3.1" and "required Ruby version," and drew the wrong conclusion. The gemspec is the canonical source. It says 3.2.

## Where the Recommended Version Comes From

Minimum and recommended are different things. The minimum is what Rails will tolerate. The recommended is what Rails works *best* on.

The [Rails issue tracking the Ruby recommendation for Rails 8](https://github.com/rails/rails/issues/50447) captures the core team's reasoning: push adoption forward by defaulting new applications to Ruby 3.3 (and later, Ruby 3.4), and drop internal support cruft for older versions where possible.

In practice, this means:

- **`rails new` in Rails 8** scaffolds a Gemfile with Ruby 3.3 or newer.
- **Generated Dockerfiles** assume a current Ruby.
- **Official benchmarks and guides** use Ruby 3.3+ as the reference.

If you install Ruby 3.2 to satisfy the minimum, Rails 8 will work. But you will be running on a Ruby that Rails considers the floor, not the target. Performance, YJIT improvements, and new standard-library features all favor the newer Ruby.

## What You Should Actually Install

For a new Rails 8 application, install the latest stable Ruby. At the time of writing, that means **Ruby 3.4.x**, which is the latest stable release and the one the Rails team is actively optimizing against.

For an existing Rails 8 application that needs to be upgraded, the order of operations is:

1. Upgrade Ruby first to at least 3.2, ideally 3.3 or 3.4.
2. Run your full test suite on the new Ruby, on the current Rails version, for at least a week in staging.
3. Then upgrade Rails.

Skipping step 1 — trying to upgrade Ruby and Rails simultaneously — is the number-one source of upgrade stalls. Each change has its own failure modes. Stacking them multiplies the debugging surface.

## The Ruby Release Cadence and What It Means

Ruby ships a new minor version every December. Each version gets roughly three years of maintenance, then moves to security-only mode, then reaches end-of-life. In practical terms:

- **Ruby 3.2**: End-of-life approximately March 2026. If you are on 3.2 today, plan to move.
- **Ruby 3.3**: Maintained into early 2027. Safe choice for a new production deploy.
- **Ruby 3.4**: Latest stable. First choice for new applications.

Rails 8 supports all three, but the cost of being on the lowest version climbs over time. Today's "minimum" is next year's "deprecated." A Ruby that is comfortable today is at end-of-life in 18 months.

For a fuller treatment of Ruby versioning from a business perspective, see our post on [understanding Ruby versioning for founders](/2026/04/07/understanding-ruby-versioning-for-founders.html).

## The Rails-Ruby Compatibility Table

For reference, here is the compatibility matrix for recent Rails versions, pulled from the [Rails upgrade guides](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html) and the FastRuby compatibility table:

| Rails Version | Minimum Ruby | Recommended Ruby | Status              |
|---------------|--------------|------------------|---------------------|
| Rails 6.0     | 2.5.0        | 2.6.x            | End-of-life         |
| Rails 6.1     | 2.5.0        | 3.0.x            | End-of-life         |
| Rails 7.0     | 2.7.0        | 3.3.x            | End-of-life         |
| Rails 7.1     | 2.7.0        | 3.4.x            | End-of-life         |
| Rails 7.2     | 3.1.0        | 3.4.x            | Security only       |
| Rails 8.0     | 3.2.0        | 3.4.x            | Active              |
| Rails 8.1     | 3.2.0        | 3.4.x            | Active              |

The pattern is consistent: each minor bumps the Ruby floor, and the "recommended" tracks whatever the latest stable Ruby is at release time.

## Practical Recommendation

If you are deploying Rails 8 today and you do not have a specific constraint pushing you lower:

- Install Ruby 3.4.x.
- Pin it in `.ruby-version`.
- Add `ruby "3.4.x"` to your Gemfile.
- Match the version in your Dockerfile, CI config, and any other environment that runs Ruby.

This gives you the smoothest Rails 8 experience today, the longest runway before the next forced Ruby upgrade, and avoids every "minimum version is fine" conversation that tends to come back as a problem a year later.

---

Planning a Rails 8 deployment or a Ruby upgrade ahead of one? Our [Rails Upgrade Express](/services/rails_upgrade_express/) service handles both the Ruby and Rails sides in a single engagement, so the version alignment is correct the first time. For ongoing Ruby and Rails currency, see the [Rails Care Plan](/services/rails_care_plan/).

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to plan your Rails 8 upgrade.
