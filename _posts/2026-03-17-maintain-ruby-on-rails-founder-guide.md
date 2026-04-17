---
layout: post
title: "How to Maintain Ruby on Rails: A Founder's Guide to Keeping Your App Healthy"
subtitle: "Why Rails maintenance is a business decision, not a developer preference - and what good maintenance looks like from the outside"
description: "A founder-friendly guide to maintaining Ruby on Rails apps. Learn what Rails maintenance actually costs, why it matters, and how to tell whether your team (or vendor) is doing it right."
author: "Wale Olaleye"
categories: ["Rails Maintenance", "Founders"]
tags: [maintain ruby on rails, rails maintenance, saas, technical debt, founder guide]
image: "/assets/images/blog/man-fixing-vehicle-engine-800x600.webp"
preview_image: "/assets/images/blog/man-fixing-vehicle-engine-400x300.webp"
---

If you are a founder whose product runs on Ruby on Rails, there is a question that will eventually land on your desk. Not when things are going well. Usually in the week after a CVE lands, or the month a Ruby version goes EOL, or the quarter a customer asks about SOC 2.

The question: *"are we maintaining this app properly?"*

Most founders cannot answer it confidently. Not because they are bad founders - because nobody ever told them what "maintaining Ruby on Rails" actually involves, what it costs, or what it looks like when it is going right. This guide is that explanation, in business terms.

If you want the deep developer playbook, see [how to maintain an existing Rails app](/2026/03/03/how-to-maintain-an-existing-rails-app.html). This post is for the person who signs the check.

## What "Maintain Ruby on Rails" Really Means

Rails is a framework that ships a new minor version roughly once a year and a new major version every couple of years. Ruby does something similar. The ecosystem of gems you depend on moves continuously.

"Maintaining Ruby on Rails" means keeping pace with that movement at a rhythm that fits your business:

- Applying security patches when CVEs land.
- Upgrading Ruby before your version goes end-of-life.
- Upgrading Rails often enough to stay supported.
- Keeping the hundred or so gems in your app reasonably current.
- Keeping your own code clean enough that the above is not terrifying.

Done well, this costs a predictable, small amount of engineering capacity each month. Done poorly, it costs you a giant emergency project every few years.

## Why This Is a Business Decision

Founders often delegate "maintenance" to the dev team because it sounds technical. That is a mistake. The dev team can execute maintenance. Only you can decide how much risk the business is willing to carry.

Three business realities only you can judge:

### 1. Security risk tolerance

If you hold customer data, payments, PII, or health information, your tolerance for running an un-patched framework is very low. If your app is an internal tool used by 12 people, it is higher. Maintenance frequency should follow risk.

### 2. Enterprise sales readiness

The moment you start selling to larger companies, security questionnaires start arriving. "What versions of Ruby and Rails are you on?" is on every one. "We're on Rails 5.2 and Ruby 2.7" closes no deals.

### 3. Engineering velocity

Unmaintained Rails apps get slower to change. Features that took a day in year one take a week in year five. That is a direct drag on your roadmap - and it is invisible until you try to hire someone new and watch them flinch at your Gemfile.

None of those are "should we use Propshaft?" questions. They are founder questions.

## The Three Modes of Rails Maintenance

I have seen three patterns across the dozens of Rails apps I have worked on. Only one is sustainable.

### Mode A: Reactive maintenance (the default, the worst)

Maintenance happens when something breaks. A CVE hits the news, a gem stops installing, a customer reports a bug. The team scrambles, fixes it, goes back to features.

Symptoms:
- Nobody knows what Rails version you are on off the top of their head.
- Last Rails upgrade was "a while ago."
- `bundle install` fails on new engineers' laptops.
- Every production incident surprises somebody.

Cost profile: flat line for 2-3 years, then one enormous project - often $150K-$300K of emergency upgrade work. See [Ruby on Rails maintenance cost](/2025/08/18/ruby-on-rails-maintenance-cost.html) for the numbers.

### Mode B: Big-bang periodic upgrades

Every 3-4 years, the team stops feature work for a quarter and does a giant upgrade. Rails 4 to 6. Rails 5 to 7. Everything at once.

Symptoms:
- Predictable pain, roughly every 3 years.
- Features freeze during the project.
- One or two senior engineers carry all the risk.
- The app is "current" for about 18 months after, then starts drifting again.

Cost profile: lumpy, expensive, and the roadmap suffers for a full quarter or two each time.

### Mode C: Continuous, incremental maintenance

Maintenance is a small, predictable slice of engineering capacity each month. Gems are upgraded continuously. Rails moves minor-by-minor. Ruby gets bumped when patch releases drop. Upgrades are a PR, not a project.

Symptoms:
- You can answer "what versions are we on?" without checking.
- Rails and Ruby are within one minor of latest supported.
- CVEs are handled within days, not months.
- The team feels safe deploying on Fridays.

Cost profile: smooth, predictable, and measurably cheaper over 3+ years than either of the first two modes. This is the only mode that actually scales with the business.

## What Good Maintenance Looks Like From the Outside

Even if you are not an engineer, you can tell whether your Rails app is being maintained well by looking at a few signals. Ask your CTO, lead engineer, or agency these questions:

### 1. "What versions of Ruby and Rails are we on, and when were they last upgraded?"

A healthy answer is specific and recent. "Ruby 3.3.x, Rails 8.0.x, last upgraded in the current quarter." A concerning answer is vague or older than two years.

### 2. "How often do we apply security patches?"

Healthy: on a set schedule (weekly or monthly) plus emergency patches within days of disclosure. Concerning: "when we notice something."

### 3. "What's our plan for the next Rails version?"

Healthy: a rough quarter, a rough scope, and an owner. Concerning: "we'll look at it next year."

### 4. "Can we deploy to production right now?"

Healthy: yes, today, with one command, by any engineer on the team. Concerning: "only Person X knows the deploy."

### 5. "When's the last time we upgraded a gem that wasn't Rails itself?"

Healthy: recent and regular. Concerning: "I'd have to check."

These five questions tell you more about your Rails maintenance health than any technical deep-dive.

## What Maintenance Costs

Rough numbers for a typical small-to-medium Rails app (one to five engineers, no regulated industries):

- **Continuous maintenance**: 5-10% of an engineer's time, or roughly $2K-$6K/month depending on who does it.
- **Quarterly minor upgrades**: 1-3 days per quarter of focused work.
- **Annual major upgrades**: 1-3 weeks per year, planned and scoped.
- **Unplanned CVE response**: rare and short, when you are current.

Compare that to:

- **Big-bang upgrades from 3+ majors behind**: $80K-$300K, 3-6 months.
- **Emergency rescue from production outages**: six figures, unpredictable.
- **Lost enterprise deals** because your stack fails a security review: possibly the biggest line item.

The math isn't subtle. Continuous maintenance is one of the highest-ROI engineering investments a SaaS business can make.

For a longer take on the economics, see [stop wasting money on one-off Rails upgrades](/2025/11/02/stop-wasting-money-one-off-rails-upgrades.html).

## Who Should Do the Maintenance

You have three real choices, each with tradeoffs.

### Your existing team

Works if:
- The team has time and isn't drowning in product work.
- Someone on the team genuinely enjoys maintenance (rare but valuable).
- You can protect maintenance time from being constantly deprioritized for features.

Does not work if:
- The team is full-stretch on product.
- Your Rails expertise is concentrated in one person (bus factor of one).
- Maintenance time keeps getting cannibalized - which is what usually happens.

### A specialist consultancy

Works if:
- Your team is focused on shipping the product.
- You want predictable monthly spend rather than surprise projects.
- You want an outside expert to review your team's decisions.

This is what a [Rails Care Plan](/services/rails_care_plan/) is designed to do.

### Fractional CTO

Works if:
- You are pre-CTO, or your CTO is stretched across product, hiring, and fundraising.
- You need someone to own the whole technical health picture, not just Rails maintenance.

See our [Fractional CTO service](/services/fractional_cto/).

The right answer depends on company stage, team shape, and how complex the app is.

## Red Flags That You Need to Act Now

A short list of signals that say "maintenance is not happening, intervene this quarter":

- You cannot answer "what Ruby version are we on?" in under 30 seconds.
- Your last Rails upgrade was more than 18 months ago.
- Your CI is red "sometimes" and nobody has fixed it.
- Engineers are afraid to touch parts of the codebase.
- Deploys require a specific person's laptop.
- You have been hit by a CVE and waited more than a week to patch.
- You've lost at least one deal over your security questionnaire answers.

One of these is a yellow light. Three of these is a red one.

## The Founder's One-Page Maintenance Checklist

If you only take one thing from this post, take this:

1. Know what Ruby and Rails versions you are on.
2. Know the date of your last framework upgrade.
3. Make sure someone is explicitly responsible for Rails maintenance - by name.
4. Put a fixed monthly maintenance capacity on the calendar.
5. Plan the next Rails upgrade before you need to.
6. Review the health of the app once a year with an outside expert.

Do those six things and you will not be the founder calling for an emergency rescue.

---

Not sure where your Rails app stands? We offer a [Rails technical audit](/services/rails_tech_audit/) that gives you an honest, founder-readable report on your maintenance health. For ongoing care, our [Rails Care Plan](/services/rails_care_plan/) replaces reactive scrambling with predictable monthly maintenance. For broader technical leadership, explore our [Fractional CTO service](/services/fractional_cto/).

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to talk through your Rails app's health.
