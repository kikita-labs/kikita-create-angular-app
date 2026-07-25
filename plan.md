# Init Plan

Execute in order. Do not reorder. Do not skip a step because it "seems unnecessary" —
if a step doesn't apply (e.g. no UI library chosen), mark it skipped explicitly and move on.

1. **Ask the questionnaire** (`SKILL.md` section 1). Do not proceed until every answer is
   recorded.

2. **Install `angular-mcp` first.** Nothing else happens before this. If it fails or is
   unavailable, stop and tell the user — do not continue scaffolding blind.

3. **Scaffold the Angular app** with the latest stable Angular CLI (`ng new`), using the
   questionnaire answers: CSS engine, project prefix, SSR on/off, routing enabled,
   standalone (default in current Angular — do not pass a deprecated NgModule flag), and
   the chosen package manager (`--package-manager={{PACKAGE_MANAGER}}`; default `pnpm`
   unless the user asked for npm/yarn).

4. **If a UI library was chosen**:
   - Check for an `ng add` schematic first; use it if present.
   - Otherwise follow the library's install docs manually.
   - Search for an MCP server for that library. If one exists, install it the same way as
     `angular-mcp` (see `templates/.agents/mcp.md` for the Claude vs Codex config split).
   - Record the library and its MCP (or lack thereof) in the generated `.agents/mcp.md`.

5. **If tests were chosen**, install and wire the chosen runner(s) (Vitest for unit,
   Playwright for e2e). Add the corresponding npm scripts. Document them in the generated
   `.agents/testing-and-quality.md`.

6. **Set up tooling**:
   - Copy `templates/.gitignore`, `templates/.editorconfig`, `templates/.prettierrc`,
     `templates/.prettierignore`, `templates/.vscode/extensions.json` into the project as-is.
   - Copy `templates/.nvmrc`, replacing `{{NODE_VERSION}}` with the Node version the chosen
     Angular release actually requires (check the Angular CLI's own `engines` requirement,
     don't guess).
   - Latest ESLint flat config (`eslint.config.js`): `@angular-eslint` recommended +
     `typescript-eslint` `strict-type-checked` + `eslint-plugin-simple-import-sort` +
     `@typescript-eslint/consistent-type-imports` + a restricted-import boundary rule
     (`no-restricted-imports` or `eslint-plugin-boundaries`), with `eslint-config-prettier`
     last in the config array. `ignores` covers `dist/`, `node_modules/`, `.angular/`,
     `coverage/`, lockfiles. See `templates/.agents/testing-and-quality.md` for the full
     rationale.
   - Wire `lint` / `format` / `format:check` scripts in `package.json` using
     `{{PACKAGE_MANAGER}}`.
   - Tighten `angular.json` `architect.build.options.budgets` from the CLI defaults — don't
     leave them unset.
   - Install and configure Husky + `lint-staged`, and add `"prepare": "husky"` to
     `package.json` (this is what actually installs the hooks after a fresh clone):
     `pre-commit` runs `lint-staged` (ESLint `--fix` + Prettier `--write` on staged files
     only) plus a non-English content check (grep staged files for non-ASCII letters);
     `pre-push` runs the full `lint` + `format:check` + test gate. See
     `templates/.agents/git-policy.md` and `templates/.agents/testing-and-quality.md` for
     the exact hook responsibilities.

7. **Generate the documentation tree** from `templates/`:
   - `CLAUDE.md`, `AGENTS.md` at project root.
   - `.agents/README.md` — flat index of everything under `.agents/`, kept in sync with
     whichever conditional files actually got generated.
   - `.agents/*.md` flat topic docs (workflow, git-policy, documentation, mcp,
     testing-and-quality, refactoring, progress, accessibility — always include all of
     these) plus `agent-surface.md` (only if mandatory JSDoc was chosen) and `i18n.md`
     (only if i18n was requested).
   - `.agents/code-style/` with its `README.md` hub + `imports.md`, `html-markup.md`,
     `component-structure.md`, `rxjs-and-signals.md`, `css-architecture.md`.
   - `.agents/architecture/` with its `README.md` hub + `aliases-and-barrels.md`,
     `folder-structure.md`, `routing.md`, and `platform-adapter.md` only if SSR was chosen.
   - `.agents/shared/README.md` and `.agents/core/README.md` — both start empty (a
     registry table with no rows yet); explain in each how future entries get registered.
   - `.agents/decisions/README.md` — always generated; explains when an ADR is required
     and the file format. Starts with no ADR files, just the README.
   - Fill every `{{PLACEHOLDER}}` with the real questionnaire answer. Leave no placeholder
     text in the generated project.
   - Update `AGENTS.md`'s "Must Read" list to only reference files that were actually
     generated (no dead links).

8. **Wire `mcp` config** in the new project itself, not just install:
   - Claude Code: `.mcp.json` at project root.
   - Codex: `.codex/config`.
   Both should end up with the same server entries; generate whichever the current agent
   runtime uses, and mention the other in `.agents/mcp.md` so a future agent on the other
   runtime knows to add it.

9. **`git init`, wire remote if given one, first commit.**
   - `git init` if not already a repo.
   - If the questionnaire gave a remote URL: `git remote add origin <url>`. If no URL was
     given, skip — do not invent or guess a remote.
   - Commit following the generated `.agents/git-policy.md` (commit message rules, no AI
     attribution).
   - Only push if the git-policy answer authorizes it without asking; otherwise stop after
     the commit and ask before pushing.

10. **Run `checklist.md`.** Fix anything that fails before reporting completion to the user.
