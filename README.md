# kikita-create-angular-app

An agent skill that scaffolds a brand-new Angular project (latest stable) and generates a
full `.agents/` documentation tree alongside it — code style, architecture, git policy,
MCP setup, testing/quality gate, and reuse registries — so any AI agent working in the
project afterwards has a complete, self-maintaining source of truth from commit one.

Packaged as an [Agent Plugin](https://agent-plugins.org) (spec v1.0.0) — a portable format
usable by any compatible client (Cursor, GitHub Copilot, ChatGPT/Codex, VS Code, Kiro, …),
not just Claude Code.

See [`skills/kikita-create-angular-app/SKILL.md`](./skills/kikita-create-angular-app/SKILL.md)
for what it does, [`plan.md`](./skills/kikita-create-angular-app/plan.md) for the exact
step-by-step scaffolding sequence, and [`checklist.md`](./skills/kikita-create-angular-app/checklist.md)
for the post-init verification it runs before handing the project back to you.

## What it generates

- `CLAUDE.md` → `AGENTS.md` → `.agents/*.md` — the full documentation tree, described in
  [`templates/.agents/README.md`](./skills/kikita-create-angular-app/templates/.agents/README.md).
- A working Angular app: latest stable Angular CLI, signals/Signal Forms/`@Service`
  throughout, ESLint + Prettier + Husky pre-wired, `angular-mcp` installed first.
- A short pre-init questionnaire (CSS engine, UI library, tests, SSR, i18n, JSDoc policy,
  project prefix, git policy, package manager, git remote) drives which docs and config
  get generated.

## Install

This repo is an [Agent Plugin](https://agent-plugins.org): a `plugin.json` manifest at the
root plus a `skills/kikita-create-angular-app/` directory holding the actual
[Agent Skill](https://agent-plugins.org/specification). Any Agent-Plugins-compatible client
can load it straight from a clone of this repo.

### Agent-Plugins-compatible clients (Cursor, GitHub Copilot, ChatGPT/Codex, VS Code, Kiro, …)

Point the client's plugin install flow at this repo (clone URL or local path). The client
discovers `plugin.json`, then the skill under `skills/kikita-create-angular-app/`. Refer to
your client's own docs for the exact install command — the Agent Plugins spec defines the
package format, not a universal installer.

### Claude Code

Claude Code doesn't read the Agent Plugins format natively yet, so install the skill
subdirectory directly:

**Personal (all your projects):**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git /tmp/kikita-cli-app && \
  cp -r /tmp/kikita-cli-app/skills/kikita-create-angular-app ~/.claude/skills/kikita-create-angular-app
```

**Project-scoped (this project only, committed to the repo):**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git /tmp/kikita-cli-app && \
  cp -r /tmp/kikita-cli-app/skills/kikita-create-angular-app .claude/skills/kikita-create-angular-app
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
git clone https://github.com/kikita-labs/kikita-create-angular-app.git /tmp/kikita-cli-app && \
  cp -r /tmp/kikita-cli-app/skills/kikita-create-angular-app "$HOME/.agents/skills/kikita-create-angular-app"
```

**Repo scope (this project, and any subdirectory under it):**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git /tmp/kikita-cli-app && \
  cp -r /tmp/kikita-cli-app/skills/kikita-create-angular-app .agents/skills/kikita-create-angular-app
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
diffs `skills/kikita-create-angular-app/templates/.agents/` between the commit the project was
scaffolded/last-updated from and the current `HEAD`, and merges what changed into the
project's `.agents/` files — never a blind overwrite, since those files usually pick up
project-specific edits after scaffolding. See [`update.md`](./skills/kikita-create-angular-app/update.md)
for the exact algorithm. Note this requires a git-clone install (not a copy) so `<plugin-root>`
has history to diff against — see `update.md` section 1.

This works the same way whether you're driving the agent by hand or a fully agent-driven
("vibecoding") workflow never opens the project directly — it's the same slash command either
way, no separate `-update` skill to install or remember.

## Repo structure

```
plugin.json         # Agent Plugins manifest (name, version, metadata) — see agent-plugins.org
skills/
  kikita-create-angular-app/
    SKILL.md          # skill entry point: mode detection, questionnaire + generation rules
    plan.md           # step-by-step init sequence the skill follows
    update.md         # step-by-step sequence for updating an already-scaffolded project
    adopt.md          # step-by-step sequence for retrofitting docs onto an existing project
    checklist.md      # post-init verification
    templates/        # everything copied into the generated project
      AGENTS.md, CLAUDE.md, .gitignore, .editorconfig, .prettierrc, .prettierignore,
      .nvmrc, .vscode/extensions.json
      .agents/          # the documentation tree template, mirrors what gets generated
```

No `mcp.json` at the plugin root: `angular-mcp` is installed *into the generated project*
by `plan.md`, not run as an MCP server for this skill itself.

## License

[MIT](./LICENSE)
