# kikita-create-angular-app

An agent skill that scaffolds a brand-new Angular project (latest stable) and generates a
full `.agents/` documentation tree alongside it — code style, architecture, git policy,
MCP setup, testing/quality gate, and reuse registries — so any AI agent working in the
project afterwards has a complete, self-maintaining source of truth from commit one.

Packaged as an [Agent Skill](https://agentskills.io) — an open, portable format usable by
any compatible client (Claude Code, Codex, Cursor, GitHub Copilot, VS Code, Kiro, …), not
just one product.

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

This repo follows the [Agent Skills spec](https://agentskills.io/specification): the actual
skill is the `skills/kikita-create-angular-app/` directory, with `SKILL.md` at its root. The
spec doesn't define an update mechanism, so this skill's own `update.md` falls back to git:
it walks up from wherever `SKILL.md` is running to find a `.git`, then diffs against
upstream. That means the installed skill folder must still be inside a real git clone — a
bare `cp` that drops `.git` breaks updates silently (see the Update section below).

Since clients expect `SKILL.md` directly at the top of the installed skill folder, but the
clone's `SKILL.md` sits one level down (`skills/kikita-create-angular-app/`), install by
cloning the repo to a fixed source location once, then linking the client's skills folder to
the subdirectory inside it — a symlink (or, on Windows, a directory junction, which unlike a
symlink needs no admin rights) keeps `.git` reachable through the link.

### Claude Code

**Personal (all your projects), macOS/Linux:**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git ~/.kikita-create-angular-app-src
ln -s ~/.kikita-create-angular-app-src/skills/kikita-create-angular-app ~/.claude/skills/kikita-create-angular-app
```

**Personal, Windows (PowerShell):**

```powershell
git clone https://github.com/kikita-labs/kikita-create-angular-app.git "$HOME\.kikita-create-angular-app-src"
New-Item -ItemType Junction -Path "$HOME\.claude\skills\kikita-create-angular-app" -Target "$HOME\.kikita-create-angular-app-src\skills\kikita-create-angular-app"
```

**Project-scoped (this project only, committed to the repo), macOS/Linux:**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git .kikita-create-angular-app-src
ln -s ../../.kikita-create-angular-app-src/skills/kikita-create-angular-app .claude/skills/kikita-create-angular-app
```

Add `.kikita-create-angular-app-src/` to the project's `.gitignore` (it's a vendored clone,
not this project's own source) — commit only the symlink/junction under `.claude/skills/`.

Claude Code picks up new/changed skills under `~/.claude/skills/` and `.claude/skills/` live,
within the current session — no restart needed, unless the top-level `.claude/skills/`
directory didn't exist yet when the session started (in that case restart once so Claude
Code starts watching the new directory).

### Codex

Codex does **not** use `$CODEX_HOME` or `.codex/skills` for skills — that's a common but
incorrect claim floating around. The real locations, per Codex's own docs, same
clone-then-link pattern as above:

**User scope (all your projects), macOS/Linux:**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git ~/.kikita-create-angular-app-src
ln -s ~/.kikita-create-angular-app-src/skills/kikita-create-angular-app "$HOME/.agents/skills/kikita-create-angular-app"
```

**Repo scope (this project, and any subdirectory under it), macOS/Linux:**

```sh
git clone https://github.com/kikita-labs/kikita-create-angular-app.git .kikita-create-angular-app-src
ln -s ../.kikita-create-angular-app-src/skills/kikita-create-angular-app .agents/skills/kikita-create-angular-app
```

On Windows use `New-Item -ItemType Junction` as shown for Claude Code above, pointed at
`.agents/skills/kikita-create-angular-app` instead.

Codex scans `.agents/skills` in the current directory and every parent up to the repo
root, so this also works from a subfolder of a larger repo. If a newly installed or
updated skill doesn't show up, restart Codex.

### Other Agent-Skills-compatible clients

Any client that implements the [Agent Skills spec](https://agentskills.io/specification) can
load the skill straight from `skills/kikita-create-angular-app/` in a clone of this repo —
refer to that client's own docs for its install command, since the spec defines the skill
folder format, not a universal installer.

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
update mode instead of re-running the questionnaire: it `git pull`s its own source clone,
diffs `skills/kikita-create-angular-app/templates/.agents/` between the commit the project was
scaffolded/last-updated from and the current `HEAD`, and merges what changed into the
project's `.agents/` files — never a blind overwrite, since those files usually pick up
project-specific edits after scaffolding. See [`update.md`](./skills/kikita-create-angular-app/update.md)
for the exact algorithm. This is why the Install section above always clones (never `cp`s) —
without a `.git` reachable from the installed skill folder, `update.md` has nothing to diff
against and update mode can't run.

This works the same way whether you're driving the agent by hand or a fully agent-driven
("vibecoding") workflow never opens the project directly — it's the same slash command either
way, no separate `-update` skill to install or remember.

No MCP server for this skill itself: `angular-mcp` is installed *into the generated project*
by `plan.md`, not run against this repo.

## License

[MIT](./LICENSE)
