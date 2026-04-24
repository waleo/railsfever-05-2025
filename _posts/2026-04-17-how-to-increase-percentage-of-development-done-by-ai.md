---
layout: post
title: "How to Increase the Percentage of Development Done by AI"
subtitle: "Specific, repeatable moves that take a team from AI-curious to AI-integrated — and actually raise the share of shipped code that came from the assistant"
description: "A practical playbook for increasing the share of development work performed by AI on Ruby on Rails teams. Tactical moves, organizational patterns, and the investments that compound over time."
author: "Wale Olaleye"
categories: ["AI Development", "Engineering Leadership"]
tags: [ai coding productivity, claude code, ai software development, rails ai tools, engineering leadership]
image: "/assets/images/blog/ai-on-whiteboard-800x600.webp"
preview_image: "/assets/images/blog/ai-on-whiteboard-400x300.webp"
---

"We are using AI" has become the software version of "we are doing Agile." Technically true at most companies. Not actually changing the output at most of them.

The question that matters is not whether your team is *using* AI. It is: what percentage of the code shipping to production each week was written by AI? For most Rails teams today, the honest answer is 10-20%. The teams that are genuinely getting leverage are at 40-60%. A few are higher.

This post is about what separates those tiers. It is not a list of prompts. It is a set of organizational and technical moves that compound — and each one raises the ceiling on how much of your team's output can reasonably come from AI.

## Start by Measuring Honestly

You cannot improve what you do not measure. Before any intervention, get a baseline. A rough but useful way to measure the AI share of development:

- Count commits in the last month and estimate what share were AI-initiated (co-authored, generated, or heavily assisted).
- Alternatively, count PR counts by engineer and cross-reference against self-reported AI usage.
- Or just ask each engineer: "What share of your code this sprint came from an AI first draft?"

You will get a rough number. That is fine. Measure the same way each month and watch the trend. If the number is not moving, none of the investments are sticking.

## Remove the Friction That Blocks AI From Being Useful

Most engineers do not use AI heavily because the friction of using it well is higher than the friction of just writing the code themselves. To raise the AI share, lower that friction systematically.

**Make the test suite fast and runnable from a clean slate.** If `bundle exec rspec spec/models/user_spec.rb` takes 45 seconds to boot, the AI loop is broken. Every iteration costs a minute. Get boot time under 5 seconds. Use Spring, bootsnap, or whatever it takes. This one change doubles usable AI velocity for most Rails teams.

**Eliminate external dependencies from the default test run.** If running a single spec requires Redis, Elasticsearch, Postgres, and S3 credentials, the AI cannot close the loop on its own. Use WebMock, in-memory Redis alternatives, and the Rails test fixtures pattern aggressively.

**Standardize the development environment.** If half the team is on `asdf`, a quarter is on `rbenv`, and a few people use Docker, the AI has to guess at conventions. Pick one. Document it in `CLAUDE.md`. Make it reproducible with a single setup script.

**Make deploys boring.** If every deploy is a manual ritual, the AI cannot ship. If deploys are `bin/deploy` with a one-command rollback, AI-driven work can flow all the way to production with a human in the loop for review.

These are not AI projects. They are developer-experience projects. But they are the projects that determine whether AI adoption actually sticks.

## Write the Conventions Down — Specifically

The number one reason AI-generated code gets rewritten by reviewers is that it does not match the team's conventions. The number one reason that happens is that the conventions live in reviewers' heads, not in any file.

Fix this. Put conventions in a `CLAUDE.md` at the root of the repo. Specifically:

- Which test framework (RSpec, Minitest) and which style (mocks, fixtures, factories)
- Service object patterns (method name, base class, return shape)
- Error handling patterns (raise, return `Result`, return `nil`)
- API response shapes
- i18n coverage expectations
- Controller patterns (skinny controllers? fat controllers? what goes where?)
- Database migration patterns (strong migrations, batched updates, expected review gates)

Every convention that lives only in review becomes a convention the AI will get wrong. Move them into text. The payoff: AI output that ships with 80% fewer convention-level review comments.

## Build a Library of Skills

If you are writing the same prompt three times, it should be a skill. (See our [guide to creating Claude skills for Rails apps](/2026/04/13/creating-claude-skills-for-rails-apps.html).)

Skills to build first, in order of payoff:

1. **A test-writing skill** that follows your project's conventions
2. **An upgrade prep skill** for Rails and Ruby version bumps
3. **An i18n audit skill** for finding hardcoded strings
4. **A dead code audit** to remove unused methods and routes
5. **A code review skill** that checks your team's style before PR review

Each of these shifts work from humans to AI, and each one runs consistently because the instructions are persistent. The compound effect is substantial: after six skills, a meaningful chunk of maintenance work runs on command.

## Invest in Pattern Corpora, Not Just Prompts

AI is better at imitating than at following abstract instructions. So the best investment is not a cleverer prompt — it is a richer set of *examples* for the AI to imitate.

Maintain a `docs/patterns/` directory in your repo with exemplar files:

- One "perfect" service object
- One "perfect" controller action with strong parameters and authorization
- One "perfect" background job with idempotency and retry semantics
- One "perfect" RSpec spec with factories and shared examples
- One "perfect" migration with `strong_migrations` safety

When you ask the AI to generate something, point it at the pattern file. "Write a service object for X, following the pattern in `docs/patterns/example_service.rb`." The output quality jump is immediate and persistent.

## Shift From Individual Use to Team Use

AI that lives in one engineer's terminal helps that engineer. AI that is configured, committed, and shared with the whole team helps everyone.

Commit the `CLAUDE.md`. Commit the `.claude/skills/` directory. Commit the pattern files. Build a shared library that every engineer's Claude session inherits on day one.

The difference in velocity between a team where each engineer configures AI independently and a team where the configuration is shared is substantial — probably 2x at steady state. Most teams are in the first camp because it is the default; the work is in getting to the second.

## Restructure Work for AI Throughput

Beyond tooling, the biggest lever is how work itself is organized. Two structural changes that raise the AI share:

**Break work into smaller, well-specified units.** AI is great on a 200-line change with clear acceptance criteria. It is less great on a 2000-line "fix the billing system" epic. Stories that would normally be estimated at two days can often be decomposed into four half-day units, each of which is mostly AI-drivable.

**Front-load specification.** When the spec is clear — inputs, outputs, edge cases, test cases — AI can execute most of the implementation. When the spec is "make it work like the old thing but better," AI flounders. Engineering leaders who invest 20 minutes writing a crisp spec buy 4 hours of AI execution.

This reshapes the job of senior engineers. Less time writing code, more time decomposing problems and reviewing output. That transition is harder culturally than technically.

## Parallel Work Is the Frontier

One engineer + one AI session is the baseline. One engineer orchestrating three or four parallel AI sessions is where the leverage gets interesting — and where most teams are not yet.

The pattern: while you are reviewing one PR, a second session is drafting tests for the work you just landed, a third is running a codebase audit you kicked off an hour ago, and a fourth is preparing the next feature's scaffolding. You check in periodically, redirect where needed, and merge what passes review.

This requires infrastructure: clean worktrees, non-conflicting branches, AI tools that can run in the background. It also requires a mental shift from "I am coding" to "I am directing several coding streams." Teams that crack this pattern report step-function velocity gains.

## Do Not Skimp on Review

Every post about increasing AI share should come with this caveat: the goal is not "AI writes everything while humans rubber-stamp." That path ends in production incidents and slow decay of codebase quality. The goal is "AI drafts more, humans review more carefully."

Protect review rigor as the AI share rises. If anything, review standards should *tighten* as more AI-generated code lands — because the failure mode of AI code is plausible-looking bugs that compile and pass superficial tests.

Specifically: trace logic by hand for anything touching business rules, money, authorization, or data integrity. Read migrations line by line. Re-run tests on the actual change (not just on the AI's self-written tests).

## A Realistic Timeline

Getting from 10-20% AI share to 40-60% is not a weekend project. Realistic pacing:

- **Month 1:** Write `CLAUDE.md`. Commit shared AI config. Measure baseline.
- **Month 2-3:** Build first five skills. Clean up test suite friction. Add pattern files.
- **Month 4-6:** Restructure work breakdown to favor AI-drivable units. Pilot parallel AI workflows with one senior engineer.
- **Month 6+:** Scale the parallel workflow pattern across the team. Refresh skills quarterly.

Teams that try to rush this — six weeks to "AI does 50% of our code" — tend to ship more code but also more bugs, and then retreat. The teams that go steady state at high AI share got there deliberately over six to nine months.

## What Not to Chase

A few things that sound like they should raise AI share but mostly do not:

- **Bigger models.** Useful, but marginal compared to the investments above. A 10% better model on top of 0% infrastructure still gets you 10% of something small.
- **Cleverer prompts.** Prompts lose to persistent configuration every time. Write it in `CLAUDE.md` once, not in a prompt every day.
- **More AI tools.** Adding Cursor *and* Claude Code *and* Copilot does not 3x the AI share. Pick one, commit to it, and go deep.
- **Tracking AI metrics obsessively.** Measure quarterly, not daily. The interventions are slow-acting.

## The Compounding Effect

The thing that is easy to miss: each of these investments compounds with the others. A better `CLAUDE.md` makes skills more reliable. More reliable skills make parallel workflows viable. Parallel workflows reward cleaner work decomposition. Cleaner work decomposition makes the whole team faster, which frees capacity to refine `CLAUDE.md`.

Six months in, the teams that stuck with it are running at a fundamentally different velocity than the teams that treated AI as a tool to sprinkle on. The gap widens over time.

Getting there takes deliberate investment, not enthusiasm.

---

Working on raising AI share on a Rails team and not sure where the biggest blockers are? Our [technical audit](/services/rails_tech_audit/) now includes an assessment of AI-readiness — test suite friction, convention documentation, skill library — alongside the usual Rails health check. See also our post on [AI-assisted development on a large legacy Rails app](/2026/04/15/ai-assisted-development-on-large-legacy-rails-app.html).

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to talk about AI strategy on your Rails team.
