---
layout: post
title: "Creating Claude Skills for Rails Apps"
subtitle: "How to package repeatable Rails workflows as Claude skills — what they are, when to use them, and a worked example"
description: "A practical guide to building Claude skills for Ruby on Rails development. What a skill is, how it differs from a CLAUDE.md file, and step-by-step examples for common Rails workflows like upgrade prep and gem audits."
author: "Wale Olaleye"
categories: ["AI Development", "Developer Guide"]
tags: [claude skills, claude code, ai rails development, rails automation, ai pair programming]
---

If you use Claude Code on Rails work regularly, you end up typing the same prompts over and over. "Audit this gem for known CVEs and suggest replacements." "Find every controller action missing a before_action authentication check." "Walk the app and list hardcoded strings that should be in i18n." Useful, but tedious to re-articulate every time.

Claude skills are the fix. A skill is a reusable, invokable workflow — a named package of instructions that Claude can execute on demand. Once you have a skill, you do not re-explain the workflow; you invoke it.

For Rails teams, skills are where a lot of AI leverage actually lives. This post explains what a skill is, when to build one, and walks through a concrete Rails example.

## What a Claude Skill Actually Is

A skill is a markdown file with a small bit of frontmatter, stored in a directory Claude knows to look in. The frontmatter gives the skill a name and a short description; the body gives Claude the instructions to follow when the skill is invoked.

The key difference from a `CLAUDE.md` file: `CLAUDE.md` is **always** in context — it shapes every response. A skill is only loaded when triggered, either by the user typing `/skill-name` or by Claude deciding the task matches the skill's description.

That distinction matters. `CLAUDE.md` is for things that are true about your codebase at all times ("we use RSpec, not Minitest"). Skills are for workflows you run occasionally but want to be consistent when you do ("audit our Gemfile for outdated gems").

## When a Skill Is the Right Tool

Build a skill when:

- **The workflow is multi-step and the steps matter.** If you want to make sure Claude always checks CVEs, then flags deprecation notices, then proposes upgrades *in that order*, a skill encodes the sequence.
- **You want the workflow to be invokable by a less technical team member.** A product manager who cannot write a detailed prompt can still type `/audit-gems` and get a consistent result.
- **You want to capture institutional knowledge.** The senior engineer who knows the right way to profile an N+1 leaves; the skill they wrote still runs the same way.

Do not build a skill for a one-off task. And do not try to encode all of Rails into a single mega-skill — small, focused skills compose better.

## A Worked Example: Rails Upgrade Prep

Let us build a skill that prepares an app for a Rails upgrade by producing a concrete readiness report. This is a workflow most Rails teams run manually every time; a skill makes it consistent and fast.

Create a file at `.claude/skills/rails-upgrade-prep.md` in your repository:

```markdown
---
name: rails-upgrade-prep
description: Audit the current Rails app and produce a readiness report for upgrading to the next Rails minor version. Use when the user asks to prepare for a Rails upgrade, assess upgrade readiness, or check what blockers exist.
---

You are preparing an upgrade readiness report for this Rails application.

Steps to follow in order:

1. Read `Gemfile.lock` and identify the current Rails version and Ruby version.
2. Determine the target Rails version (the next minor release, unless the user specifies otherwise).
3. Check the target version's minimum Ruby requirement from the Rails upgrade guides.
4. Run `bundle outdated --only-explicit` and note gems that have major version bumps available.
5. Run `bundle exec bundle-audit check --update` if the gem is available; flag any CVEs.
6. Grep the codebase for deprecation warnings currently being raised (look for `ActiveSupport::Deprecation`, silenced deprecations, and comment markers).
7. List any frozen or version-pinned gems that would block the upgrade.
8. Check the test suite:
   - Does `bin/rspec` or `bin/rails test` pass?
   - Are there skipped or pending tests relevant to the upgrade surface area?
9. Produce a markdown report with sections:
   - Current state (Rails/Ruby versions, test status)
   - Blockers (things that must change before the upgrade)
   - Risks (things that might break, ranked by likelihood)
   - Recommended sequence (what to upgrade, in what order)

Do not modify any files. This skill is read-only and produces a report.
```

Now when anyone on the team types `/rails-upgrade-prep` — or even asks "can you check if we are ready to upgrade Rails?" — Claude runs the same structured audit.

## A Second Example: Hardcoded String Audit for i18n

A common Rails task: find every user-facing string that should be in a locale file. Here is what that looks like as a skill, stored at `.claude/skills/audit-i18n.md`:

```markdown
---
name: audit-i18n
description: Scan views, helpers, and mailers for hardcoded user-facing strings that should be moved to locale files. Use when the user asks to audit i18n coverage, find missing translations, or prepare for internationalization.
---

Scan the Rails app for hardcoded strings that should be in `config/locales/*.yml`.

Include:
- ERB/HAML/Slim templates in `app/views/`
- Helpers in `app/helpers/` returning user-visible strings
- Flash messages in controllers (`flash[:notice] = "..."`)
- Mailer subject lines and bodies in `app/mailers/` and `app/views/*_mailer/`
- Validation error messages in models (if not already using `I18n`)

Exclude:
- HTML attributes that are not user-visible (aria-hidden labels that match visible text, data attributes)
- Strings that are clearly technical (log messages, error class names)
- Anything already wrapped in `t()`, `I18n.t`, or `translate`

Output a markdown table with columns: File, Line, Current String, Suggested Key.

Group the table by file. Do not modify any files — this is read-only.
```

Same idea: a tedious, error-prone audit becomes a one-word command.

## Structural Tips for Good Skills

A few patterns that separate skills people actually use from ones that gather dust:

**Keep the body short.** A skill that fits on one screen is easier to maintain and more likely to execute cleanly. If you are writing 200 lines of instructions, you probably have two skills, not one.

**Name the skill after what it does, not how.** `audit-gems` is better than `run-bundle-audit`. The description should make the trigger obvious: "Use when the user asks to check gem security or find vulnerable dependencies."

**Make read-only skills explicitly read-only.** For anything that just produces a report, say "Do not modify any files" in the skill body. Claude is less likely to be helpful in unhelpful ways.

**Add a stop condition.** If the skill could run indefinitely (e.g., scanning a large codebase), tell Claude when to stop: "Stop after the first 100 matches and note that the scan was truncated."

**Version-control the skills.** `.claude/skills/` should be in the repo, not in a developer's home directory. The whole point is team consistency.

## Skills That Are Worth Building for Most Rails Apps

If you want a starter list, here are the skills that tend to pay for themselves quickly on Rails projects:

- `rails-upgrade-prep` — readiness report for the next Rails version (above)
- `audit-gems` — outdated gems, CVEs, replacement recommendations
- `audit-i18n` — hardcoded strings for translation (above)
- `audit-n-plus-one` — scan for likely N+1 patterns in controllers
- `audit-strong-params` — find controllers missing strong parameters or using permissive params
- `audit-authorization` — find controller actions missing authorization checks
- `generate-service-object` — create a new service object following your project's conventions
- `audit-deprecations` — surface Rails and gem deprecations currently being triggered

Build the one that addresses your biggest current pain. You can always add more later.

## What Skills Are Not

Skills are not a way to automate a deploy pipeline. They are not a CI replacement. They run in a Claude session, against your codebase, with a human watching the output.

They also do not remove the need for review. A good skill produces a report or a change set that a human still signs off on. The value is consistency and speed of the *first pass*, not autonomy.

## Getting Started

The minimum viable path:

1. Create `.claude/skills/` in your Rails repo.
2. Pick one workflow you run repeatedly — the gem audit or the i18n scan are good starters.
3. Write a 30-line skill file following the frontmatter format above.
4. Commit it.
5. Use it. Tweak the instructions based on what Claude actually produces.

By the third skill, the pattern is obvious. By the tenth, your team has a meaningful library of AI-driven Rails workflows that just run.

---

Looking to integrate AI tooling into a Rails team in a durable way? Our [Rails Care Plan](/services/rails_care_plan/) increasingly includes skill libraries tuned to each client's codebase — so the AI leverage compounds instead of dissipating between sessions. See also our post on [tips for using Claude with an existing Rails app](/2026/04/10/tips-for-using-claude-with-an-existing-rails-app.html) for the broader setup.

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to talk about AI-assisted Rails development.
