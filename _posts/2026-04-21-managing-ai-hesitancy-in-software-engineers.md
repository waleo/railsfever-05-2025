---
layout: post
title: "Managing AI Hesitancy in Software Engineers"
subtitle: "Why engineers push back on AI adoption harder than most employees — and what actually works when the people you need to bring along are senior developers"
description: "A guide for engineering leaders on addressing AI hesitancy among software engineers specifically. Why engineers resist AI more than other functions, what the real concerns are, and how to lead adoption on a senior engineering team."
author: "Wale Olaleye"
categories: ["Engineering Leadership", "AI Development"]
tags: [ai adoption engineers, software engineer ai hesitancy, claude code adoption, engineering leadership, ai pair programming]
image: "/assets/images/blog/robot-on-wooden-floor-besides-a-chalk-board-with-the-words-my-favorite-robot-800x600.webp"
preview_image: "/assets/images/blog/robot-on-wooden-floor-besides-a-chalk-board-with-the-words-my-favorite-robot-400x300.webp"
---

Software engineers are a special case in AI adoption. On one hand, they are the function where AI has the most immediate and measurable leverage — code generation, test writing, refactoring, and debugging are all workflows where modern LLMs demonstrably help. On the other hand, engineers tend to push back on AI harder and more articulately than most employees.

That combination — highest potential value, strongest resistance — makes engineering adoption both the most important and the hardest part of an AI rollout. Generic "change management" playbooks do not land. The standard talking points ("AI will make you more productive") get reverse-engineered and rebutted inside thirty seconds of a team meeting.

This post is specifically for engineering leaders — CTOs, VPs, staff engineers — who need to bring a senior engineering team along on AI adoption. The general dynamics of employee hesitancy are covered in our post on [managing employee AI hesitancy](/blog/managing-employee-ai-hesitancy/); this post goes deeper into what is specific to engineers.

## Why Engineers Push Back Harder

The surface-level explanation is wrong. It is not that engineers are "resistant to change" or "afraid of being replaced." Engineers have spent their careers adopting new languages, frameworks, and tools. Change itself is not the problem.

The real dynamics are more specific:

**Engineers can evaluate AI output technically.** Most employees see AI output and take it at face value. Engineers read the code and see the bugs. They see the hallucinated API calls, the subtly wrong logic, the mismatched types, the tests that test nothing. When a non-engineer says "AI is amazing at drafting emails," the engineer hears the equivalent of "AI is amazing at writing plausible-looking nonsense" — because they have watched it do both.

**Craftsmanship is identity.** Senior engineers built their careers on their code being good. Being asked to ship code they did not fully write — and in some cases did not fully understand — attacks that identity directly. This is not ego. It is the rational response of a professional whose reputation rests on the work being correct.

**The failure modes are expensive.** A marketing email drafted by AI and sent with a typo is recoverable. A subtle bug in payment code shipped by an over-enthusiastic AI user is not. Engineers have a well-developed sense for this asymmetry; they know the blast radius of their mistakes.

**Engineers have the most mature tooling to compare against.** Modern IDEs, linters, test suites, and type systems already remove a lot of boilerplate. The value proposition "AI will speed up your work" lands differently for an engineer whose IDE already autocompletes half their code than for a marketer who has no comparable tooling.

**Engineers read leadership more skeptically.** Every engineer has lived through at least one technology-of-the-year mandate that did not work out. They have calibrated their skepticism accordingly, and leadership that sounds like the last failed rollout gets the pattern-matched response.

Each of these is a legitimate reason to push back. Engineering leadership that treats them as obstacles to overcome rather than inputs to take seriously loses credibility fast.

## The Concerns That Are Specifically Engineering

Beyond the general concerns common to all employees, engineers tend to raise issues that are unique to their work:

**Code quality and long-term maintainability.** "Even if AI writes working code today, is this the code I want to maintain in two years?" Engineers know that code is read far more often than it is written, and that the cost of a bad abstraction compounds.

**Intellectual property and codebase exposure.** "When I paste code into an LLM, where does that code go? Is it training future models? Is it leaking to competitors?" This concern is especially acute at companies with proprietary systems.

**The craft regression problem.** "If junior engineers use AI to skip the hard parts, they never develop the instincts to recognize when AI is wrong. We are building a generation of senior engineers who cannot function without AI." This is not a straw man — it is a serious concern about professional development.

**Autonomy and authorship.** "My satisfaction in this job comes from solving problems. If the AI solves the problem and I just review, what exactly am I doing?"

**Review load.** "I used to review one PR. Now I review three PRs, all of which were written in 30 minutes by someone using an AI. The review bottleneck has moved to me and nobody adjusted my workload."

**Skill depreciation.** "If I spend two years mostly reviewing AI output instead of writing code, am I still a strong engineer when I look for my next job?"

Each of these is a real concern with a real answer. Glossing over them loses the room.

## What Does Not Work on Engineers

**Productivity numbers without code.** "Teams using AI ship 40% more features" is meaningless to engineers who do not know what teams, what features, or what the code quality looked like six months later. Engineers will always ask for the detail you do not have.

**Vendor demo videos.** Every engineer has seen the polished demo and knows the difference between a demo and a real codebase. Demos actively hurt credibility.

**Framing AI as replacing work engineers dislike.** "AI will handle the boring parts!" Engineers know that some of the "boring parts" — writing tests, documentation, maintenance — are the parts that build institutional knowledge and prevent outages. Handing them entirely to AI is not obviously a win.

**Hiding the skepticism.** Engineering leadership that pretends the concerns are unreasonable or that the team is "just resistant" destroys its own credibility. Engineers respect leaders who engage honestly with hard problems and distrust leaders who do not.

**Productivity theater.** Dashboards tracking "lines of code AI-generated per engineer" are transparently silly. Engineers know that lines of code is a bad metric, and that "AI-generated" is trivially gamed.

## What Does Work

### Adopt AI Yourself, Visibly

Engineering leaders who want their teams to use AI need to be using it themselves, meaningfully, and talking about it in technical terms. Not "I love Claude, it's so cool." Actual specifics: "I used Claude Code to generate the scaffolding for the new admin interface this morning. Here is the diff. Here is where it got things right and here is where I had to rewrite."

This does several things. It normalizes AI use at the senior level. It gives engineers concrete examples to react to. It signals that leadership is evaluating the tools on the same grounds engineers are, rather than just repeating marketing claims.

Leaders who advocate AI without using it themselves lose credibility immediately.

### Respect the Quality Concerns With Real Process

The "code quality" concern is legitimate. The answer is not to dismiss it but to build process that addresses it.

- **Require thorough code review on AI-generated PRs.** Same standard as human-written code, or stricter. No rubber-stamping.
- **Invest in the test suite and CI.** AI-assisted development works well when tests are comprehensive and fast, because the AI can close the loop. It works badly when tests are sparse and slow.
- **Set up pattern libraries** (see the [creating Claude skills for Rails apps](/blog/creating-claude-skills-for-rails-apps/) post). AI that imitates your team's patterns produces code that fits the codebase; AI that follows its own defaults produces code that doesn't.
- **Adopt a code ownership model** where the human who merges the PR is responsible for the code, not the AI.

When engineers see that leadership is as serious about quality as they are, they engage differently. When leadership's posture is "just ship more," engineers resist — and they are right to.

### Address the IP and Privacy Concerns Explicitly

Before telling engineers to use AI tools on company code, answer in writing:

- Which tools are approved for code access, and under what terms?
- What do the vendor's data retention and training policies say, in plain language?
- Is there an enterprise agreement that prevents training on customer data?
- What code is off-limits (customer data, security-sensitive modules, regulated systems)?

This is not a one-page document. This is a clear, specific, technical answer to questions engineers will ask within the first week. Have it ready.

If you do not have an enterprise agreement with your LLM provider, get one before asking engineers to use AI on proprietary code. The additional cost is small compared to the adoption cost of unclear policy.

### Acknowledge the Craft Regression Concern

The concern that junior engineers using AI might not develop the instincts of their senior colleagues is real. Pretending it is not makes leadership look naive.

The response is not to forbid juniors from using AI — that ship has sailed, and AI is going to be part of the permanent tooling for the rest of their careers. The response is to be deliberate about what juniors are learning:

- **Pair junior engineers with senior ones on code review of AI output.** This is where the craft transfers.
- **Require juniors to explain, in writing, the reasoning behind AI-generated code they submit.** Not a pro-forma comment — actual "why does this work, and what are the failure modes?"
- **Reserve some projects or features as "AI-off" training ground** for junior engineers who need to build raw coding skill.
- **Build internal education around the things AI does badly** — systems design, performance analysis, debugging production incidents — where craft still matters and AI is still weak.

Done well, this produces juniors who are stronger than their predecessors because they know both the fundamentals and how to use AI effectively.

### Rebalance Workload as AI Adoption Rises

If AI-generated code means engineers review three PRs instead of one, that has to be reflected in workload expectations. Otherwise you create a burnout trap.

Concrete adjustments:

- **Reduce individual feature-ship quotas** as AI assistance increases review load.
- **Rotate review responsibility** so no single engineer becomes a review bottleneck.
- **Track review turnaround time** as a team metric, not just PR velocity. If reviews are slowing down, the AI output has outpaced the review capacity.

This is where many AI rollouts quietly fail. The velocity goes up, the engineers burn out, and six months later the "AI is great" narrative is replaced by "engineers are quitting."

### Frame AI as Leverage, Not Replacement

The framing that works with engineers is not "AI will make you faster." It is "AI is a leverage multiplier on your judgment."

Concretely: AI writes the boilerplate, you review the architecture. AI drafts the tests, you validate the coverage. AI suggests refactors, you decide whether they are right for this codebase. AI produces the first draft of documentation, you refine for audience and accuracy.

This framing preserves the part of engineering most engineers care about — judgment, taste, and ownership — while offloading the parts most engineers agree are tedious. It also sets realistic expectations about where review effort goes.

### Give Engineers Time to Learn the Tools Properly

Using AI well is a skill. Senior engineers are used to being competent at their tools, and a drop in competence when adopting a new one is uncomfortable. If leadership rushes engineers past the awkward learning phase, many will give up and go back to what they know.

Concrete investments:

- **Protected learning time** — 4-6 hours a week, first two months, explicitly for figuring out AI workflows.
- **Internal Claude Code playbooks** documenting your team's specific setup, conventions, and skills.
- **Office hours** with engineers who are further along on AI adoption.
- **Permission to be slower in the short term.** Engineers who are told to "use AI and hit your normal velocity" will rationally conclude that AI is a distraction.

The teams that invest in this phase get engineers who are genuinely competent with AI tooling by month three or four. The teams that skip it get engineers who "use AI" cosmetically and have not actually integrated it.

### Measure What Actually Matters

The temptation to build AI dashboards is strong. Most of the metrics people instinctively track are wrong:

- Lines of AI-generated code (gamed trivially, says nothing about quality)
- Number of AI sessions per engineer (same)
- Time saved per AI interaction (hard to measure, easy to manipulate)

Better metrics — still imperfect, but at least meaningful:

- Team PR throughput and cycle time (did adopting AI actually ship more shippable work?)
- Defect rate and time-to-resolution on production issues (did quality hold?)
- Engineer self-reported satisfaction and sustainability (are they enjoying the work or drowning in review?)

These metrics move slowly and require honest conversation. That is what makes them useful.

## A Specific Move That Builds Trust

If you want a single, high-leverage move: pick a real, non-trivial engineering project — something with enough shape that it would normally take a week — and do it end-to-end yourself with heavy AI assistance, in front of the team. Not a tutorial. A real project that ships to production.

Walk the team through what worked and what did not. Show the prompts, the failures, the corrections, the final diff. Be honest about what was faster and what was slower.

This single exercise does more for credibility than any number of all-hands messages. It demonstrates that leadership has actually tried to do the work the team is being asked to do, and has formed specific, honest opinions about it.

## The Long Arc

Engineering teams that successfully adopt AI tend to go through a recognizable arc:

- **Month 1-2:** Skepticism. Early experiments mostly frustrating. Most engineers try AI, find it mediocre, and return to their old workflows.
- **Month 3-4:** A few engineers get genuinely good at AI-assisted workflows. They start producing visibly faster work without quality degradation.
- **Month 5-6:** Skilled users begin teaching other engineers. Internal patterns emerge. Team-level tooling (CLAUDE.md, skill libraries) starts getting built.
- **Month 7-12:** AI integration stabilizes. Most engineers use it daily for specific, well-defined workflows. A minority remain skeptical holdouts; that is fine and usually healthy.

The teams that fail at AI adoption tend to quit between month 2 and month 3, when the initial experiments have not paid off and the learning investment has not yet compounded. Leadership that pushes through this valley — without dismissing the legitimate concerns that drove the skepticism — comes out the other side with a genuinely AI-capable team.

Leadership that tries to shortcut the valley with mandates produces a team that claims to use AI and actually does not.

---

Leading an engineering team through AI adoption and looking for outside perspective? Our [Fractional CTO service](/services/fractional_cto/) includes engineering-culture and tooling guidance, often with direct hands-on work on AI integration. For teams on Rails specifically, see also our posts on [tips for using Claude with an existing Rails app](/blog/tips-for-using-claude-with-an-existing-rails-app/) and [how to increase the percentage of development done by AI](/blog/how-to-increase-percentage-of-development-done-by-ai/).

[Schedule a consultation]({{ site.schedule_meeting_link }}) or email <a href="mailto:hello@railsfever.com" class="email-link">hello@railsfever.com</a> to talk about AI adoption on your engineering team.
