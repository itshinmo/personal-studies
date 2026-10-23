**Table of Contents**

* [What are AI Artifacts](#what-are-ai-artifacts)
* * [Key Features](#key-features)
* * [Types of Artifacts](#types-of-artifacts)
* [What a spec is](#what-a-spec-is)
* [What a slice is](#what-a-slice-is)

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