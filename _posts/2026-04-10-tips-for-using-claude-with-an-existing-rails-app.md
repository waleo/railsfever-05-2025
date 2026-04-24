---
layout: post
title: "Tips for Using Claude with an Existing Rails App"
subtitle: "Practical ways to get real value from Claude Code on a Rails codebase that is already in production"
description: "A practical guide to using Claude and Claude Code with an established Rails application. CLAUDE.md files, test-first prompting, context windows, and the specific things that trip up AI on Rails conventions."
author: "Wale Olaleye"
categories: ["AI Development", "Developer Guide"]
tags: [claude code, ai rails development, claude rails, ai pair programming, rails ai tools]
image: "/assets/images/blog/close-up-of-computer-screen-with-code-and-menu-options-800x600.webp"
preview_image: "/assets/images/blog/close-up-of-computer-screen-with-code-and-menu-options-400x300.webp"
---

Using Claude on a greenfield Rails app is easy. The codebase is small, the conventions are Claude's defaults, and there is almost no prior art to conflict with. Using Claude on a six-year-old Rails monolith with 40 models, 200 controllers, and a decade of team-specific conventions is a different exercise.

The AI does not magically understand your app. It understands Rails. Your job is to close the gap between the two.

Here are the tips that actually move the needle when you put Claude in front of a real, established Rails codebase.

## Start With a CLAUDE.md File

The single highest-leverage thing you can do is drop a `CLAUDE.md` in the root of your repository. Claude Code reads it automatically on every session and treats it as persistent instructions.

Put the things a new senior engineer would want to know on day one:

- The Ruby and Rails versions in use.
- Test runner and command (RSpec? Minitest? `bin/rails test`? `bundle exec rspec`?).
- Any local conventions that deviate from default Rails (service object patterns, form objects, query objects, naming conventions for concerns).
- What **not** to touch without asking — legacy modules, vendored gems, anything load-bearing that does not have tests.
- The database. If it is Postgres with specific extensions enabled, say so. If it is MySQL with unusual collation, say so.
- How to run the linter and which rules are non-negotiable (RuboCop config quirks, Standard, Packwerk boundaries).

Keep it short. A 50-line `CLAUDE.md` is more useful than a 500-line one, because Claude will actually respect every line. Long documents dilute attention.

## Make the Tests Runnable From a Cold Start

Claude operates best when it can run your tests and see the result. If `bundle exec rspec spec/models/user_spec.rb` requires five env vars, a running Postgres, a seeded database, and a Sidekiq worker, Claude will either skip running tests (and ship code that does not work) or waste a lot of context figuring out the setup.

The fix: make it possible to run a single spec file against a test database with one command, ideally with no external services required. Use `:rack_test` or WebMock for external calls. Seed the test DB with Rails fixtures or a minimal seeds file that runs as part of `db:test:prepare`.

If you cannot get there in one afternoon, at least document in `CLAUDE.md` exactly what needs to be running before Claude attempts tests.

## Point Claude at the Right Part of the Codebase

Context windows are large but not infinite, and a Rails monolith will never fit entirely. The trick is scoping.

Before asking Claude to change anything, give it the relevant files explicitly. Not "the user signup flow" — actually: `app/controllers/registrations_controller.rb`, `app/models/user.rb`, `app/services/user_onboarding.rb`, and the associated spec files.

This sounds tedious, but it has two benefits. First, Claude's output stays grounded in the actual code rather than inventing plausible Rails defaults. Second, you end up understanding your own signup flow better than you did before you asked.

For exploratory work, the `Grep` and `Glob` tools inside Claude Code let it find files on its own. That works well for "find every place we call the Stripe API" but less well for "change the way we validate signups" — for the second case, you want to be specific.

## Respect the Conventions Already in the Codebase

Claude has strong Rails defaults. Your app has its own conventions. When those conflict, Claude will happily generate code that works but looks nothing like the rest of the codebase — which turns every PR into a style review.

Two moves help:

1. **Write the convention down in `CLAUDE.md`.** "We use service objects in `app/services/` with a `call` method. Controllers should not contain business logic." Claude will follow this.
2. **Show it an example.** When asking for a new service object, paste a reference service object into the prompt, or tell Claude which file to use as a template. Imitation is more reliable than instruction.

This is especially true for apps that predate the current Rails conventions — if your app still uses `before_filter` or hand-rolled strong parameters, Claude needs to know, or it will "helpfully" modernize as it goes.

## Use Claude for the Boring, High-Volume Work

The highest return on investment is on tasks that are tedious and pattern-matched: writing RSpec tests for existing code, adding i18n keys to hardcoded strings, generating form objects from existing controllers, backfilling YARD documentation, converting ERB to ViewComponent.

These are the tasks that a senior engineer *can* do but does not *want* to do, and where getting it 85% right and reviewing the diff is faster than writing it from scratch.

What Claude is less good at: anything that requires understanding a subtle business rule buried across three files. For that, you still need a human in the loop — or at a minimum, you need to explicitly surface the business rule in the prompt.

## Be Careful With Migrations and Destructive Changes

Claude Code will happily generate and run a database migration if you let it. On a toy app, that is fine. On a production Rails app, it is not.

Three safeguards:

- Configure Claude's permissions so that `rails db:migrate` requires approval. Do not auto-allow.
- Always have Claude write the migration file first, then stop. Review the migration, run it manually, and then come back to Claude for the code that depends on it.
- For any migration that touches a table with more than a few million rows, review the locking behavior yourself. Claude knows about `ALGORITHM=INPLACE` and `DISABLE_DDL_TRANSACTION!` but will not always use them unprompted.

The same principle applies to anything that mutates production-adjacent state: deploys, data backfills, cron jobs, feature flags.

## Keep a "Known Pitfalls" List

After a few weeks of using Claude on your codebase, you will start to notice the same mistakes. Maybe it keeps importing a gem you removed six months ago. Maybe it insists on using `update_attribute` instead of `update!`. Maybe it generates specs that assume a test helper you do not have.

Every time you correct one of these, add a one-line note to `CLAUDE.md`:

```
- Do not use `update_attribute`; use `update!` and let it raise on failure.
- Our Sidekiq queues are named `critical`, `default`, `low`. Do not invent new ones.
- We use Factory Bot, not fixtures. Factories live in `spec/factories/`.
```

This is the single most important investment in getting Claude to feel like a team member rather than a generic Rails engineer.

## Do Not Let Claude Run Forever Without Review

Agent mode is powerful. It is also the fastest way to generate a PR that compiles, passes the tests Claude wrote, and does the wrong thing.

Set a cadence: after every logical chunk of work (one feature, one bug fix, one refactor), read the diff carefully. Not just "does this look right" — actually trace the logic. Claude is unusually good at producing code that *reads* like it works. The bugs show up when it runs against real data.

On larger changes, break the work into smaller commits and review each one. This makes bisecting easy when something does go wrong in production.

## The Bigger Picture

Claude is not a replacement for understanding your own codebase. It is a very fast junior engineer who has read every Rails tutorial ever written but has never seen your app before. The leverage you get is proportional to how much context you give it and how carefully you review its output.

Most teams under-invest in the `CLAUDE.md` file and over-invest in clever prompts. The lasting wins come from the boring, persistent configuration.

---

Need help integrating AI tooling into an established Rails development process? Our [Rails Care Plan](/services/rails_care_plan/) and [technical audit](/services/rails_tech_audit/) engagements are increasingly built around pairing senior Rails engineers with AI tooling — so you get the velocity without the cleanup bills.

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to talk about AI-assisted Rails development.
