# AGENTS.md

The only instruction source for AI agents working on **<PROJECT NAME>**. Every
tool reads this file (Claude Code through `CLAUDE.md` → `@AGENTS.md`); tool
folders only point here. Read this first, then follow the links.

> **How to use this file:** replace every `<...>` placeholder, delete anything
> that does not apply, and keep it under ~100 lines. A short file that is true
> beats a long file that is aspirational. Re-read it when the agent gets
> something wrong — the missing instruction is usually the bug.

## Approval gate — never stage or commit on your own

**Do not run `git add`, `git commit`, or `git push` (or anything that stages,
such as `git mv` or `git rm`) unless I explicitly ask.** After verifying, stop
and hand me the diff. An approval covers the reviewed diff only.

Do not install or upgrade dependencies, change CI, or delete files without
asking first.

## What this project is

<One or two sentences: what it does, for whom, and how it is built. Example:
"A statically generated Next.js site, deployed to <host>, with content in
Markdown.">

## Commands

```sh
<install>   # e.g. pnpm install
<dev>       # e.g. pnpm dev
<test>      # e.g. pnpm test
<lint>      # e.g. pnpm check:lint
<build>     # e.g. pnpm build
```

Always use these commands. Do not invent flags or substitute a different package
manager.

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

| Topic                            | Location         |
| -------------------------------- | ---------------- |
| How it works now, and the rules  | `docs/`          |
| Why a decision was made (ADRs)   | `decisions/`     |
| Proposals not yet decided        | `rfcs/`          |
| Agent orientation, skills, roles | `.ai/`           |
| Local task specs (gitignored)    | `.ai/specs/`     |

Project knowledge never lives only in `.ai/`. **Runtime code never reads
`.ai/`** — removing every AI tool must leave the project buildable and every
decision readable.

## Spec-driven development

For an **architectural or design change** — module boundaries, the data model,
cross-cutting patterns, major dependencies, or a decision with real trade-offs:

1. write a spec from `.ai/templates/spec.md` into `.ai/specs/` (local);
2. ask me about anything ambiguous;
3. write an RFC from `.ai/templates/rfc.md` if there are real alternatives;
4. break it into ordered tasks;
5. **confirm with me before writing code.**

Routine changes, small fixes, and bug fixes need no spec. When in doubt, ask.

## Non-negotiables

**Code**

- <e.g. keep business logic out of route handlers / UI components>
- <e.g. one module talks to another only through its public interface>
- <e.g. no new runtime dependency without an ADR>
- <e.g. no comments unless asked; document the code by naming things well>
- Files are kebab-case; component identifiers are PascalCase.

**Safety and truth**

- Never fabricate commands, numbers, sources, decisions, or file paths.
  Unknowns become `<!-- TODO -->`, `[unverified]`, or a question to me.
- No secrets, tokens, credentials, or private hostnames in any file.
- Never hand-edit generated output: `<e.g. dist/, build/, out/>`.
- Never weaken a test or lint rule to make something pass. A suppression needs
  the specific rule and a reason.
- One logical change at a time.
- If a spec is missing or ambiguous, stop and ask. Do not invent behavior.

## Before you finish

- Run `<lint>` and `<test>`; `<build>` when the app or build config changed.
- Update the doc that the change contradicts, **in the same change**.
- If a decision was made, record an ADR in `decisions/`.
- Report: files changed, what was verified, anything you could not verify.
- **Then stop. Do not commit.**
