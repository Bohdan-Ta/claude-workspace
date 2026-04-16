# `.claude/` — Claude Code Workspace (not yet initialized)

**Run `/init` to configure this workspace for your project.**

This is a template workspace. After initialization, this file will be updated with your project's specific structure.

---

## How Claude Code loads this

1. **`CLAUDE.md` in project root** — read at session start. Contains `@`-imports for rules and lessons — always in context.
2. **`rules/*.md`** — without `paths:` always loaded; with `paths:` only when matching files are edited.
3. **`agents/*.md`** — registered as `subagent_type`, lazy-loaded when dispatched.
4. **`skills/*/SKILL.md`** — registered as `/<name>` slash commands, lazy-loaded.
5. **`settings.json`** — team-wide permissions and configuration.

## Available Skills

- `/init` — Initialize this workspace for your project (run this first!)
- `/review-pr` — Orchestrate reviewer agents for PR review
- `/release-tag` — Create and push release tags
