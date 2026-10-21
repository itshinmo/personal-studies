# AI project starter

A minimal, copy-paste starting point distilled from
`mimshins/sudo-overclock`. Read `../ai-development-reference.md` for the full
explanation of every idea here.

## Files

| File        | Purpose                                                        |
| ----------- | -------------------------------------------------------------- |
| `AGENTS.md` | The single instruction source. **Edit this one.**              |
| `CLAUDE.md` | One line (`@AGENTS.md`) so Claude Code reads the same file.    |

## How to use it

1. Copy `AGENTS.md` and `CLAUDE.md` into the root of your project.
2. Fill in the placeholders in `AGENTS.md`: project description, the five
   commands, and your 3–5 non-negotiables. Delete what doesn't apply.
3. Create `docs/overview.md`: ten lines on what the project is, how it runs, and
   where things live.
4. Create `decisions/` with a `README.md` index and one real `ADR-001` for a
   choice you have already made.
5. Add a **"Do not fix"** section to your docs listing the things about your
   project that look wrong but are deliberate.
6. Start a session with: *"Read AGENTS.md and summarize the project and its
   rules back to me."* Whatever it gets wrong is a documentation gap — fix it.

## Optional next steps (add only when you feel the pain)

```bash
# Point other tools at the same skills/agents folder
mkdir -p .ai/skills .ai/agents .claude .opencode .agents
ln -s ../.ai/skills .claude/skills
ln -s ../.ai/agents .claude/agents
ln -s ../.ai/skills .opencode/skills
ln -s ../.ai/agents .opencode/agents
ln -s ../.ai/skills .agents/skills
```

Add to `.gitignore`:

```gitignore
.ai/specs/
.claude/settings.local.json
```

Then, in order: `.ai/memories/overview.md`, `.ai/templates/spec.md`, a
`.github/PULL_REQUEST_TEMPLATE.md` definition of done, and a pre-push hook that
runs your lint and tests.

## The two habits that matter most

- **Never let the agent commit without showing you the diff.** Read the diff.
- **When it gets something wrong, fix the instruction, not just the code.**
