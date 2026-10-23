**Table of Contents**

* [What are AI Artifacts](#what-are-ai-artifacts)
* * [Key Features](#key-features)
* * [Types of Artifacts](#types-of-artifacts)
* [What a spec is](#what-a-spec-is)
* [What a slice is](#what-a-slice-is)
* [Questions](#questions)
* * [Sub Agents VS Main Agents](#sub-agents-vs-main-agents)
* * * [Why sub agents exist](#why-sub-agents-exist)
* * * [Why the main agent did not use them](#why-the-main-agent-did-not-use-them)
* * * [What the sub agents are actually for](#what-the-sub-agents-are-actually-for)
* * * [The important distinction](#the-important-distinction)

---

# What are AI Artifacts?

AI artifacts are the different results or by products that come from creating and using AI systems. These can be things like trained models, patterns in data, unexpected errors or biased behavior. Some artifacts are useful for improving performance while others reveal problems such as unfair treatment or incorrect outputs. Studying these artifacts helps developers build AI systems that are more accurate, fair and easier to understand.

## Key Features

* **Diverse Forms:** AI artifacts appear in many forms, including model files, unexpected behaviors, data patterns, interface elements and social impacts. This diversity reflects the complexity of AI systems.
* **Unintended Consequences:** Many artifacts are side effects that were not deliberately designed such as biases or errors which can affect AI reliability and fairness.
* **Insightful for Improvement:** Artifacts provide valuable information that helps developers understand, debug and improve AI models and data quality.
* **Impact on Trust and Transparency:** By revealing how AI systems work or misbehave artifacts influence how users perceive and trust AI technologies.

## Types of Artifacts

* Model Artifacts
* Behavioral Artifacts
* Data Artifacts
* Interface Artifacts
* Ethical and Social Artifacts

--- 

# What a spec is
A spec (specification) is a short written agreement for one deliverable. It answers:

- Why are we doing this?
- What is in scope, and what is deliberately not?
- What does “done” mean, in checkable terms?
- What contracts does the change introduce — state shape, rules, input, UI, tokens?
- What can go wrong, and how will we verify it?

It is not code, and it is not a permanent document. In this repo:

- It lives in `.ai/specs/` — local and gitignored. Gameplay specs go in `.ai/specs/mechanics/<slug>.md`.
- It follows `.ai/templates/spec.md`.
- When the work lands, durable behavior is folded into `docs/`, decisions into an ADR, and the spec is deleted. It is never committed as project knowledge.
- You review and approve it before code, so I do not invent scope or add “helpful” extras.

# What a slice is

A slice is a small, end-to-end, playable or testable increment — not a technical layer like “the state layer” or “the UI layer.” It cuts through domain, feature, UI, docs, and tests, but deliberately stops before the rest of the game.

Your full vision is the roguelike loop: survive, earn points, defeat bosses, die, spend points in the skill tree, repeat. This slice is the movement foundation:

- Full-screen black board
- Square player
- Keyboard and touch-joystick movement
- Whole-app landscape gate and best-effort orientation lock
- Pure player stats and health/shield model

Deliberately excluded: HUD, enemies, bosses, points, skill tree, timer, damage source, death, run/meta progression.
It is the smallest piece you can actually playtest before adding combat and economy. It also proves the architecture: pure rules in src/game, browser input/loop in the feature, DOM rendering with transforms, and docs that match the code.

# Questions

These questions are the ones i asked my coding assistant when working on different projects, thats why some questions/answers are a bit vague!

## Sub Agents VS Main Agents

### Why sub agents exist

“The main agent can do it” is not really the reason subagents exist. Subagents are less about capability and more about **separation of concerns, context isolation, permissions, and independent review.**

### Why the main agent did not use them

I worked end-to-end in the main session because the tasks were explicit, incremental, and tightly coupled: routing, layout, clipping, title, and settings were changing back and forth in one working tree. Handoffs would have added coordination overhead, and each subagent would have started with less context than this conversation had.

That was a valid trade-off for velocity, but it underused the workflow. For the larger changes—the shared shell, settings schema migration, and glow feature—I should have used at least:

- **designer** for the UI brief,
- **implementer** for the approved implementation,
- **reviewer** before the final commits,
- **documenter** for the knowledge reconciliation.  

I especially did not get the benefit of an independent review because I reviewed my own work.

### What the sub agents are actually for

- **Specialization:** each role has a narrower prompt and checklist. A designer focuses on hierarchy, states, tokens, and accessibility; an implementer focuses on the approved spec and tests; a reviewer looks for regressions without the author’s bias.
- **Context isolation:** a subagent gets a fresh context. That keeps the main conversation manageable and lets it focus on one task.
- **Independent review:** the reviewer is read-only and should not be the same voice that wrote the code. That catches things self-review misses.
- **Approval gates:** the intended flow is design → brief → approval → implementation → review → documentation. A single agent doing all roles collapses those gates.
- **Different tools or models:** a reviewer can be read-only; an implementer can edit; an explorer can search broadly; a game designer should not write production code.
- **Parallelism:** independent work can run in separate worktrees/sessions without blocking the main thread.
- **Consistency:** roles and skills encode repeatable process. They are useful precisely when a task is large enough that “just do it directly” starts to lose rigor.

### The important distinction

**A skill is instructions. An agent is a worker with a role and a fresh context. Loading `design-a-feature` gave me the UI checklist, but it did not give me the independent `designer` or `reviewer` perspective. That is why using a skill is not a substitute for using a subagent.

They are not mandatory for every edit. For a small styling fix, direct work is reasonable. For a multi-file feature with persistence, routing, accessibility, and tests, they add real value.

Going forward, a better default would be:

- small fix → main agent directly
- new UI → designer → implementer → reviewer
- gameplay rule → game-designer → implementer → reviewer
- knowledge/docs → documenter or knowledge-drift-sync
- broad exploration → explore

So the point of adding them is not that the main agent cannot type the code. It is that one agent should not be the only designer, implementer, reviewer, and documenter on a substantial change. In this session, I chose speed and continuity over that separation, and that is a legitimate thing to call out.