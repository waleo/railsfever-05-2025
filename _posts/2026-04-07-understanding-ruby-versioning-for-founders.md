---
layout: post
title: "Understanding Ruby Versioning: A Simple Guide for CTOs and Founders"
subtitle: "What the version numbers mean, how long each version is supported, and what happens when support runs out"
description: "A founder-friendly guide to Ruby versioning and the Ruby release cycle. Understand what Ruby version numbers mean, how long each release is supported, and when you need to upgrade."
author: "Wale Olaleye"
categories: ["Rails Maintenance", "CTO Guide"]
tags: [ruby versioning, ruby release cycle, ruby end of life, ruby upgrade, cto guide]
---

If you run a product built on Ruby on Rails, your app depends on two major pieces of infrastructure: the Rails framework and the Ruby programming language. Most founders and CTOs pay attention to Rails versions but forget about Ruby — until something breaks or a security audit asks "what Ruby version are you running?" and nobody can answer confidently.

This post is a plain-language explanation of how Ruby versioning works, how long each version is supported, and what it means for your business when a version reaches end of life. It is based on the [official Ruby maintainer's guide to the release cycle](https://dev.to/hsbt/is-your-ruby-version-still-supported-a-maintainers-guide-to-rubys-release-cycle-799), simplified for a non-developer audience.

## How Ruby Version Numbers Work

Ruby uses a three-part version number: **MAJOR.MINOR.TEENY**.

- **MAJOR** (the first number) changes rarely — only when the language makes a significant leap. Ruby went from 2.x to 3.0 in 2020. Ruby 4.0 shipped in December 2025. These jumps happen years apart.
- **MINOR** (the second number) changes every year. A new Ruby minor version ships every December 25th (yes, Christmas — it is a tradition). Ruby 3.3 shipped December 2023, Ruby 3.4 shipped December 2024, Ruby 4.0 shipped December 2025.
- **TEENY** (the third number) changes every few months for bug fixes and security patches. These are the small, safe updates: 3.3.5 → 3.3.6 → 3.3.7.

The important thing to know: **TEENY updates are almost always safe to apply immediately.** MINOR and MAJOR updates require testing because they can change how the language behaves.

## How Long Is Each Version Supported?

Each Ruby minor version follows a roughly three-year lifecycle:

### Year 1-2: Normal Maintenance

The version receives bug fixes and security patches. This is the "fully supported" window. Everything works, gems are compatible, and you are in good shape.

### Year 3: Security Maintenance Only

The version stops getting bug fixes. It only receives patches for security vulnerabilities and critical build issues. Your app will still run, but you are on borrowed time — new gems may stop supporting this version, and the pool of fixes shrinks.

### After Year 3: End of Life

No more releases. No more security patches. Any new vulnerability discovered in your Ruby version will not be fixed. You are exposed.

## Where Things Stand Right Now

As of early 2026:

| Ruby Version | Released | Status | What It Means |
|---|---|---|---|
| Ruby 4.0 | Dec 2025 | Normal maintenance | Fully supported. Bug fixes and security patches. |
| Ruby 3.4 | Dec 2024 | Normal maintenance | Fully supported. Bug fixes and security patches. |
| Ruby 3.3 | Dec 2023 | Security maintenance | Security patches only. Plan your upgrade. |
| Ruby 3.2 | Dec 2022 | End of life | No patches. Upgrade now. |
| Ruby 3.1 and older | — | End of life | No patches. Upgrade urgently. |

If your app is on Ruby 3.2 or older, you are running on an unsupported runtime. Any new CVE will not be patched for your version.

## Why This Matters for Your Business

Ruby version is not just a developer concern. It has direct business implications:

### Security and compliance

Running an end-of-life Ruby version means unpatched vulnerabilities. If you handle customer data, payment information, or health records, this is a compliance risk. Security audits and SOC 2 assessments will flag it.

### Gem compatibility

The Ruby ecosystem moves forward. Gem authors drop support for older Ruby versions over time. The longer you stay on an old Ruby, the harder it becomes to update any of your dependencies — including Rails itself. Rails 8 requires Ruby 3.2 at minimum.

### Hiring and retention

Developers prefer working on current technology. An app stuck on Ruby 3.0 is a harder sell to candidates than one on 3.4 or 4.0. This affects both hiring speed and retention.

### Upgrade cost

Ruby upgrades are usually straightforward when done one minor version at a time (3.2 → 3.3 → 3.4). They become expensive when deferred across multiple versions. A three-version jump involves more breaking changes, more gem incompatibilities, and more testing.

## The One Rule That Saves Money

**Upgrade Ruby every year.**

Each December, a new Ruby minor ships. Sometime in Q1 of the following year, update your app to the new version. The jump from one minor to the next is usually small — a few days of testing, a handful of gem updates, and a deploy.

If you do this annually, you never fall behind. You never hit end of life. You never face a multi-version emergency upgrade.

If you skip it for three years, you are looking at a project, not a patch.

## How Ruby Upgrades Relate to Rails Upgrades

Ruby and Rails are separate projects with separate version numbers and separate support timelines. But they are tightly coupled:

- Each Rails version requires a minimum Ruby version.
- Rails 8 requires Ruby 3.2+.
- When you upgrade Rails, you often need to upgrade Ruby first.
- When Ruby hits end of life, you need to upgrade Ruby regardless of your Rails version.

The practical advice: keep both current. Upgrade Ruby annually (small, boring). Upgrade Rails on its own cadence (see [when to upgrade Rails](/2025/10/20/when-to-upgrade-rails.html)). Never let either fall more than one version behind.

## What to Do Right Now

1. **Find out what Ruby version your app is running.** Ask your developer, or check the `.ruby-version` file in your repository.
2. **Check it against the table above.** If it is 3.2 or older, you need to act this quarter.
3. **If you are on 3.3**, plan the upgrade to 3.4 within the next two quarters. You are in security-only maintenance.
4. **If you are on 3.4 or 4.0**, you are in good shape. Make sure you have a plan to stay current when the next version ships in December.
5. **Put "Ruby upgrade" on your maintenance calendar** as an annual Q1 task. Treat it like renewing insurance — boring, cheap, and catastrophically expensive to skip.

## Further Reading

- [Ruby maintainer's guide to the release cycle](https://dev.to/hsbt/is-your-ruby-version-still-supported-a-maintainers-guide-to-rubys-release-cycle-799) — the original technical source
- [Rails 7 LTS: A CTO's guide](/2026/02/10/rails-7-lts-long-term-support-guide-for-ctos.html) — what long-term support means on the Rails side
- [How to maintain Ruby on Rails: a founder's guide](/2026/03/17/maintain-ruby-on-rails-founder-guide.html) — the bigger maintenance picture

---

Not sure what Ruby version your app is on, or whether it is time to upgrade? A [Rails technical audit](/services/rails_tech_audit/) covers Ruby, Rails, and gem health in a single assessment. For ongoing Ruby and Rails maintenance, our [Rails Care Plan](/services/rails_care_plan/) keeps both current on a monthly cadence so you never hit end of life.

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to check your Ruby version health.
