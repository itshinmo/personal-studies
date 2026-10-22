# Agent prompt templates

Copy-paste prompts for working with an AI agent in this project and in other workspaces.

The first two templates are universal: they assume an `AGENTS.md` + `.ai/` knowledge layer but include
fallbacks for workspaces that use different names. The third is specific to `react-shapebound`.

## Contents

1. [Universal — cold start](#1-universal--cold-start)
2. [Universal — warm start](#2-universal--warm-start)
3. [react-shapebound — project task prompt](#3-react-shapebound--project-task-prompt)

## Universal substitutions

Before using the universal templates, substitute as needed:

- `AGENTS.md` → your workspace's instruction file
- `.ai/` → your agent configuration/orientation directory
- `docs/` → your project documentation root
- `decisions/` → your ADR directory
- `rfcs/` → your proposals directory
- `<TASK>` → the actual request
- Verification commands → whatever the workspace actually documents

If a directory does not exist, skip it and say so. Do not invent missing rules, commands, or paths.

---

## 1. Universal — cold start

Use once at the start of a session, or whenever you switch tools/models and the agent has no context.

````text
# Cold start — new session

You are starting a new session in this workspace. Your first job is orientation, not implementation.
Do not edit files, install dependencies, create a plan file, stage, commit, or push.
Do not run destructive commands.

## 1. Find the source of truth

Read in this order, and stop when you have enough for the task:

1. `AGENTS.md` (or the workspace's equivalent instruction file). Follow its links. Treat adapter
   files such as `CLAUDE.md` as pointers, not as a second source.
2. If `.ai/` exists, read `.ai/README.md`, then the orientation memories that exist:
   `.ai/memories/overview.md`, `architecture.md`, `conventions.md`, and
   `development-workflow.md`.
3. `README.md`, `docs/overview.md`, and `docs/architecture.md` (or their equivalents).
4. `decisions/README.md` and the accepted ADRs relevant to the workspace. Read `rfcs/README.md` if it
   exists.
5. `CONTRIBUTING.md` for the human workflow and definition of done.
6. The package/build config and workspace tree, to confirm how the project actually runs.

If a file or directory does not exist, say so and use the fallback. Do not invent commands, paths,
rules, or decisions.

Use progressive disclosure: read indexes and orientation first. After you read the task below, open
only the docs, skills, role definitions, and source files relevant to it. Do not read `node_modules`,
`vendor`, `dist`, `build`, `coverage`, or generated output.

If code/config and docs disagree, code/config wins unless the workspace defines a different
precedence order. Flag the lower source and say what you would fix; never silently choose.

## 2. Identify the operating model

- Find the available roles/subagents and skills in the workspace. State which ones exist and which
  you expect to use for this task.
- If the workspace defines no roles, proceed as a single agent.
- Do not invoke a reviewer/reviewer-equivalent unless I ask.
- Do not invoke an implementer until a plan or spec is agreed.
- If you use a subagent, relay its questions and artifacts through you. Subagents cannot talk to me
  directly.

## 3. Brief me back

Return a compact briefing, not a file dump:

- What this project is, who it is for, and its current state.
- Exact commands: install, dev, build, test, lint, typecheck.
- Architecture in 5–10 lines: main layers/modules and dependency direction.
- The non-negotiables and the approval gate.
- Where knowledge lives: docs, decisions, rfcs, specs, memories.
- What is implemented vs scaffolded vs explicitly not built.
- The roles/skills available and which you expect to use for this task.
- Contradictions, missing files, stale docs, or risks you noticed.
- Anything you could not verify. Tag it `[unverified]` instead of guessing.

Then give a short plan for the task below:

- the first 3–5 steps
- whether this is routine or architectural
- whether a spec, RFC, or ADR is required
- exactly what you need from me before starting

Do not start editing until I approve, unless the task is explicitly routine and non-architectural.
If I already gave you a task, still give the briefing and plan first.

## My task

<TASK>

## Constraints

<optional: must use, must not use, scope, acceptance criteria, files to avoid>
````

---

## 2. Universal — warm start

Use for every subsequent task in the same session, after the agent has already given you its
orientation briefing.

````text
# Warm start — same workspace, context already loaded

You already oriented in this workspace earlier in this session and gave me the briefing. Do not
repeat the full onboarding. Do not re-read every memory or every doc.

## 1. Context check

In 3–5 lines, state:

- the project and its current phase
- the task you are about to do and its acceptance criteria
- the files/modules you expect to touch
- the verification commands you will run

If you are not confident about any of these, use the cold-start template instead and say why.

## 2. Re-read only the delta

- Read the current versions of the files you are about to change, plus the docs/skills directly
  relevant to this task.
- Check `git status` and the current diff before editing.
- If the workspace, tool, model, or session changed since orientation, re-read `AGENTS.md` and the
  orientation memories first.
- If docs and code disagree, flag it; do not silently choose.
- Do not read `node_modules`, `vendor`, `dist`, `build`, `coverage`, or generated output.
- If the task references a spec, RFC, or ADR, read that before anything else.

## 3. Use the established operating model

- Use the roles/skills already identified for this workspace and respect their boundaries.
- If the workspace defines subagents, relay their questions/artifacts through you; they cannot talk
  to me directly.
- Do not invoke a reviewer/reviewer-equivalent unless I ask.
- Do not invoke an implementer until the plan or spec is agreed.
- If this is a new architectural decision, stop and write a spec or RFC first.

## 4. Before editing, confirm

Give me:

- what will change and why
- files/modules affected
- whether this is routine or architectural
- the spec/RFC/ADR path if one is required
- the exact verification commands
- what is explicitly out of scope

Wait for approval if the change is architectural, the spec is missing or ambiguous, or you are about
to add a runtime dependency.

For a routine, reversible change, proceed after the context check.

## 5. Work and handoff

- Follow existing patterns; no unrelated refactors.
- Add or update tests for new behavior.
- Never weaken tests or lint to go green.
- Do not edit generated output, secrets, or lockfiles by hand.
- Do not create a plan file unless I ask.
- Do not stage, commit, or push unless I explicitly ask.
- Report: files changed, behavior delta, commands/results, docs updated, drift-check result, open
  questions, and a diff summary. Stop before commit.

## My task

<TASK>

## Constraints

<optional>
````

---

## 3. react-shapebound — project task prompt

Use this for a task in `react-shapebound`. It is self-contained enough to use after the universal
cold-start orientation, or as the task prompt in a session that already knows the workspace.

````text
# react-shapebound — project task

You are working in `/Users/gasman/react-shapebound`: a single-player, offline, DOM-only browser game
built with React 19, TypeScript 6, Vite 8, React Router 8 (`HashRouter`), and Tailwind CSS 4. The
current state is an architectural scaffold; gameplay rules are not designed yet.

## Read before acting

Always read:

- `AGENTS.md` — single instruction source, approval gate, precedence, non-negotiables.
- `.ai/README.md` — placement rules, roles, workflow.
- `docs/working-with-agents.md` — how roles/skills are invoked.
- `docs/architecture.md` — layers, routing, game domain, persistence, quality gates.
- `docs/design-system.md` — palette, typography, Tailwind conventions, accessibility.
- `docs/game-design.md` — design skeleton; currently mostly TODO.
- `.ai/memories/overview.md`, `architecture.md`, `conventions.md`, `development-workflow.md`.
- `decisions/ADR-002-dom-only-rendering.md`,
  `decisions/ADR-003-pure-game-domain.md`, and
  `decisions/ADR-006-cyberpunk-reference-visual-system.md`.

Then read only what the task needs:

- Gameplay: `.ai/skills/design-a-mechanic/SKILL.md` (Stage 1 only unless I approve later stages),
  `src/game/`, and the relevant `docs/game-design.md` sections.
- UI: `.ai/skills/design-a-feature/SKILL.md`, `docs/components.md`, `src/styles/global.css`, and the
  screen/feature being changed.
- Architecture/state/save/dependency: `.ai/templates/spec.md`, `.ai/skills/write-an-rfc/SKILL.md`,
  `decisions/README.md`, and the relevant source.
- Docs/process: `.ai/skills/knowledge-drift-sync/SKILL.md`.

Do not read every file. Use progressive disclosure.

## Project constraints

- **DOM only.** No canvas, WebGL, or canvas-backed library. No backend, no runtime network calls.
- **`src/game` is pure.** No React, DOM, storage, timers, `Date.now`, or `Math.random`. Seeds and
  timestamps are inputs.
- **Tailwind only.** Semantic tokens in `src/styles/global.css`; no CSS Modules, CSS-in-JS, or raw
  hex in components. The visual system is the live Cyberpunk 2077 reference: yellow `#fcee0a`,
  cyan `#00f0ff`, red `#fe1038`, black structures, Anton/Chakra Petch/Rajdhani/JetBrains Mono.
  Do not copy proprietary fonts, artwork, or logos.
- **No new runtime dependency without an ADR.** Prefer zero.
- **No commit/stage/push unless I explicitly ask.** Run the gate and show me the diff.
- **No invented mechanics.** Gameplay changes go through `game-designer` and
  `design-a-mechanic`; the play screen is a labelled scaffold, not a game.
- **Commands:** `pnpm dev`, `pnpm build`, `pnpm preview`, `pnpm lint`, `pnpm format`,
  `pnpm typecheck`, `pnpm test`, `pnpm check`, `pnpm verify`. Use these exact commands.

## Role routing

Use the roles deliberately. Subagents cannot talk to me directly; relay their questions and
artifacts.

- **Gameplay rule, loop, win condition, balance, or player system** → use the `game-designer`
  subagent with `design-a-mechanic`. Stop after the Stage 1 brief unless I approve the next gate.
  Do not write production code.
- **Screen, component, interaction, or visual state** → use the `designer` subagent with
  `design-a-feature` for the brief. After I approve it, use `implementer` for code and tests.
- **Routing, state shape, save format, layer boundary, or major dependency** → use `documenter` for
  a spec; use an RFC if there are real alternatives; record the accepted decision as an ADR. Do not
  implement before I approve.
- **Small fix, copy change, test addition** → handle directly or with `implementer`; no spec needed.
- **Review** → use `reviewer` only if I explicitly ask. It is read-only and judges the diff.
- **Docs/process correction** → use `documenter`, then `knowledge-drift-sync`.

If the task is ambiguous, stop and ask. Do not fill gaps with plausible design.

## My task

<TASK>

## Task type

<gameplay | UI | architecture | small fix | review | docs>

## Acceptance criteria

<what must be true when this is done>

## Constraints

<must use / must not use / out of scope / files to avoid>
````

---

## When to use which

| Situation | Template |
|---|---|
| New session | **Universal cold start** |
| Switched from Plan to Build, or changed model/tool | **Universal cold start** |
| Context was compacted and you are unsure what the agent remembers | **Universal cold start** |
| Same session, agent already briefed you | **Universal warm start** |
| Second/third task in the same session | **Universal warm start** |
| Small follow-up fix after a completed change | **Universal warm start** |
| Agent starts re-reading everything and burning context | **Universal warm start**, explicitly |
| Agent seems to have forgotten the rules | **Universal cold start** again |
| Any task in `react-shapebound` | **Project task prompt** |
