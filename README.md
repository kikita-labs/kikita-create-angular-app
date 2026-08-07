# kikita-create-angular-app

An agent skill that scaffolds a brand-new Angular project (latest stable) and generates a
full `.agents/` documentation tree alongside it — code style, architecture, git policy,
MCP setup, testing/quality gate, and reuse registries — so any AI agent working in the
project afterwards has a complete, self-maintaining source of truth from commit one.

See [`SKILL.md`](./SKILL.md) for what it does, [`plan.md`](./plan.md) for the exact
step-by-step scaffolding sequence, and [`checklist.md`](./checklist.md) for the
post-init verification it runs before handing the project back to you.

## What it generates

- `CLAUDE.md` → `AGENTS.md` → `.agents/*.md` — the full documentation tree, described in
  [`templates/.agents/README.md`](./templates/.agents/README.md).
- A working Angular app: latest stable Angular CLI, signals/Signal Forms/`@Service`
  throughout, ESLint + Prettier + Husky pre-wired, `angular-mcp` installed first.
- A short pre-init questionnaire (CSS engine, UI library, tests, SSR, i18n, JSDoc policy,
  project prefix, git policy, package manager, git remote) drives which docs and config
  get generated.

## Install

This is a skill for AI coding agents (Claude Code, Codex). Installing it means placing
this repo's contents under a `<skill-name>/` folder inside the agent's skills directory,
so the folder name matches this repo's name.

### Claude Code

**Personal (all your projects):**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git \
  ~/.claude/skills/kikita-create-angular-app
```

**Project-scoped (this project only, committed to the repo):**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git \
  .claude/skills/kikita-create-angular-app
```

Claude Code picks up new/changed skills under `~/.claude/skills/` and `.claude/skills/`
live, within the current session — no restart needed, unless the top-level `.claude/skills/`
directory didn't exist yet when the session started (in that case restart once so Claude
Code starts watching the new directory).

### Codex

Codex does **not** use `$CODEX_HOME` or `.codex/skills` for skills — that's a common but
incorrect claim floating around. The real locations, per Codex's own docs:

**User scope (all your projects):**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git \
  "$HOME/.agents/skills/kikita-create-angular-app"
```

**Repo scope (this project, and any subdirectory under it):**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git \
  .agents/skills/kikita-create-angular-app
```

Codex scans `.agents/skills` in the current directory and every parent up to the repo
root, so this also works from a subfolder of a larger repo. If a newly installed or
updated skill doesn't show up, restart Codex.

## Use

From an empty (or near-empty) directory where you want a new Angular app:

```
/kikita-create-angular-app
```

The agent will ask the pre-init questionnaire, then follow `plan.md` end to end: install
`angular-mcp`, scaffold the app, wire tooling, generate the `.agents/` doc tree, set up
git, and run `checklist.md` before telling you it's done.

## Update

Already scaffolded a project with this skill and the templates have moved on since? Run the
exact same command inside that project:

```
/kikita-create-angular-app
```

The skill detects `.agents/.kikita-scaffold.json` (written at scaffold time) and switches to
update mode instead of re-running the questionnaire: it `git pull`s its own install directory,
diffs `templates/.agents/` between the commit the project was scaffolded/last-updated from and
the current `HEAD`, and merges what changed into the project's `.agents/` files — never a
blind overwrite, since those files usually pick up project-specific edits after scaffolding.
See [`update.md`](./update.md) for the exact algorithm.

This works the same way whether you're driving Claude Code by hand or a fully agent-driven
("vibecoding") workflow never opens the project directly — it's the same slash command either
way, no separate `-update` skill to install or remember.

## Repo structure

```
SKILL.md          # skill entry point: mode detection, questionnaire + generation rules
plan.md           # step-by-step init sequence the skill follows
update.md         # step-by-step sequence for updating an already-scaffolded project
checklist.md       # post-init verification
templates/         # everything copied into the generated project
  AGENTS.md, CLAUDE.md, .gitignore, .editorconfig, .prettierrc, .prettierignore,
  .nvmrc, .vscode/extensions.json
  .agents/          # the documentation tree template, mirrors what gets generated
```

## License

[MIT](./LICENSE)
