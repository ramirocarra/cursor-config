# Global Cursor configuration (GitHub)

This repository holds **user-level Cursor rules** (`.cursor/rules/`) and **two custom Agent skills** (`skills/`) so you can version and sync global agent config.

Copy or symlink the `skills/` folders into `~/.cursor/skills/` so Cursor loads them globally (see **Install**).

## Custom skills

| Skill | In-repo path | Summary |
|-------|----------------|---------|
| `session-change-review` | `skills/session-change-review/SKILL.md` | End-of-session / pre-commit impact, bugs, DB index hints, quality review from git diff |
| `database-index` | `skills/database-index/SKILL.md` | Evidence-backed index suggestions from SQL/ORM usage |

## Install on a machine

1. Clone (or fork) this repo where you like, for example `~/Documents/cursor-settings`.
2. **Rules** — copy the entire rules tree into Cursor’s user rules directory (nested folders must be included):

   ```bash
   cd /path/to/cursor-settings
   mkdir -p ~/.cursor/rules
   cp -R .cursor/rules/* ~/.cursor/rules/
   ```

   To symlink only grouped folders (stays updated on `git pull`):

   ```bash
   mkdir -p ~/.cursor/rules
   ln -sf "$(pwd)/.cursor/rules/general-coding-guidelines" ~/.cursor/rules/general-coding-guidelines
   ln -sf "$(pwd)/.cursor/rules/rust" ~/.cursor/rules/rust
   ```

   Symlink any other top-level `.mdc` files the same way if you use symlinks for the rest.

3. **Custom skills** — install into `~/.cursor/skills/` so Cursor discovers them:

   ```bash
   cd /path/to/cursor-settings
   mkdir -p ~/.cursor/skills
   cp -R skills/session-change-review skills/database-index ~/.cursor/skills/
   ```

   Or symlink so pulls update skills automatically:

   ```bash
   mkdir -p ~/.cursor/skills
   ln -sf "$(pwd)/skills/session-change-review" ~/.cursor/skills/session-change-review
   ln -sf "$(pwd)/skills/database-index" ~/.cursor/skills/database-index
   ```

4. Restart Cursor or open a new chat so rules and skills are picked up.

**Note:** Project `.cursor/rules/` and `.cursor/skills/` in a workspace still apply alongside global ones. Cursor scans **nested** `.mdc` files under `.cursor/rules/` (v0.47+). If you use a **multi-root** workspace, avoid mixing only some rules in subfolders and some at the root of `rules/`—use one consistent layout to avoid known loading quirks.

## Push to GitHub

```bash
git remote add origin git@github.com:<you>/<repo>.git
git branch -M main
git push -u origin main
```

Replace `<you>/<repo>` with your GitHub user and repository name.

## What’s in here

| Area | Location |
|------|----------|
| Custom skills | `skills/session-change-review/`, `skills/database-index/` |
| Compliance, execution, communication | `.cursor/rules/` — `compliance-and-skills`, `execution-environment`, `markdown-communication`, `conversation-context` |
| General coding | `.cursor/rules/general-coding-guidelines/` — `scoped-code-changes`, `documentation-policy`, `dependencies-extensibility`, `performance`, `error-context`, `logging-diagnostics` |
| Git / workflow | `.cursor/rules/` — `git-commands-restriction`, `git-commit-format`, `new-project-agents` |
| Rust only | `.cursor/rules/rust/rust-conventions.mdc` (`globs: **/*.rs`) |
| UI / styling | `.cursor/rules/ui-web-polish.mdc` (`globs: **/*.{tsx,jsx,vue,svelte,css,scss}`) |

Edit `.mdc` files under `.cursor/rules/` (and subfolders) with `alwaysApply` / `globs` in frontmatter; edit custom skills under `skills/*/SKILL.md`.
