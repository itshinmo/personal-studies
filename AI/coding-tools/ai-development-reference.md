# Developing with AI: A Reference Guide

**Built from a real repository:** [`mimshins/sudo-overclock`](https://github.com/mimshins/sudo-overclock) — a statically generated Next.js engineering blog whose author built an unusually complete "AI layer" around it.

**Who this is for:** you, if you are new to AI-assisted development and want to understand not just *what* files to create, but *why* each one exists and what breaks without it.

**How to use it:**

- Read Part I first. It gives you the mental model; everything else is detail hanging off it.
- Parts II–V are the tour of the repository, topic by topic. Each section ends with **Why it matters** and **What to copy**.
- Part VI turns it into something you can do this week. Don't build the whole thing — build Level 0.
- The appendices contain copy-paste starters. Appendix A is a complete `AGENTS.md` you can adapt in 20 minutes.

**A note on honesty:** this document describes what *this* repository does. It is one opinionated solution, not the only correct one. Where the repo made a trade-off, the cost is stated too.

---

## Table of contents

**Part I — The mental model**
1. [What "developing with AI" actually means](#1-what-developing-with-ai-actually-means)
2. [The four problems every AI setup must solve](#2-the-four-problems-every-ai-setup-must-solve)
3. [The one test that shapes everything](#3-the-one-test-that-shapes-everything)

**Part II — The repository tour**
4. [The annotated map](#4-the-annotated-map)
5. [`AGENTS.md`: the single instruction source](#5-agentsmd-the-single-instruction-source)
6. [Tool adapters: one source, many tools](#6-tool-adapters-one-source-many-tools)

**Part III — Knowledge management**
7. [Precedence: what wins when sources disagree](#7-precedence-what-wins-when-sources-disagree)
8. [The knowledge taxonomy](#8-the-knowledge-taxonomy)
9. [Memories and working memories](#9-memories-and-working-memories)

**Part IV — Capabilities**
10. [Skills: packaging a repeatable workflow](#10-skills-packaging-a-repeatable-workflow)
11. [Agents: roles with boundaries](#11-agents-roles-with-boundaries)

**Part V — Process and safety**
12. [Spec-driven development](#12-spec-driven-development)
13. [Spec → RFC → ADR → docs → archive](#13-spec--rfc--adr--docs--archive)
14. [Guardrails and verification](#14-guardrails-and-verification)
15. [The nine-stage writing pipeline](#15-the-nine-stage-writing-pipeline)

**Part VI — Practice**
16. [The maturity ladder](#16-the-maturity-ladder)
17. [Ten rules of thumb](#17-ten-rules-of-thumb)
18. [Start-today checklist](#18-start-today-checklist)
19. [Anti-patterns](#19-anti-patterns)
20. [What this repository deliberately omits](#20-what-this-repository-deliberately-omits)
21. [Glossary](#21-glossary)

**Appendices**
- [A. Starter `AGENTS.md`](#appendix-a-starter-agentsmd)
- [B. Minimal folder layout and adapters](#appendix-b-minimal-folder-layout-and-adapters)
- [C. Starter skill and starter ADR](#appendix-c-starter-skill-and-starter-adr)
- [D. Prompts to use with your agent](#appendix-d-prompts-to-use-with-your-agent)
- [E. Source index](#appendix-e-source-index)

---

# Part I — The mental model

## 1. What "developing with AI" actually means

An AI coding tool is not a smarter autocomplete. It is three things bolted together:

1. **A model** — a text predictor. It has no memory of your project between sessions, no ability to *know* your conventions unless they are written down somewhere it reads.
2. **Tools** — file read/write, search, shell, git. These let it *act*, not just talk. This is why an agent can break your build.
3. **A context window** — the finite amount of text it can hold at once. Everything it "knows" about your task is whatever fits in there.

A useful metaphor: the model is a brilliant contractor who arrives each morning with **total amnesia** and a **small desk**. It can do excellent work, but only from what is on the desk, and it will confidently guess when something is missing.

Almost every practice in this repository exists to answer one of two questions:

- *What goes on the desk?* (knowledge, instructions, conventions)
- *What stops it from doing damage?* (boundaries, approval gates, verification)

The word **agent** means a model that can take multiple actions in a loop — read a file, edit it, run a test, read the failure, edit again — until a goal is met. A **subagent** is a separate agent session with its own clean context, given one job. You will see five of those in this repo.

**Why it matters:** once you see the model as an amnesiac with a small desk, "prompt engineering" stops being about magic phrases and becomes about *information architecture*: putting the right knowledge where it will be read, and nothing else.

---

## 2. The four problems every AI setup must solve

This repository's `.ai/` layer is essentially four solutions in one folder. Knowing the four problems makes the files stop looking arbitrary.

### Problem 1 — Context: the agent doesn't know your project

Without written knowledge, every session starts with you re-explaining the architecture, the commands, the naming rules, and the thing you tried last week that didn't work. And when the window fills up, older instructions silently fall out.

The repository's answer: a layered knowledge system, loaded **on demand**:

- one entry file (`AGENTS.md`) that is always read;
- short **memories** that orient the agent in 30 seconds;
- **docs** that hold the full contract, read only when relevant;
- **skills** with stage files, so only the branch you need is loaded.

### Problem 2 — Drift: written knowledge and reality diverge

The moment a doc says one thing and the code does another, the agent picks whichever it read last. Ten small divergences later, the agent is confidently wrong.

The repository's answer: **link, don't copy** (one canonical home per fact), a **knowledge-maintenance table** ("if you change X, review Y"), and a **drift-sync skill** that runs before handoff and requires docs to change *in the same commit* as the code.

### Problem 3 — Authority: two sources disagree and nobody decides

The model needs a tie-breaker that is written down, not inferred. Otherwise it resolves conflicts by recency or by plausibility, which is exactly the wrong heuristic.

The repository's answer: an explicit **precedence order**, plus the rule *"fix or flag the lower source and say so"* — never silently choose.

### Problem 4 — Safety: the agent can commit, publish, and delete

The most expensive failures are irreversible: a push to main, a rewritten post, a deleted branch, a new dependency nobody reviewed.

The repository's answer: an **approval gate** (agents never stage, commit, or push unless asked), **read-only boundaries** around generated output and published content, a **human-owned publish command**, and a **pre-push hook** that runs lint and tests.

**Why it matters:** when you design your own setup, ask which of the four you are solving. Most people only ever solve Problem 1, then wonder why the agent keeps "forgetting" conventions and committing things.

---

## 3. The one test that shapes everything

From `.ai/README.md`:

> If every AI tool and this directory were removed, the site, the authoring CLI, and every decision would still stand on its own.

This is the **stand-alone test**, recorded formally in `ADR-008 — Agent-Agnostic Knowledge Layer and Guardrails`. Before the refactor that produced it, the repo failed this test in three concrete ways:

| Symptom | Consequence |
|---|---|
| Post templates lived in `.ai/templates/` and were read at runtime by `pnpm author:new` | Deleting `.ai/` **broke the CLI** |
| Every decision was written up to three times (RFC + memory + docs) | The copies drifted apart |
| Only `AGENTS.md` existed; skills weren't in a discoverable format | The process was locked to one vendor's tool |

After the refactor:

- `docs/` holds current behavior and rules.
- `decisions/` holds ADRs (why).
- `rfcs/` holds proposals.
- `.ai/` holds **only** orientation, workflows, and roles — and it **links** to the above.
- Post templates moved into the application code (`src/modules/blog/infrastructure/authoring/templates/`).

The rule stated in `AGENTS.md`: **"Runtime code never reads from `.ai/`."**

**Why it matters:** this single constraint prevents the most common long-term failure of AI-shaped repositories — the project slowly becomes unbuildable without the AI folder, and its history becomes invisible to humans. It also means you can adopt a new tool without a migration.

**What to copy:** before adding anything to your AI folder, ask *"would a human need this with no AI tooling?"* If yes, it does not go in the AI folder. If it is a fact that already exists in your docs, **link to it** instead of restating it.

---

# Part II — The repository tour

## 4. The annotated map

This is the actual layout, with what each part is for:

```
AGENTS.md                     ← THE instruction file. Always read first.
CLAUDE.md                     ← one line: @AGENTS.md  (a pointer, not a copy)

.claude/agents   -> ../.ai/agents     (symlink)
.claude/skills   -> ../.ai/skills     (symlink)
.opencode/agents -> ../.ai/agents     (symlink)
.opencode/skills -> ../.ai/skills     (symlink)
.agents/skills   -> ../.ai/skills     (symlink)

.ai/
  README.md                   map of the AI layer + placement rules
  agents/                     5 role definitions
    documenter.md             spec / RFC / ADR author, knowledge keeper
    designer.md               UI briefs, design-system guardian
    implementer.md            writes code + tests to an agreed spec
    reviewer.md               definition-of-done check, on request
    editor.md                 editorial lead for blog posts
  skills/                     4 reusable workflows
    post-authoring/
      SKILL.md                dispatcher: reads `stage:` and routes
      stages/                 brief, research, outline-critique,
                              editorial, preflight (one file each)
    design-a-feature/SKILL.md
    write-an-rfc/SKILL.md
    knowledge-drift-sync/SKILL.md
  memories/                   short present-state orientation
    overview.md               what the project is, where knowledge lives
    architecture.md           layering + "decisions not to fix"
    conventions.md            naming, imports, styling, tests
    content-pipeline.md       compiler orientation
    development-workflow.md   routing, gates, hooks, approval
    working/                  committed in-flight context (has a lifecycle)
  templates/                  7 templates
    spec.md  rfc.md  adr.md  memory.md  working-memory.md
    skill.md  agent.md
  specs/                      local task specs — GITIGNORED

docs/                         human project knowledge (NOT AI-specific)
  architecture.md  design-language.md  design-system.md
  components.md    imagery.md  authoring.md  runbook.md
decisions/                    ADRs: why things are the way they are
rfcs/                         active/ and archived/ proposals
.githooks/pre-push            versioned hook: lint + test before push
.github/PULL_REQUEST_TEMPLATE.md   definition of done
CONTRIBUTING.md               human onboarding + how to invoke agents
```

Two things to notice immediately:

1. **The AI folder is small.** ~20 markdown files. It is not a second documentation site; it is a *routing layer*.
2. **The valuable stuff is outside it.** `docs/`, `decisions/`, `rfcs/` are ordinary, human-readable project knowledge. The AI layer points at them.

**Why it matters:** the ratio is the lesson. A beginner's instinct is to put everything in `.cursor/rules` or a giant `AGENTS.md`. This repo puts *orientation* there and keeps *substance* in normal project docs.

---

## 5. `AGENTS.md`: the single instruction source

`AGENTS.md` is the emerging cross-tool convention (used by Codex, OpenCode, Cursor, and others) for "the file an agent should read first". It is ~250 lines and deliberately contains **no duplicated detail** — it is an index plus rules.

Its sections, and why each exists:

### a) Scope statement

> "The only instruction source for AI agents working on sudo-overclock… Every tool reads this file (Claude Code through `CLAUDE.md` → `@AGENTS.md`); tool folders only point into `.ai/`. Read this first, then follow the links; do not duplicate their contents here."

This does two jobs: it tells the agent where to start, and it tells future-you not to add a second source of truth.

### b) The approval gate, at the top

> **"Do not run `git add`, `git commit`, or `git push` (or anything that stages, such as `git mv` or `git rm`) unless the author explicitly asks.** After verification, stop and hand the diff to the author."

Placed first, not last, because instruction-following degrades over a long context. Safety rules go where they are seen.

Note the phrasing: it names the *specific commands*, including the sneaky ones (`git mv`, `git rm` stage changes). And it says what to do **instead**: verify, then hand over the diff. A rule without an alternative behaviour gets ignored.

### c) Precedence

An eight-level tie-breaker (see §7).

### d) Spec-driven development

A threshold and a five-step loop (see §12). Crucially it defines **when the heavy process is not needed**: "Routine implementation, small changes, and bug fixes need no spec unless they are architectural."

### e) Mandatory actions

Six concrete obligations: follow the authoring pipeline, hold the design line, RFC before trade-off decisions, ADR for hard-to-reverse choices, keep knowledge in sync, add a changeset, verify before finishing.

Each one names the file or command that implements it. "Keep knowledge in sync" is not a vibe; it links to `knowledge-drift-sync/SKILL.md` and a table.

### f) The knowledge-maintenance table

Twelve rows of the form *"if you change X, review Y"*:

| If you change... | Review... |
|---|---|
| layers, module shape, imports, aliases | `docs/architecture.md`, `oxlint.config.ts`, `.ai/memories/architecture.md` |
| the compiler or generated output | `docs/architecture.md#content-pipeline`, `docs/authoring.md`, `docs/runbook.md`, `.ai/memories/content-pipeline.md` |
| tokens, typography, color, motion, layout | `docs/design-system.md`, `docs/design-language.md`, `src/app/globals.css`, `src/app/theme-contrast.test.ts` |
| a dependency | an ADR for runtime `dependencies`; `README.md` tech stack |
| … | … |

This table is the highest-leverage artifact in the file. It converts "keep the docs updated" (a wish) into a lookup you can execute.

### g) Where things live

A topic → path index, so the agent can stop reading and jump. It ends by naming the architectural shape in three lines, so you don't need `docs/architecture.md` for orientation.

### h) Non-negotiables

Grouped into **Code**, **Design**, and **Process and safety**. These are short, testable sentences:

- "Inner layers never import outer ones."
- "No barrel files except where an interface boundary genuinely needs one."
- "Runtime code never reads from `.ai/`."
- "No comments unless asked, with two standing exceptions…"
- "Style with CSS Modules; reference semantic tokens. Never inline hex…"
- "Never fabricate commands, numbers, sources, or decisions — unknowns become `<!-- TODO -->` markers, `[unverified]` flags, or questions."
- "A new runtime `dependencies` entry needs an ADR."

### i) Key commands

A fenced block of the ten commands with one-line comments. No guessing, no `npm` vs `pnpm` mistakes, no invented flags.

**Why it matters:** an `AGENTS.md` written like this is *executable knowledge*. An `AGENTS.md` written as a wall of aspirational prose ("write clean code") is noise that consumes context.

**What to copy:** the nine-section shape. Even if your file is 60 lines, include scope, the git gate, precedence (even a 3-level version), the commands, and 5 non-negotiables.

---

## 6. Tool adapters: one source, many tools

Each AI tool historically demanded its own config file. The naive setup copies the same content into each, and they rot out of sync within weeks.

This repo uses **pointer-only adapters**. The evidence is in the file sizes:

| File | Bytes | Content |
|---|---|---|
| `CLAUDE.md` | 11 | `@AGENTS.md` |
| `.claude/skills` | 13 | symlink → `../.ai/skills` |
| `.claude/agents` | 13 | symlink → `../.ai/agents` |
| `.opencode/skills` | 13 | symlink → `../.ai/skills` |
| `.opencode/agents` | 13 | symlink → `../.ai/agents` |
| `.agents/skills` | 13 | symlink → `../.ai/skills` |

Eleven bytes. That is the entire Claude Code configuration.

`@AGENTS.md` is Claude Code's import syntax: "inline that file here". So Claude sees exactly what OpenCode sees, with no copy to maintain.

`CONTRIBUTING.md` documents the invocation difference per tool:

| Tool | Finds instructions / skills / agents | Invoke a role |
|---|---|---|
| Claude Code | `CLAUDE.md` → `AGENTS.md`; `.claude/skills`, `.claude/agents` | "use the designer agent" or `@agent-designer` |
| OpenCode | `AGENTS.md`; `.opencode/skills`, `.opencode/agents` | `@designer` |
| Codex / others | `AGENTS.md`; `.agents/skills` | "Act as the designer defined in `.ai/agents/designer.md`" |

And the caveat is recorded rather than hidden:

> "Symlinked adapters need `core.symlinks` on Windows."

**Why it matters:** duplicated instructions don't just waste space — they create *silent behavioral differences* between tools. The same agent will follow different rules in Cursor and in Claude, and you will blame the model.

**What to copy:**

```bash
# One source of truth
mkdir -p .ai/skills .ai/agents
# Point every tool at it
ln -s ../.ai/skills .claude/skills
ln -s ../.ai/agents .claude/agents
ln -s ../.ai/skills .opencode/skills
ln -s ../.ai/agents .opencode/agents
ln -s ../.ai/skills .agents/skills
# The one-line import
printf '@AGENTS.md\n' > CLAUDE.md
```

If symlinks are impractical on your OS, a tiny pointer file with a single link works too: *"See `.ai/skills/` for skills."*

---

# Part III — Knowledge management

## 7. Precedence: what wins when sources disagree

From `AGENTS.md` and `.ai/README.md`:

```
1. the author's explicit instructions
2. code and config
3. docs/
4. decisions/            (accepted ADRs)
5. accepted rfcs/
6. .ai/memories/
7. working memories
8. assumptions
```

Three rules ride along with it:

- **"Fix or flag the lower source and say so."** The agent is not allowed to silently pick. If `docs/` contradicts the code, it either fixes the doc or reports the conflict.
- **"An RFC that is not `Accepted` never overrides accepted knowledge."** Proposals are not decisions.
- **Assumptions are last** — which is a polite way of saying "if you had to assume, you are probably wrong; go check."

**Why it matters:** without an order, models resolve contradictions by whatever heuristic is cheapest — recency, or which text *sounds* more authoritative. Both are unreliable. With an order, the failure mode becomes visible: "code says X, docs say Y, docs lose — fixing docs."

**Why "code wins over docs" is the interesting choice:** documentation that has drifted is a *bug in the documentation*, not evidence about the system. Trusting the artifact that actually runs is the only defensible default. But note that the human's instruction outranks even the code — you can always say "yes, I know the code does X; do Y anyway."

**What to copy:** even a three-level version is worth having:

```
1. What I tell you in this session
2. What the code and config actually do
3. What the docs say
4. Your assumptions — never act on these without saying so
```

---

## 8. The knowledge taxonomy

The core insight of `.ai/README.md`: **different kinds of writing have different jobs**, and mixing them causes duplication. Each location has a *test question* you can answer in one sentence.

| Location | Holds | Test question |
|---|---|---|
| `docs/` | current behavior, rules, design system, procedures | *Would a human need it with no AI tooling?* |
| `decisions/` | accepted decisions and why (ADRs) | *Does it record **why** something was decided?* |
| `rfcs/` | proposals (`active/`) and closed ones (`archived/`) | *Is it proposed, not yet decided?* |
| `.ai/memories/` | short, present-state orientation for agents | *Does it help an agent decide, without being a contract or procedure?* |
| `.ai/memories/working/` | committed in-flight context | *Is it still unresolved?* |
| `.ai/skills/` | how to perform a reusable class of task | *Is it a repeatable workflow?* |
| `.ai/agents/` | role-specific behavior and boundaries | *Does it apply to one role only?* |
| `.ai/templates/` | templates for the above | — |
| `.ai/specs/` | local task specs (**gitignored**) | *Is it one deliverable's working spec?* |

Plus the rules that sit underneath the table:

- **One source of truth — link, don't copy.**
- **Memories describe the present, not history.** History is ADRs, RFCs, and git.
- **No secrets, tokens, or private hostnames.**
- **Runtime code never reads from `.ai/`.**

### A decision tree for "where does this go?"

```
Is it a fact about how the system works *now* (or a rule/procedure)?
├── Yes → docs/
└── No
    ├── Is it a record of a choice already made, with reasoning?   → decisions/ADR-NNN
    ├── Is it a proposal with open alternatives?                   → rfcs/active/RFC-NNN
    ├── Is it one deliverable's plan, not meant for history?       → .ai/specs/   (gitignored)
    ├── Is it temporary context that must survive a session?       → .ai/memories/working/
    ├── Is it short orientation that helps an agent decide?        → .ai/memories/
    ├── Is it a repeatable multi-step procedure?                   → .ai/skills/<name>/SKILL.md
    └── Is it behavior for one role only?                          → .ai/agents/<role>.md
```

**Why it matters:** the most common failure in AI-assisted repositories is **triplication** — the same explanation in `README.md`, in `AGENTS.md`, and in three chat sessions. Every copy is a future contradiction. The taxonomy gives you a place to put a fact *once* and link to it forever.

**What to copy:** run the decision tree on the next thing you are tempted to paste into your AI rules file. Half the time the answer is "that belongs in `docs/`".

---

## 9. Memories and working memories

### Memories: orientation, not documentation

A memory is short. It carries frontmatter with a **description** (when to read it) and a **`last-verified` date** (a staleness signal):

```yaml
---
description: Naming, imports, styling, and comment conventions. Read before
  writing any code or CSS.
last-verified: 2026-10-09
---
```

The five memories and their trigger conditions:

| Memory | When to read |
|---|---|
| `overview.md` | always, first |
| `architecture.md` | before changing structure, adding files, or moving code |
| `conventions.md` | before writing any code or CSS |
| `content-pipeline.md` | before touching the compiler, authoring CLI, or post assets |
| `development-workflow.md` | before starting and before handing off |

Each one is 20–40 lines and ends by pointing at the authoritative doc. Example, from `conventions.md`:

> "The rules are in `AGENTS.md#non-negotiables`, `docs/architecture.md#naming`, and `docs/design-system.md`. The practical reminders: …"

### The "decisions not to fix" pattern

This is the most underrated idea in the repository. `architecture.md` has a section literally titled **"Decisions not to 'fix'"**:

> - Drafts sit beside `raw/` and are excluded because the compiler globs `raw/**` only (ADR-001) — **do not add draft filtering to the compiler.**
> - Image filenames hash the **source** bytes + transform parameters, not the encoded output, so names are stable across macOS and CI (ADR-004).
> - Encoding concurrency and post fan-out are deliberately bounded (ADR-005, ADR-006); Shiki grammars load lazily.
> - The display face is a pixel font used only at 10px-grid sizes and never subset or modified (ADR-007).

Why this matters so much: an agent that sees "globs `raw/**`, drafts are excluded by accident" will helpfully add explicit draft filtering — a plausible improvement that duplicates an intentional design and quietly changes behavior. A line saying **"this is deliberate, here's the ADR, don't touch it"** prevents a whole class of confident sabotage.

**What to copy:** start a `docs/decisions-not-to-fix.md` (or a section) listing the three things about your project that look wrong but aren't, each with one sentence of why.

### Working memories: a buffer with a lifecycle

`.ai/memories/working/` holds committed, temporary context — investigations, hypotheses, open questions that must survive between sessions or between tools. The README defines the lifecycle:

- **Resolve before the linked change lands:** promote the durable parts, then **delete the file**.
- **Promotion targets:** current behavior → `docs/`; accepted decisions → `decisions/`; proposals → `rfcs/active/`; durable agent context → `.ai/memories/`; reusable workflows → `.ai/skills/`.
- **Idle for 30 days → promote, update, or delete.**

Template:

```yaml
---
description: <what is being investigated or built>
status: active          # active | blocked | ready-to-promote
related: <branch, issue, RFC, or spec>
created: YYYY-MM-DD
---
```

with sections: Context / Findings & hypotheses / Open questions / **To promote on resolution**.

Note the design: working memories are *committed* (unlike specs) because they must survive switching machines or tools mid-task.

**Why it matters:** AI sessions end. Without a buffer, you either lose the thread or keep a growing pile of stale notes that poison future context. The 30-day rule and the "promote then delete" step turn a junk drawer into a working memory.

**What to copy:** the two-file discipline — `memories/` for durable orientation, `memories/working/` for in-flight — and the rule that a working note is deleted when its change lands.

---

# Part IV — Capabilities

## 10. Skills: packaging a repeatable workflow

A **skill** is a folder with a `SKILL.md` that teaches the agent how to perform one class of task. It is the difference between "please write a design brief" (reinvented every time, differently) and a repeatable procedure with a named output.

The repository uses the open `SKILL.md` format, with frontmatter restricted to `name` and `description`, "the open skills format that Claude Code, OpenCode, Codex, and others discover." `templates/skill.md` says why:

> "Keep it to `name` and `description`."

Tool-specific fields would break portability — the same principle as pointer-only adapters.

### Anatomy

```markdown
---
name: <kebab-name>
description: Use when <situation>. One or two sentences.
---

# <kebab-name>

## Prerequisites      ← files to read before starting
## Workflow           ← numbered steps
## Constraints        ← what NOT to do
## Useful commands
## Expected output    ← the artifact
## References         ← links; never duplicate content
```

### The description is a routing instruction

Look at the real one:

> "Use when a feature needs UI — a page, component, variant, interaction, or visual change. Produces a design brief built from the existing design system and runs the conformance checklist so the phosphor/CRT identity does not drift."

The description is often the **only** text the model sees when deciding whether to load the skill. So it must say *when to use it*, not *what it is*. "Design system helper" would never fire; "Use when a feature needs UI — a page, component, variant…" fires reliably because it lists the trigger words.

**Rule of thumb:** write the description as `Use when <trigger situation>. <what it produces>.`

### Progressive disclosure

The `post-authoring` skill is the best example. It does not contain five workflows; it is a dispatcher:

```
skills/post-authoring/
  SKILL.md              ← ~50 lines: check the gate, then route
  stages/
    brief.md
    research.md
    outline-critique.md
    editorial.md
    preflight.md
```

`SKILL.md` says:

> "1. Read `post.md` frontmatter `stage` and confirm the previous gate … is met. If it is not, stop and say what is missing.
> 2. Load the prompt for the stage and follow it exactly: …"

Only one stage file is ever loaded. This is **progressive disclosure**: an index, then exactly one branch. It keeps the context window free for the actual work.

**Why it matters:** context is a budget, and irrelevant instructions actively hurt — models attend to everything on the desk, including the parts that don't apply. A 2,000-line mega-skill makes the model worse at the one thing you asked.

### The stage-prompt skeleton

Every stage file in `post-authoring/stages/` uses the same five sections. This is the most reusable pattern in the repository:

```markdown
# Stage N — <Name>

## Role
You are a **<specific job title>**. <one sentence of posture>

## Inputs
- <exact file(s)>

## Instructions
1. <numbered, concrete steps>
2. ...

## Output contract
Write `<file>`:
<literal markdown template of the artifact>

## Guardrails
- **Never** <the expensive mistake>.
- <what to do instead>
```

Three properties make it work:

1. **A literal output template.** `brief.md`'s contract is not "write a brief" — it is the exact headings:

   ```md
   # Brief — <working title>

   - **Audience:** who this is for, and what they already know
   - **Angle:** the specific lens (not "an intro to X")
   - **Takeaway:** the one thing the reader keeps
   - **Why now:** what makes this worth publishing today
   - **Target length:** <n> words (~<m> min)
   - **Non-goals:** what this post deliberately does not cover
   ```

   Models follow shapes far more reliably than adjectives. "One screen" is a wish; a six-bullet template is a result.

2. **Negative guardrails.** Not "be careful" but: "**Never assert a fact you did not verify.** Tag it `[unverified]`." "Do not write post prose — this artifact is a plan." "Propose, do not rewrite."

3. **A role that constrains ambition.** "You are a **structural editor**… your job is to stress-test its argument and flow — not to rewrite it." A role is a cheap way to set the *scope* of the work, and it pairs with the boundary sections in the agent files.

**What to copy:** the five-section skeleton for any repeated task. Even a 15-line skill ("how we add a database migration here") pays for itself by the third use.

---

## 11. Agents: roles with boundaries

An **agent** file defines *who* is doing the work; a **skill** defines *how* to do a class of work. The repository has five agents, all with the same skeleton:

```markdown
---
name: <role>
description: Use when <role applies>. One or two sentences.
---

# <Role>

## Responsibilities
## Boundaries            ← what this role must not do
## Applicable skills
## When invoked and handoff
## Expected outputs
## References
```

### The five roles

| Agent | Job | Hard boundaries |
|---|---|---|
| **documenter** | turn requests into specs, RFCs, ADRs; keep knowledge in sync | no production code; never fabricate commands/numbers/sources; link instead of duplicating |
| **designer** | design UI from the existing system; produce a design brief; audit drift | no values outside the token system; no new typeface/accent without an RFC; no business logic |
| **implementer** | write code + tests to an agreed spec | no invented behavior (ambiguous spec → stop and ask); no architecture changes; never weaken tests or lint to go green |
| **reviewer** | definition-of-done check against `AGENTS.md`, `docs/`, ADRs | runs **only when asked**; requests changes, does not rewrite them; judges the diff, not intent |
| **editor** | run assisted stages of the writing pipeline | never edits `post.md` in place; never writes the draft or resolves edits; never publishes |

And the flow, from `.ai/README.md`:

```
documenter (spec/RFC) → designer (UI brief, if UI) → implementer → AUTHOR REVIEW
    → reviewer (on request) → commit (only when the author asks)

Posts: editor ↔ author
```

### Two things that make this more than roleplay

**1. Boundaries are specific and operational.**

Compare:
- Weak: "Be careful not to introduce unnecessary dependencies."
- Actual: "**A new runtime `dependencies` entry needs an ADR.**"

- Weak: "Preserve the author's voice."
- Actual: "**Never edit `post.md` in place.** Output is a snapshot plus a diff."

- Weak: "Ask if unsure."
- Actual: "**A missing or ambiguous spec means stop and ask** — or hand to the documenter."

The boundary tells the agent what to do *at the moment of temptation*, which is the only moment that matters.

**2. Handoffs are typed.**

Every agent names its input and its output:

> **implementer** — Input: an agreed spec (`.ai/specs/`), an accepted RFC, or an explicit task. Handoff: to the author with a summary of what changed, what was verified, and any drift found.
>
> **reviewer** — Input: the diff (`git diff` or `git diff main...HEAD`) and the spec/RFC/task it claims to implement. Output: a verdict (approve / request changes) with findings ordered by severity, each citing `file:line` and the rule, doc section, or ADR it violates.

So agents are not launched to "help". They are launched to produce a named artifact for the next step. That is what makes a multi-agent pipeline dependable instead of decorative.

### The reviewer is deliberately opt-in

`ADR-008` is explicit:

> "The reviewer agent runs on request … a two-step gate is overhead for a solo repository."

And `reviewer.md` repeats: "Runs **only when the author asks**; it is not a mandatory gate."

**Why it matters:** process that exceeds the team's size becomes theatre. A five-person team might make review mandatory; a solo author with a pre-push hook and a diff review already in the loop does not need two gates. Copy the *shape*, then set the threshold for your reality.

### Why "reviewer judges the diff, not intent"

> "Judges what the diff and docs show, not intent."

This is a subtle but excellent calibration. Models are agreeable; told "I intended to follow the layer rules", a reviewer will often accept the claim. Restricting it to observable evidence makes the review falsifiable.

**What to copy:** start with two agents — **implementer** and **reviewer**. Add a *documenter* when your specs start living in chat logs. Add a *designer* only if you have a real design system to protect.

---

# Part V — Process and safety

## 12. Spec-driven development

The term sounds heavy. In this repository it is a **threshold plus a loop**.

### The threshold

From `AGENTS.md`, the trigger is an **architectural or design change** — one that alters:

- module boundaries,
- the content pipeline,
- cross-cutting patterns,
- major dependencies,
- the design system's foundations,
- or otherwise involves meaningful trade-offs.

Everything else — "routine implementation, small changes, and bug fixes" — needs no spec. UI work starts from a design brief instead.

### The loop

1. **Require a spec.** Ask the author for one, or draft it for them to edit, from `templates/spec.md`, saved to `.ai/specs/` (**local, gitignored**).
2. **Clarify** until ambiguities are resolved.
3. **RFC** when there are real alternatives.
4. **Task breakdown** into ordered, actionable tasks.
5. **Confirm with the author before writing code.**

Step 5 is the one people skip. It is also where most wasted work is avoided: the agent has already read everything and formed a plan, and you get to correct the plan for the price of reading a page — instead of correcting the implementation.

### The spec template

```markdown
# Feature/Task Specification

## 1. Objective
_What this change achieves and why it is needed._

## 2. Scope
_What is included, and what is explicitly out of scope._

## 3. Acceptance Criteria
_Specific, checkable requirements for completion._
- [ ] Criterion 1
- [ ] Criterion 2

## 4. Design / Contracts (if applicable)
_Interfaces, types, ports, or boundaries introduced or changed._

## 5. Data / Content Model (if applicable)
_Schema, frontmatter, content pipeline, or generated-output changes._

## 6. UI / Design (if applicable)
_Which tokens and primitives the feature uses, and any new token, variant, or
component it needs — each justified against the extension rules._

## 7. Business Logic / Domain Rules
_Core logic, constraints, error handling._

## 8. Risks & Alternatives (if applicable)
_What could go wrong, and the options considered._
```

Two details worth stealing:

- **Acceptance criteria are checkboxes**, not prose. The implementer's "Expected outputs" and the reviewer's job both key off them.
- **Scope includes what is out of scope.** This is the cheapest defense against scope creep, which agents are prone to because "while I was there I also…" feels helpful.

### Why the spec is gitignored

`.ai/specs/` is in `.gitignore`. A spec is one deliverable's working plan; once the work lands, the durable knowledge lives in `docs/`/ADRs and the plan itself is noise in history. Working memories are committed precisely because they *must* survive a session; specs are local because they are scaffolding.

**Why it matters:** without a written spec, "the agent misunderstood me" is unfalsifiable, and the rework is invisible. With one, ambiguity surfaces *before* code.

**What to copy:** the threshold sentence and the spec template. Even if you only use it for changes over ~2 hours, it will pay for itself.

---

## 13. Spec → RFC → ADR → docs → archive

Three documents, three jobs, one lifecycle. Getting this straight removes most confusion about "where do I write things down?".

| Document | Answers | Lives | Lifetime |
|---|---|---|---|
| **Spec** | What are we building, and how will we know it's done? | `.ai/specs/` (gitignored) | Until the change lands |
| **RFC** | What are the options, and which do we propose? | `rfcs/active/` → `rfcs/archived/` | Until implemented |
| **ADR** | What did we decide, and why? | `decisions/` | Permanent (superseded, never rewritten) |

### The RFC template

````markdown
# RFC-NNN — <Title>

- **Status:** Draft
- **Date:** YYYY-MM-DD
- **Supersedes:** —
- **Superseded by:** —

## Context
_What forces are at play. What problem are we solving, and why now._

## Options

### Option A — <name>
- **Pros:** …
- **Cons:** …

### Option B — <name>
- **Pros:** …
- **Cons:** …

## Decision
_The option chosen and the reasoning. One decision per RFC._

## Consequences
_Positive, negative, and follow-up work once implemented._
````

### The ADR template

````markdown
# ADR-NNN — <Title>

- **Status:** Accepted
- **Date:** YYYY-MM-DD
- **Topic:** <module or area>
- **RFC:** [RFC-NNN](../rfcs/archived/RFC-NNN-<kebab>.md) <!-- or — -->

## Context
_What forced the decision._

## Decision
_What was decided, precisely._

## Options considered
_Optional: the alternatives, especially when the RFC step was waived._

## Rationale
_Why this over the alternatives._

## Consequences
_What changed, follow-ups, and what to watch out for._
````

### The lifecycle rules

From `decisions/README.md`:

- **Naming:** `ADR-NNN-<kebab-case-title>.md`; next free number, **never reused**.
- **Self-contained:** "a reader with only the ADR understands the decision and why."
- **Immutable once accepted:** "a later decision supersedes an ADR with a new ADR; the old one only gains `Status: Superseded by ADR-NNN`."

From the `write-an-rfc` skill:

1. **Route the change:** real alternatives → **RFC**; a hard-to-reverse choice without meaningful alternatives (new runtime dependency, hosting change, layer-rule change) → **ADR directly**; otherwise a **spec** is enough.
2. Draft in `rfcs/active/RFC-NNN-<kebab>.md` with the next free number, add a row to the index in `rfcs/README.md`.
3. **One decision per RFC.** Split independent decisions.
4. Status `Draft` → `Proposed` when ready; **the author moves it to `Accepted`. Do not implement before `Accepted`.**
5. **On ship:** fold current-state facts into `docs/` → write the ADR → cross-link RFC ↔ ADR → set the RFC to `Implemented` → move it to `rfcs/archived/` → update both indexes.

### What a real one looks like

`RFC-008` (which produced `ADR-008`) is worth reading as a model. Its Context is a list of *observable* problems, not opinions:

> "Deleting `.ai/` would have lost the decision history and broken `pnpm author:new`. Decisions were written up to three times (RFC, a history-style memory entry, and `docs/`)."

Its Options are real, with honest cons:

- **Option A —** keep everything in `.ai/`, add adapters: "smallest diff" but "fails the 'stands alone without AI tooling' test".
- **Option B —** project knowledge under `docs/`: "tidy root" but "mixes current-state docs with historical records in one tree".
- **Option C —** root-level `rfcs/` + `decisions/`, docs for current state, `.ai/` consumes: chosen, with its costs listed ("larger one-time migration with link fixes; symlinks need `core.symlinks` on Windows").

And the accepted RFC records an amendment made at acceptance:

> "**Amended on acceptance (2026-10-09, by the author):** … a new runtime `dependencies` entry requires an ADR …"

Note what the option set does for you: it forces the *rejected* paths to be documented. Six months later, "why not just put docs under `docs/`?" is already answered.

### The escape hatch that still produces knowledge

> "The author may waive the RFC step for a decision they make directly (for example in a design review); the ADR then records the waiver and the options that were considered."

Even skipping the process leaves a trace. That is the mark of a process designed by someone who has watched process get skipped.

**Why it matters for AI work specifically:** an agent will happily implement a plausible design and move on. Six weeks later the chat log is gone and every choice is folklore. The RFC forces *alternatives to exist*; the ADR forces the *reason to survive*. That is the difference between a codebase you can still reason about and one you can only archaeology.

**What to copy:** `decisions/` with an index, and the rule "new dependency ⇒ ADR". You do not need RFCs on day one.

---

## 14. Guardrails and verification

Guardrails are the rules that stop an agent from turning a small mistake into an expensive one. Collected from across the repo, by category:

### A. Git and human control

- Agents **never** run `git add`, `git commit`, `git push`, or anything that stages (`git mv`, `git rm`) unless explicitly asked.
- "After verification, stop and hand the diff to the author."
- "An approval covers the reviewed diff only" — approving one change does not authorize the next.
- One logical change at a time.
- The reviewer runs on request.

Why: a commit is cheap to make and expensive to untangle; a push is worse. The gate costs one sentence per session and removes the entire category of "the agent committed my broken work at 2am".

### B. Truthfulness

- "**Never fabricate commands, numbers, sources, or decisions** — unknowns become `<!-- TODO -->` markers, `[unverified]` flags, or questions."
- Research stage: "**Never assert a fact you did not verify.** Tag it `[unverified]`."
- "No fabricated links, papers, or statistics."
- Reviewer checks: "missing tests for new logic, weakened assertions, unexplained lint disables."

Why: models produce fluent, confident, wrong output. A convention that says "the unknown has a *name*" (`[unverified]`, `<!-- TODO -->`) makes uncertainty **visible and greppable** instead of invisible and dangerous.

### C. Scope and architecture

- A new **runtime** `dependencies` entry always needs an ADR.
- "Never weaken a rule to go green." An `oxlint-disable` needs the specific rule and a reason.
- Never hand-edit generated output (`content/compiled/`, `public/posts/`, `out/`).
- "Does not invent behavior: a missing or ambiguous spec means stop and ask."

### D. Secrets

- "No secrets, tokens, credentials, or private hostnames in any file" — including docs and memories.
- `.claude/settings.local.json` is gitignored (per-machine), while shared adapters are committed.

Why the split matters: an AI config folder is a very easy place to paste an API key "just for testing". Naming it as a rule is cheap insurance.

### E. Verification commands

- `pnpm check:lint` and `pnpm test` before finishing; `pnpm build` when the compiler, content, or app changed.
- A **versioned** `.githooks/pre-push` runs lint and test:

  ```bash
  #!/usr/bin/env bash
  # pre-push: run the local quality gate before code leaves the machine.
  # Installed by `pnpm install` (prepare: git config core.hooksPath .githooks).
  set -euo pipefail
  cd "$(git rev-parse --show-toplevel)"
  echo "pre-push: running pnpm check:lint and pnpm test..."
  if ! pnpm check:lint || ! pnpm test; then
    echo >&2
    echo "pre-push: checks failed; push aborted. Fix the issues above (pnpm format fixes formatting)." >&2
    exit 1
  fi
  ```

  Installed automatically by `pnpm install` via `git config core.hooksPath .githooks`, so it is not a per-machine ritual. Emergency bypass is documented, and "CI runs the same gates".

Why versioned hooks matter: the agent cannot "forget" to run the checks. The gate lives in the repository, not in the prompt.

### F. Drift

The `knowledge-drift-sync` skill runs before handoff:

1. List what changed in behavior or structure.
2. For each, find its canonical home in the maintenance table and compare:
   - code changed behavior → **update the doc in the same change**;
   - a doc changed → the code implements it, or the doc change is reverted;
   - a lint-enforced rule changed → the lint config and the docs agree.
3. Decisions: shipped RFC → ADR, `Implemented`, moved to `archived/`, indexes updated. New runtime dependency → ADR.
4. Releases: changeset with the right bump.
5. Memories: update `last-verified`; fix or delete stale statements.
6. Resolve working memories tied to the change.
7. **Every relative link in changed `*.md` files resolves.**
8. **Report what was synchronized, or state "no drift" with the evidence.**

Step 8 is the quiet gem: a report must be *evidence*, not a feeling. "No drift" with nothing to back it is not accepted.

### G. Definition of done

`.github/PULL_REQUEST_TEMPLATE.md` turns all of the above into a checklist: lint/test/build pass, layer rules hold, design system followed, docs match, shipped RFCs have ADRs, changeset included, new dependency has an ADR, memories updated, tests for new behavior, no secrets, no hand-edited generated output.

**Why it matters:** an LLM will satisfy exactly the criteria you make checkable. "Write good code" is unenforceable; "`pnpm check:lint` and `pnpm test` pass" is a boolean.

**What to copy, in order of value:**
1. The git approval gate in `AGENTS.md`.
2. A pre-push hook (or CI job) that runs your lint + tests.
3. The no-fabrication convention with a written marker for unknowns.
4. "New dependency ⇒ ADR."

---

## 15. The nine-stage writing pipeline

If you take one pattern from this whole repository, take this one. It is the most transferable idea for anyone using AI to produce *prose* (or any artifact where voice and judgment matter).

### The table (from `docs/authoring.md`)

| # | Stage | Owner | Artifact | Assistance |
|---|---|---|---|---|
| 0 | Seed | Author | `post.md` | none |
| 1 | Brief | Author seeds, AI expands | `brief.md` | expand into a tight brief |
| 2 | Research pack | AI drafts, author curates | `research.md` | cheatsheet + checklist + open questions |
| 3 | Outline | Author | `snapshots/outline-<date>.md` | critique structure, flag gaps |
| 4 | Draft | Author | `post.md` | **none — the voice stays the author's** |
| 5 | Editorial | Lead editor | `snapshots/editorial-<date>.md` + `.diff.md` | structural → line → copy pass |
| 6 | Resolve | Author | `post.md` | none |
| 7 | Preflight | Editor + CLI | `snapshots/preflight-<date>.md` | mechanical + content audit |
| 8 | Publish | Author | `content/raw/<slug>/` | none |

### The state machine

`post.md` frontmatter carries a `stage` field:

```
seed → brief → research → outline → draft → review → resolve → preflight → ready
```

State lives **in the file**, not in the chat. Any tool, any session, any day: read `stage`, know where you are. That is what makes the pipeline survive switching between Claude and OpenCode, or between yesterday and next month.

### The gates

A stage is complete when its artifact exists and the next stage has what it needs. "Do not skip a gate." The conditions are concrete:

- **0 → 1** `post.md` states subject, audience, and bullet-level intent.
- **1 → 2** `brief.md` has angle, takeaway, why-now, non-goals, target length.
- **2 → 3** `research.md` answers the brief's open questions **with sources**.
- **3 → 4** outline has an `h2`/`h3` skeleton with per-section intent, and the critique is addressed.
- **4 → 5** `post.md` is a complete rough draft (starting at `h2`).
- **5 → 6** `editorial-*.diff.md` lists every proposed change with rationale; the author resolves each.
- **6 → 7** `pnpm author:preflight <slug>` exits clean and the audit is recorded.
- **7 → 8** `pnpm author:publish <slug>` moves the post into `content/raw/`.

Before starting a stage, the editor skill checks the previous gate and **stops, saying what is missing** if it is unmet. That single behaviour prevents the classic AI failure of producing a polished artifact from an incomplete premise.

### The mechanisms worth stealing

**1. The AI never writes the draft.**

Stage 4 is explicitly "none — the voice stays the author's", and stage 6 (resolving edits) is the author's too. AI is at the *edges*: expand inputs (brief, research), critique outputs (outline, editorial), verify (preflight). Human in the middle.

**2. Editorial output is a snapshot plus a diff, never an in-place edit.**

> "**Never edit `post.md` in place.** Output is a snapshot + diff only."

Every note carries a severity: `blocker` (must fix: factual error, broken logic), `should` (clear improvement; address or consciously decline), `nit` (taste). The author resolves each one in stage 6.

This is the single most important writing-with-AI practice in the repository. In-place rewriting destroys the author's ability to *see* what changed, and it destroys voice gradually — each individual edit looks fine.

**3. Facts must trace to the research pack.**

> "Flag every claim not backed by `research.md` as `[unverified]`."

The research stage is where verification happens, once, with sources. The editorial stage is then not allowed to introduce new facts — only to reorganize and tighten. That separation is what stops an editor-model from "improving" a sentence by inventing a statistic.

**4. Mechanical checks vs. judgment**

`pnpm author:preflight <slug>` covers what regexes can:

- `[auto]` frontmatter: `title`, ISO `date`, `description` 50–160 chars, non-empty `tags`, `stage: ready`, no leftover draft keys;
- `[auto]` body: starts at `h2`, relative images exist, every code fence declares a language, every image has alt text, heading anchors don't collide;
- `[auto]` uniqueness: slug doesn't collide with a published post or another draft.

The model covers what regexes cannot: claims sourced, links point where they claim, code is runnable, alt text describes purpose, description reads well as an excerpt, tags consistent.

And the instruction is explicit: "The command catches what regexes can, you catch what they cannot." Plus: "If the CLI fails, do not paper over it — report the failure."

**5. Drafts are excluded by construction, not by code**

Drafts live in `content/drafts/<slug>/`, beside `raw/`. The compiler globs `raw/**` **only**, so drafts can never leak into a build — with zero filtering logic. And a memory explicitly records: *do not add draft filtering*. Two lines of filesystem layout replace a whole class of "oops, the draft got published" bugs.

**6. Publishing is a human command**

AI has no write access to `content/raw/`, `content/compiled/`, or `public/`. `pnpm author:publish` is run by the author. The entire pipeline is designed so the irreversible step is deliberate.

### Why it matters

This is a working answer to "how do I use AI to write without sounding like AI?" The answer isn't a better prompt — it's **structural**: the AI never touches the draft, only produces advisory artifacts around it, each with a severity and a rationale, and the human resolves them. The output stays yours because the machine never held the pen.

The same shape works for anything where judgment matters: design docs, ADRs, performance analysis, hiring feedback. Generate the inputs, critique the output, verify mechanically, keep the writing.

**What to copy:** the stage `stage:` field, the "snapshot + diff, never in-place" rule, the severity labels, and the mechanical-vs-judgment split. Those four work even without the rest of the process.

---

# Part VI — Practice

## 16. The maturity ladder

Do not build this repository's setup on day one. It evolved over many changes, and each piece was added in response to a specific pain. Add it in the same order.

### Level 0 — Start today (about 5 files, 30 minutes)

1. **`AGENTS.md`** — what the project is, how to run it, 5–10 non-negotiables, and the git approval gate. (Appendix A.)
2. **`CLAUDE.md`** — one line: `@AGENTS.md`.
3. **`README.md`** — for humans. Most projects already have one.
4. **`docs/`** — one file describing how the system works now.
5. **`decisions/`** + one real **ADR** for a choice you already made.

At this level you get: no re-explaining the project every session, no surprise commits, and one choice documented.

### Level 1 — Add when you feel the pain

- **`.ai/memories/overview.md` + `architecture.md`** with `last-verified` dates — when you notice yourself re-explaining the codebase every session.
- **`.ai/skills/<name>/SKILL.md`** — the second time you do the same multi-step task (deploy, add a migration, cut a release).
- **`.ai/templates/spec.md`** — when a change is big enough that you would otherwise paste a plan into chat and lose it.
- **A "decisions not to fix" section** — the first time the agent "improves" something intentional.

### Level 2 — Add when work gets risky

- **RFC/ADR lifecycle** with `rfcs/active/` and `rfcs/archived/` — when you start weighing real alternatives.
- **Precedence order** — the first time two sources contradict each other.
- **Drift-sync skill + knowledge-maintenance table** — when docs start lying.
- **Pre-push hook / CI gate** — the first time a broken change almost shipped.
- **PR definition of done** — when you have collaborators, or when you keep forgetting a step.

### Level 3 — Add when collaboration or scale demands it

- Role agents (implementer, reviewer, documenter) and typed handoffs.
- Working memories with a lifecycle.
- Changesets / release gating.
- Pointer-only adapters for more than one tool.

**Rule:** add a mechanism the fifth time you feel the pain, not the first time you read about it. The repository's own ADR-008 rejected a mandatory two-step review gate as "overhead for a solo repository" — that instinct is correct, and it applies to every layer above.

---

## 17. Ten rules of thumb

1. **If a human needs it, it is not AI-only.** Keep project knowledge in `docs/`, `decisions/`, `rfcs/`.
2. **One instruction source; everything else points at it.** 11 bytes beats two copies.
3. **Write the precedence order before the first conflict.** "Code beats docs; flag the loser."
4. **Link, don't copy.** Every duplicate is a future contradiction.
5. **Every workflow gets a named artifact and a literal output template.**
6. **Say "never" out loud** — especially around git, publishing, generated files, and dependencies.
7. **Put the human on every irreversible step.**
8. **Update docs in the same change**, or accept they will be wrong forever.
9. **Give unknowns a name** (`<!-- TODO -->`, `[unverified]`) instead of letting the model fill the gap.
10. **Never let runtime code read your AI config.** It is the fastest way to make your project unbuildable without a vendor tool.

---

## 18. Start-today checklist

- [ ] Create `AGENTS.md` from Appendix A; fill in the four placeholders (project name, what it is, commands, non-negotiables).
- [ ] `printf '@AGENTS.md\n' > CLAUDE.md`.
- [ ] Add the git gate, verbatim: *"Do not run `git add`, `git commit`, or `git push` unless I explicitly ask. Verify, then hand me the diff."*
- [ ] Write `docs/overview.md` in ten lines: what it is, how it runs, where things live.
- [ ] Create `decisions/README.md` and one ADR for a decision you have already made.
- [ ] Add a "Do not fix" list with one entry.
- [ ] Ask your agent to *read* `AGENTS.md` at the start of the next session, and to summarize the project back to you. Fix whatever it got wrong — that is your documentation gap.
- [ ] Install a pre-push hook (or a CI job) that runs your lint + tests.
- [ ] Next time you repeat a multi-step task, write it as a skill.

Two habits matter more than any file:

- **When the agent gets something wrong, ask "what was missing from my instructions?"** and fix the instruction, not just the code. That is how the layer grows.
- **Never let it commit without showing you the diff.** Read the diff. Every time. This is the whole safety model in one habit.

---

## 19. Anti-patterns

These are the failure modes this repository's design is specifically built to prevent.

| Anti-pattern | What goes wrong | The fix used here |
|---|---|---|
| Copy-pasting rules into `CLAUDE.md`, `.cursor/rules`, and `AGENTS.md` | Silent divergence; the agent behaves differently per tool | Pointer-only adapters; `CLAUDE.md` = `@AGENTS.md` |
| Everything lives in `.ai/` | Deleting the AI folder breaks the build; humans can't find decisions | Stand-alone test; runtime never reads `.ai/` |
| One giant rules file | Burns context; irrelevant rules degrade attention | Index + progressive disclosure (skills with stage files) |
| Docs updated "later" | Docs become fiction; agent trusts the fiction | Drift sync in the same change; maintenance table |
| No precedence rule | Agent picks whichever source it read last | Explicit 8-level order |
| "Be careful" instructions | Ignored at the moment of temptation | Named prohibitions + a positive alternative |
| Agent commits/pushes to be helpful | Irreversible surprises, unreviewed diffs | Approval gate; human publishes |
| AI rewrites your prose in place | Voice erodes one reasonable edit at a time; change is invisible | Snapshot + diff; author resolves |
| Spec-less architecture changes | Plausible design, no record of alternatives or reasons | Spec → RFC → ADR |
| Weakening a lint rule to go green | The boundary quietly disappears | "Never weaken a rule"; disables need rule + reason |
| Stale "memory" files that are really history | Context poisoned with outdated claims | Memories are present-tense; history lives in ADRs/git |
| AI config holds secrets | Leaked keys | No-secrets rule; `settings.local.json` gitignored |
| Hand-editing generated output | Lost on next build; confusing diffs | Generated paths are read-only by rule |
| Process bigger than the team | Ceremony, then abandonment | Reviewer opt-in; routine changes need no spec |

---

## 20. What this repository deliberately omits

Worth stating, so you don't assume you're missing something:

- **No MCP servers.** No `.mcp.json`, no external tool integrations. The agent uses files, shell, and git — nothing else.
- **No model or provider configuration.** No "use model X for role Y". The process is model-agnostic.
- **No custom slash commands** or per-tool workflows.
- **No automation that "runs the agents".** There is no CI job that invokes an agent. AI work is interactive and human-initiated.
- **No per-tool rule content** — only pointers.
- **No prompt library beyond skills.** Four skills, five agents, done.

The absence is the point: every added surface is something to maintain. This setup is ~20 markdown files and two symlinks, and it covers all four problems from §2.

---

## 21. Glossary

**Agent** — a model plus tools plus a loop: it can read, edit, run commands, and iterate toward a goal. Distinct from a chat model that only replies.

**Subagent** — a separate agent session with a fresh context, given one bounded job, returning a result. Used to keep the main context clean and to enforce role boundaries.

**Harness** — the program around the model that provides tools, permission prompts, and context injection (e.g. the coding CLI you are using). It decides what the model can *do*.

**Context window** — the finite text budget the model can see at once. Everything it knows about your task must fit. Filling it with irrelevant rules makes it worse, not better.

**Progressive disclosure** — loading an index first, then only the branch of knowledge the current task needs. Skills with stage files are the canonical example.

**Skill** — a folder with `SKILL.md` describing how to perform one reusable class of task, with triggers, prerequisites, workflow, constraints, and an expected output.

**Tool adapter** — a thin, tool-specific pointer (a one-line `CLAUDE.md`, a symlink) that routes a given AI tool to the single source of truth. Contains no content of its own.

**Instruction source** — the one file an agent is told to read first (`AGENTS.md` here).

**Guardrail** — a hard rule that prevents a class of expensive mistake: the git gate, no-fabrication, no secrets, don't weaken tests.

**Precedence order** — the written ranking that resolves contradictions between sources.

**Memory** — short, present-tense orientation for an agent, with a `last-verified` date, linking out to authoritative docs. Not documentation.

**Working memory** — committed, temporary in-flight context with a lifecycle: promote the durable parts, then delete.

**Spec** — one deliverable's plan: objective, scope, acceptance criteria, contracts, risks. Local and gitignored here.

**RFC (Request for Comments)** — a proposal that lays out context, options with pros/cons, a proposed decision, and consequences. Not binding until accepted.

**ADR (Architecture Decision Record)** — a permanent, self-contained record of a decision: context, decision, options considered, rationale, consequences. Immutable; superseded rather than rewritten.

**Drift** — the divergence between what the docs say and what the code does. The silent killer of AI-assisted work.

**Definition of done** — the checklist that must be true before a change is considered complete (lint, tests, docs, ADR, changeset, no secrets).

**Hallucination / fabrication** — fluent, confident, wrong output: invented APIs, numbers, citations, file paths. Countered by verification, `[unverified]` markers, and "never fabricate" rules.

**`[unverified]`** — this repo's convention for naming an unverified claim so a human can find it later instead of trusting it.

**Approval gate** — the rule that irreversible actions (stage/commit/push/publish) require an explicit human request. Approval covers only the diff reviewed.

**Changeset** — a file describing a change and its semver bump, used to drive releases. Internal-only changes get an empty one.

**Wireit / task caching** — a build-task runner that skips work whose inputs are unchanged. Relevant here because "lint passed" can mean "skipped, nothing changed".

---

# Appendix A: Starter `AGENTS.md`

A generic, copy-paste version. Replace the `<...>` placeholders, delete what doesn't apply, and keep it under ~100 lines. Its job is to be **true and executable**, not complete.

````markdown
# AGENTS.md

The only instruction source for AI agents working on <PROJECT NAME>. Every tool
reads this file (Claude Code through `CLAUDE.md` → `@AGENTS.md`); tool folders
only point here. Read this first, then follow the links.

## Approval gate — never stage or commit on your own

**Do not run `git add`, `git commit`, or `git push` (or anything that stages,
such as `git mv` or `git rm`) unless I explicitly ask.** After verifying, stop
and hand me the diff. An approval covers the reviewed diff only.

## What this project is

<One or two sentences. What it does, for whom, and how it is built.>

## Commands

```sh
<install>       # install dependencies
<dev>           # run locally
<test>          # run the test suite
<lint>          # lint / typecheck
<build>         # production build
```

Always use these commands — do not invent alternatives or flags.

## Precedence

When sources disagree, trust them in this order (highest first):

1. my explicit instructions;
2. code and config;
3. `docs/`;
4. `decisions/` (accepted ADRs);
5. accepted `rfcs/`;
6. `.ai/memories/`;
7. your assumptions.

Fix or flag the lower source and say so. Never silently pick one.

## Where things live

| Topic                                  | Location            |
| -------------------------------------- | ------------------- |
| How it works now, and the rules        | `docs/`             |
| Why a decision was made (ADRs)         | `decisions/`        |
| Proposals not yet decided              | `rfcs/`             |
| Agent orientation, skills, roles       | `.ai/`              |
| Local task specs (gitignored)          | `.ai/specs/`        |

Project knowledge never lives only in `.ai/`. Runtime code never reads `.ai/`.

## Spec-driven development

For an **architectural or design change** — module boundaries, data model,
cross-cutting patterns, major dependencies, or a decision with real trade-offs:

1. write a spec from `.ai/templates/spec.md` into `.ai/specs/` (local);
2. ask me about anything ambiguous;
3. write an RFC from `.ai/templates/rfc.md` if there are real alternatives;
4. break it into ordered tasks;
5. **confirm with me before writing code.**

Routine changes, small fixes, and bug fixes need no spec. When in doubt, ask.

## Non-negotiables

**Code**

- <e.g. keep business logic out of route handlers>
- <e.g. no new runtime dependency without an ADR>
- <e.g. no comments unless asked>
- Files are kebab-case; component identifiers are PascalCase.

**Safety and truth**

- Never fabricate commands, numbers, sources, or decisions. Unknowns become
  `<!-- TODO -->`, `[unverified]`, or a question to me.
- No secrets, tokens, credentials, or private hostnames in any file.
- Never hand-edit generated output: `<generated paths>`.
- Never weaken a test or lint rule to make something pass. A suppression needs
  the specific rule and a reason.
- One logical change at a time.

## Before you finish

- Run `<lint>` and `<test>`; `<build>` when the app or build config changed.
- Update the doc that the change contradicts, **in the same change**.
- If a decision was made, record an ADR in `decisions/`.
- Report: files changed, what was verified, anything you could not verify.
- Then stop. Do not commit.
````

---

# Appendix B: Minimal folder layout and adapters

```
your-project/
  AGENTS.md                 ← the only instruction source
  CLAUDE.md                 ← @AGENTS.md
  README.md                 ← for humans
  docs/
    overview.md             ← how it works now
    decisions-not-to-fix.md ← intentional oddities (start with one line)
  decisions/
    README.md               ← index + conventions
    ADR-001-<kebab>.md
  .ai/
    memories/
      overview.md           ← 20 lines, always read
      conventions.md        ← naming/imports/tests
      working/              ← committed in-flight notes
    skills/
      <task>/SKILL.md
    templates/
      spec.md  rfc.md  adr.md
    specs/                  ← gitignored
```

Adapters (run once):

```bash
printf '@AGENTS.md\n' > CLAUDE.md
mkdir -p .claude .opencode .agents
ln -s ../.ai/skills   .claude/skills
ln -s ../.ai/agents   .claude/agents
ln -s ../.ai/skills   .opencode/skills
ln -s ../.ai/agents   .opencode/agents
ln -s ../.ai/skills   .agents/skills
```

`.gitignore` additions:

```gitignore
# Local-only AI task specs (never committed)
.ai/specs/
# Per-machine tool settings (shared adapters stay committed)
.claude/settings.local.json
```

Memory frontmatter:

```yaml
---
description: <what this covers and when an agent should read it>
last-verified: YYYY-MM-DD
---
```

---

# Appendix C: Starter skill and starter ADR

## Skill

````markdown
---
name: add-a-migration
description: Use when the database schema changes and a migration must be
  written, applied locally, and verified before handoff.
---

# add-a-migration

## Prerequisites

- `docs/data-model.md` — current schema and rules.
- `decisions/` — any ADR touching the schema.

## Workflow

1. Read the current schema and confirm the change is additive unless an ADR
   says otherwise.
2. Write the migration in `<path>`, named `<timestamp>-<kebab>.sql`.
3. Write the down-migration.
4. Run it locally: `<command>`.
5. Run the test suite: `<command>`.

## Constraints

- Never edit an already-applied migration; add a new one.
- Never drop a column or table without an ADR.
- No destructive statement without an explicit instruction from me.

## Expected output

The migration + down-migration, a one-paragraph summary of what changed and why,
and the command output proving it applied cleanly.
````

## ADR

````markdown
# ADR-001 — Use SQLite for local development data

- **Status:** Accepted
- **Date:** 2026-10-20
- **Topic:** data / development environment
- **RFC:** —

## Context

<The force that made a decision necessary. Be concrete: what broke, what was
being weighed, what constraints applied.>

## Decision

Local development uses SQLite via <library>. Production continues to use
<Postgres>. The schema is shared through <migrations>.

## Options considered

- **SQLite** — zero setup, file-based, fast tests. Con: dialect differences.
- **Dockerised Postgres** — identical to production. Con: slower start, extra
  moving parts for a solo project.
- **In-memory stub** — fastest. Con: does not exercise real SQL.

## Rationale

Tests need to run on every save; the setup cost of a container outweighs dialect
fidelity at this scale. Dialect differences are contained in `<module>`.

## Consequences

- Tests are fast and need no services.
- Anything using dialect-specific SQL must be isolated and covered by an
  integration test against Postgres.
- Revisit if the schema grows features SQLite cannot model.
````

---

# Appendix D: Prompts to use with your agent

Copy these verbatim when you need them.

**Start of a session**

> Read `AGENTS.md` and the files it links for this task. Then tell me: what the project is, how it is run, and the rules you will follow. Flag anything in the docs that contradicts the code.

**Before an architectural change**

> This is an architectural change. Do not write code yet. Draft a spec from `.ai/templates/spec.md`, list the ambiguities you need me to resolve, and wait for my answer.

**When you want options**

> Give me the options as an RFC: at least two real alternatives with pros and cons, then a recommendation. Ground the context in files and measurements, not assertions.

**When the agent is about to do too much**

> Stop. Show me the diff of what you have so far and what you planned next. Do not continue until I say so.

**When you want a review**

> Review this diff as the reviewer defined in `.ai/agents/reviewer.md`. Judge only what the diff and docs show. Cite `file:line` and the rule, doc section, or ADR each finding breaks. Order by severity.

**Before handoff**

> Run the drift check: list what changed in behavior, find its canonical doc, and update it in the same change. Report what you synchronized or state "no drift" with evidence.

**When something was wrong (the most valuable prompt)**

> You got X wrong. Do not just fix it — tell me what was missing from my instructions or docs that let this happen, and propose the smallest edit that would prevent it next time.

**End of session**

> Summarize: what changed, what was verified, what is still open, and what I need to decide. If anything is unresolved, write it to `.ai/memories/working/<topic>.md` before you stop.

---

# Appendix E: Source index

Everything in this document comes from these files in `mimshins/sudo-overclock`. Read them directly — they are short, and the originals are better than any summary.

**Instructions and map**
- `AGENTS.md` — the single instruction source
- `CLAUDE.md` — the one-line pointer
- `.ai/README.md` — placement rules, precedence, index
- `CONTRIBUTING.md#working-with-ai-agents` — per-tool invocation table

**Decisions about the AI layer itself**
- `decisions/ADR-008-agent-agnostic-knowledge-layer.md` — the layer's design and rationale
- `rfcs/archived/RFC-008-agent-agnostic-knowledge-layer.md` — the options that were weighed
- `decisions/ADR-001-ai-post-authoring-pipeline.md` — why the pipeline is staged and human-led
- `decisions/README.md` — ADR conventions and index

**Roles**
- `.ai/agents/documenter.md`, `designer.md`, `implementer.md`, `reviewer.md`, `editor.md`

**Skills**
- `.ai/skills/post-authoring/SKILL.md` + `stages/{brief,research,outline-critique,editorial,preflight}.md`
- `.ai/skills/design-a-feature/SKILL.md`
- `.ai/skills/write-an-rfc/SKILL.md`
- `.ai/skills/knowledge-drift-sync/SKILL.md`

**Memories**
- `.ai/memories/{overview,architecture,conventions,content-pipeline,development-workflow}.md`
- `.ai/memories/working/README.md` — the lifecycle

**Templates**
- `.ai/templates/{spec,rfc,adr,memory,working-memory,skill,agent}.md`

**Process and gates**
- `docs/authoring.md` — the pipeline, gates, editorial standard, preflight checklist
- `.githooks/pre-push` — the versioned quality gate
- `.github/PULL_REQUEST_TEMPLATE.md` — definition of done
- `.ai/skills/knowledge-drift-sync/SKILL.md` — the drift procedure
- `.gitignore` — `.ai/specs/` and `.claude/settings.local.json`

---

*Reference document generated from a read of `mimshins/sudo-overclock` at commit state of 2026-10-09. Quotations are verbatim from the repository; everything else is explanation.*
