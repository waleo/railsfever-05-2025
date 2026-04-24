---
layout: post
title: "AI-Assisted Development on a Large Legacy Rails App"
subtitle: "What actually works when you point an AI coding assistant at a five-plus-year-old Rails monolith with real scale and real scars"
description: "An honest look at using Claude and AI coding assistants on large, legacy Ruby on Rails applications. Context strategies, what breaks, and the workflows that deliver value on monoliths that have been in production for years."
author: "Wale Olaleye"
categories: ["AI Development", "Rails Maintenance"]
tags: [ai rails development, legacy rails app, large rails monolith, claude code, rails ai tools]
image: "/assets/images/blog/jenga-stack-800x600.webp"
preview_image: "/assets/images/blog/jenga-stack-400x300.webp"
---

The AI demos you see online are almost always small apps. A Todo list. A Jekyll blog. A fresh Rails 8 scaffold. Those demos are honest about what they are, but they are not representative of what most Rails work looks like.

Most Rails apps in the wild are not fresh. They are five to fifteen years old. They have 50+ models, dozens of engineers who have come and gone, patched gems, monkey-patched initializers, and a `config/` directory that nobody alive fully understands. They also generate most of their company's revenue.

So the practical question is: can AI actually help on *that* kind of codebase? The honest answer is yes, but not by default, and not in the way demos suggest. This post lays out what works and what does not when you bring an AI coding assistant to a large, legacy Rails application.

## What Breaks First: Context

The single biggest constraint on AI-assisted development in a large Rails app is context. The model cannot see your whole codebase at once. Even with large context windows, a 200,000-line Rails app with views, migrations, locales, and initializers will overflow fast.

What this means in practice: the AI will confidently generate code that conflicts with conventions in files it never saw. It will import gems you removed. It will reference services that do not exist. It will assume a controller inherits from `ApplicationController` when you actually have a custom base class three levels up.

The fix is not "use a bigger model." The fix is disciplined context management:

- **Pin the relevant files.** Before asking for a change, explicitly load the files that matter. The controller, the model, the service object, the spec. Not "the signup flow" — actually the four files that make up the signup flow.
- **Summarize what the AI cannot see.** If your codebase has an unusual base class, a custom authentication middleware, or a forked gem, write that down once in `CLAUDE.md` or a skill file. The AI does not know until you tell it.
- **Scope the task.** "Fix the N+1 in `UsersController#index`" is a good prompt. "Audit the whole app for performance issues" is not — the AI will give you a generic checklist because the surface area is too large to reason about.

Teams that get value on legacy apps treat context as a budget. Every token the AI spends reading an irrelevant file is a token it cannot spend reasoning about your actual problem.

## What Breaks Second: Conventions

Fresh Rails apps look like the Rails guides. Legacy apps look like whatever the team agreed to in 2016. Common drift:

- `before_filter` instead of `before_action` (or a mix of both)
- Mass-assignment protection via `attr_accessible` instead of strong parameters
- Custom ORM patterns (squeel, hand-rolled scopes, Arel-heavy queries)
- Service objects in `app/services/` with idiosyncratic patterns — `.call`, `.execute`, `.perform`, or a mix
- View layers with ERB, HAML, and Slim coexisting
- Configuration scattered across `config/initializers/`, `config/settings.yml`, environment variables, and a `Settings` model

The AI will cheerfully "modernize" any of this when you ask it to edit a file — which means every PR turns into a style debate unless you head it off.

The mitigation is the same as above, with more specificity: write the conventions down. Not "follow our patterns" — actually: "This app uses service objects with a `.call` class method. Do not introduce instance-method variants. Service objects live in `app/services/` and do not inherit from a base class. Return a `Result` object from `lib/result.rb`." That level of specificity turns AI output from 60% usable to 90% usable.

## What Actually Works: Pattern-Matched, Well-Scoped Work

On large legacy Rails apps, AI earns its keep on tasks that are high-volume, well-understood, and locally scoped. Concrete examples:

**Writing tests for untested code.** Legacy Rails apps are famous for thin test coverage. Point the AI at a model, give it the factory definitions, and ask for comprehensive specs. Review the output — it will miss some edge cases — but you get a 70% jump in coverage in an afternoon.

**Backfilling documentation.** YARD comments, `# frozen_string_literal: true` headers, method-level docstrings. Boring, high-volume, pattern-matched. Ideal AI work.

**Extracting service objects from fat controllers.** If you show the AI the target pattern (one example service object) and the controller action to extract from, it can produce a reasonable first draft. You review and merge. The productivity difference is real.

**Gem upgrades.** AI is good at reading a changelog, matching it against your codebase, and flagging likely impact. "Upgrade Sidekiq from 6 to 7 — here is the changelog, here is our initializer, what do we need to change?" This is a genuinely useful workflow.

**Finding dead code.** With grep and careful reasoning, AI can identify methods, routes, and views that have no callers. Not perfectly, but well enough to produce a candidate list a human can verify.

**Translating old patterns to new.** `before_filter` → `before_action`, `attr_accessible` → strong parameters, ERB → ViewComponent. Mechanical translations with high volume are exactly what AI is good at.

## What Does Not Work: Business Logic Buried in Legacy Code

The AI is not going to read your `PricingCalculator` class and understand that the third `if` branch exists because a specific customer signed a custom contract in 2019 that nobody wrote down.

For anything that depends on unwritten business rules — pricing logic, fraud detection, feature gating, regulatory edge cases — the AI will produce plausible-looking code that is subtly wrong. The failure mode is insidious because the code compiles, the tests pass (the tests the AI wrote), and the bug surfaces three weeks later in production when a real customer hits an edge case.

The defense is review. Not "does this look right" — actually trace the logic and compare it against what the code *should* do. For load-bearing business logic, AI is a drafting tool; a senior engineer still owns the final shape.

## Migrations and Background Jobs: Handle With Care

Two specific categories deserve extra caution on large legacy Rails apps.

**Database migrations.** On a 50-million-row table, the difference between a locking migration and a non-locking one is the difference between a clean deploy and a three-hour incident. AI knows about `ALGORITHM=INPLACE`, `strong_migrations`, and batched updates — but it does not *always* use them unprompted. Always review migrations by hand before running them in production. Always.

**Background jobs.** Legacy Rails apps often have dozens of Sidekiq workers with subtle invariants: retry semantics, queue priorities, idempotency assumptions, coordination with external systems. AI will generate a worker that "works" but may not respect those invariants. Again: write them down. Review carefully.

## The Codebase Map Is Your Best Investment

The single highest-leverage artifact on a legacy Rails app is what we call a codebase map — a document that tells the AI (and new humans) what is where, what each major module does, and what the land mines are.

A minimal codebase map for a large Rails app might include:

- Core domain objects and where they live
- Major subsystems (billing, auth, notifications, reporting) and the directories that contain them
- External integrations and where they are isolated
- Background job topology (which queues exist, what they do, what runs where)
- Known sharp edges: "The `legacy_users` table is not what it sounds like — it is the current users table, renamed during a botched migration in 2019"
- Engineering conventions that differ from default Rails

Write this once, check it in, and update it quarterly. It compounds across every AI session you run and every new engineer you onboard.

## Parallel AI Is Where the Real Leverage Is

On a small app, one engineer plus one AI session is fine. On a large legacy Rails app, the leverage pattern is different: one engineer orchestrating several parallel AI workflows.

Example: while you are manually debugging a production issue, a background Claude session is writing specs for the module you touched yesterday, another is auditing gem vulnerabilities, and a third is preparing a readiness report for next quarter's Rails upgrade. You check the output periodically and merge what is good.

This pattern — human as conductor, AI as multiple first drafts in parallel — is where the biggest velocity gains on legacy apps come from. It also requires more organizational muscle than most teams have yet, which is why it is a frontier rather than a norm.

## Expected Results: The Honest Numbers

On a fresh Rails app, AI can plausibly do 70-90% of the initial build. On a large legacy Rails app, the realistic number for most teams today is 20-40% of development output — and that is if you have done the context and convention work above.

Those are not small numbers. A team that ships 30% more per quarter without hiring is a team that is winning. But the marketing claims of "AI will write all your code" are not what the experience on a real production monolith looks like.

The teams getting the most value are not the ones using the fanciest tools. They are the ones that invested in the boring infrastructure: `CLAUDE.md`, skills libraries, codebase maps, and disciplined review processes. Those investments compound. The AI gets better *on your specific codebase* over months of accumulated context.

---

Running a large legacy Rails app and wondering how to realistically integrate AI into your workflow? Our [Rails Care Plan](/services/rails_care_plan/) and [technical audit](/services/rails_tech_audit/) engagements increasingly include the codebase-mapping and AI-tooling work that make this possible — so you get velocity without the cleanup costs. See also our posts on [tips for using Claude with an existing Rails app](/2026/04/10/tips-for-using-claude-with-an-existing-rails-app.html) and [creating Claude skills for Rails apps](/2026/04/13/creating-claude-skills-for-rails-apps.html).

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to talk about AI on your legacy Rails codebase.
