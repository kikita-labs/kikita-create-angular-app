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

**Project-scoped (this project only):**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git \
  .claude/skills/kikita-create-angular-app
```

Restart Claude Code (or start a new session) so it picks up the new skill.

### Codex

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git \
  "$CODEX_HOME/skills/kikita-create-angular-app"
```

(`$CODEX_HOME` defaults to `~/.codex` if unset.)

## Use

From an empty (or near-empty) directory where you want a new Angular app:

```
/kikita-create-angular-app
```

The agent will ask the pre-init questionnaire, then follow `plan.md` end to end: install
`angular-mcp`, scaffold the app, wire tooling, generate the `.agents/` doc tree, set up
git, and run `checklist.md` before telling you it's done.

## Repo structure

```
SKILL.md          # skill entry point: questionnaire + generation rules
plan.md           # step-by-step init sequence the skill follows
checklist.md       # post-init verification
templates/         # everything copied into the generated project
  AGENTS.md, CLAUDE.md, .gitignore, .editorconfig, .prettierrc, .prettierignore,
  .nvmrc, .vscode/extensions.json
  .agents/          # the documentation tree template, mirrors what gets generated
```

## License

No license file yet — all rights reserved by default until one is added. Open an issue if
you need a specific license for your use case.
