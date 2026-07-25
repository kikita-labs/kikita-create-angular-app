# Post-Init Checklist

Run through every item. Do not report the project as ready until every box that applies
is genuinely true — do not check a box you didn't verify.

## Setup

- [ ] Latest stable Angular CLI used (`ng version` matches the current stable release, not
      a pinned old one).
- [ ] `angular-mcp` installed and responding before any other install happened.
- [ ] Project builds (`ng build`) with zero errors.
- [ ] Project prefix from the questionnaire is applied in `angular.json` / component
      selectors, not left as the CLI default.
- [ ] Project scaffolded with the chosen package manager (lockfile matches: `pnpm-lock.yaml`
      / `package-lock.json` / `yarn.lock` — exactly one, not several).

## Tooling

- [ ] `.gitignore` present and covers `node_modules`, `dist`, `.angular/cache`, env files,
      lockfiles of the *other* package managers.
- [ ] `.editorconfig`, `.prettierrc`, `.prettierignore`, `.vscode/extensions.json` present.
- [ ] `.nvmrc` present with the real Node version, not the `{{NODE_VERSION}}` placeholder.
- [ ] ESLint flat config present (`@angular-eslint` recommended + `typescript-eslint`
      `strict-type-checked` + `simple-import-sort` + `consistent-type-imports` + a
      restricted-import boundary rule, `eslint-config-prettier` last), lints with zero
      errors on the generated skeleton.
- [ ] Prettier configured, including Angular HTML templates,
      `"htmlWhitespaceSensitivity": "ignore"` set.
- [ ] `lint`, `format`, `format:check` scripts exist (run via the chosen package manager)
      and run clean.
- [ ] `package.json` has `"prepare": "husky"`. Husky installed: `pre-commit` runs
      `lint-staged` + the non-English content check, `pre-push` runs the full lint +
      format + test gate. Verify both hooks actually fire (e.g. dry-run a commit).
- [ ] `angular.json` build budgets tightened from CLI defaults, not left unset.
- [ ] If tests were chosen: test runner(s) installed, a sample test passes, `test` script
      exists.

## Documentation

- [ ] `CLAUDE.md` exists and only points to `AGENTS.md`.
- [ ] `AGENTS.md` "Must Read" list contains only files that actually exist — no dead links.
- [ ] `.agents/README.md` exists and links only to files that actually exist — no dead
      links, no missing conditional files (i18n, agent-surface, platform-adapter).
- [ ] Every file under `.agents/` has all `{{PLACEHOLDER}}` tokens replaced with real values.
- [ ] `.agents/code-style/README.md` links to every file in `.agents/code-style/`.
- [ ] `.agents/architecture/README.md` links to every file in `.agents/architecture/`.
- [ ] `.agents/shared/README.md` and `.agents/core/README.md` exist even if empty, each
      with instructions for how to register a new entry.
- [ ] `.agents/decisions/README.md` exists with the ADR trigger list and format.
- [ ] `.agents/code-style/css-architecture.md` present and linked from
      `.agents/code-style/README.md`.
- [ ] `.agents/i18n.md` present only if i18n was requested; absent otherwise.
- [ ] `.agents/architecture/platform-adapter.md` present only if SSR was chosen; absent
      otherwise.
- [ ] `.agents/agent-surface.md` present only if mandatory JSDoc was chosen.
- [ ] `.agents/mcp.md` lists every MCP server actually installed (angular-mcp + any UI-lib
      MCP), with the Claude (`.mcp.json`) vs Codex (`.codex/config`) split explained.
- [ ] `.agents/documentation.md` (the "how to write/maintain docs" master file) exists and
      is linked from `AGENTS.md`.
- [ ] No Cyrillic or mojibake in any tracked file, including JSDoc.

## Git

- [ ] `.agents/git-policy.md` reflects the questionnaire's ask-before-push answer correctly.
- [ ] `git remote` set to the URL the user gave, if one was given; not invented, not left
      unset if one was provided.
- [ ] First commit made, message has no AI attribution / co-authorship lines.
- [ ] `git status` clean after the commit (nothing left stray/untracked that should be
      tracked or ignored).

## Final

- [ ] Re-read `AGENTS.md` top to bottom as if you were a fresh agent — does it actually
      orient you correctly with no missing context?
