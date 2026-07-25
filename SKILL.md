---
name: kikita-create-angular-app
description: Scaffold a new Angular project (latest stable) with a full .agents/ documentation tree, code style, MCP, and git policy pre-wired. Use when the user asks to init/bootstrap/create a new Angular app, or invokes /kikita-create-angular-app in an empty or near-empty directory.
---

# kikita-create-angular-app

Bootstraps a new Angular project and generates its `AGENTS.md` / `.agents/` documentation
tree so any AI agent (Claude Code, Codex, etc.) working in the project afterwards has a
complete, self-maintaining source of truth.

Follow `plan.md` step by step. Do not skip the questionnaire. Do not skip MCP install.
When done, run through `checklist.md` before telling the user it's finished.

## 0. Preconditions

- Confirm the working directory is empty or the user explicitly wants to init here.
- Never run this against a directory that already has an unrelated project without asking first.

## 1. Questionnaire (always ask before touching the filesystem)

Ask the user, one round, all at once:

1. **CSS**: native CSS (custom properties) or SCSS?
2. **UI library**: `@kikita-labs/ui`, Taiga UI, PrimeNG, or none (build a local token-based UI set)?
   - If a library is chosen: look up whether it ships an `ng add` schematic and/or an MCP
     server. Prefer the schematic. If it has an MCP, install it (see `plan.md` step 4).
3. **Tests**: none, unit only, e2e only, or both? Which runner (Vitest/Jasmine+Karma is
   deprecated on new Angular — prefer Vitest; Playwright for e2e)?
4. **SSR**: is this app server-rendered (Angular SSR / hydration) or client-only?
   - Drives whether `.agents/architecture/platform-adapter.md` is generated.
5. **i18n**: does this project need multiple locales (`@angular/localize` or a runtime i18n
   lib)? If yes, which approach?
   - Drives whether `.agents/i18n.md` is generated.
6. **JSDoc on public API**: enforce mandatory JSDoc on every exported symbol, or skip it?
   - Drives whether `.agents/agent-surface.md` is generated (recommended default: yes).
7. **Project prefix**: short kebab-case selector prefix for components/directives.
8. **Git policy**: may the agent commit and push without asking each time, or must every
   commit/push be confirmed?
9. **Package manager**: npm, pnpm, or yarn? Default recommendation: pnpm (faster, disk-
   efficient via a shared content-addressable store, stricter dependency resolution). Only
   override the default if the user asks for something else.
10. **Git remote**: does the user already have a repo URL to push to? If yes, record it —
    `git remote add origin <url>` runs right after `git init` (see `plan.md` step 9). If no
    URL is given, skip this; the user wires the remote later themselves.

Record every answer — they drive both scaffolding and which doc files get generated.
Never silently assume a default; if the user skips a question, ask again for that one.
This skill targets a single Angular project, not a monorepo — if the user wants a
monorepo/Nx workspace, say this skill doesn't cover that and stop rather than improvising.

## 2. Generate

Follow `plan.md`. Copy files from `templates/` into the target project, including the
dotfiles (`.gitignore`, `.editorconfig`, `.prettierrc`, `.prettierignore`, `.nvmrc`,
`.vscode/extensions.json`).

Two different things happen with questionnaire answers, don't conflate them:

- **Text placeholders** — find-and-replace every `{{TOKEN}}` with the real value, leave none
  behind: `{{PREFIX}}`, `{{PROJECT_NAME}}`, `{{CSS}}`, `{{CSS_EXT}}`, `{{UI_LIB}}`,
  `{{UI_LIB_MCP_NAME}}`, `{{UI_LIB_MCP_PURPOSE}}`, `{{TESTS}}`, `{{GIT_POLICY}}`,
  `{{PACKAGE_MANAGER}}`, `{{NODE_VERSION}}`, `{{I18N_APPROACH}}`, `{{DATE}}`.
- **Inclusion gates** — SSR, i18n, mandatory-JSDoc, and UI-library answers don't fill a
  placeholder; they decide whether a whole file (or a `<!-- SCAFFOLD -->`-marked block
  inside one) is copied at all. A "no" answer means the file/block is deleted, not filled
  with an empty string. Gated files must still be linked from `AGENTS.md` / the relevant
  README when kept, and their links removed when skipped.

## 3. Verify

Run `checklist.md` in full before reporting success.

## Notes on documentation structure (read once, then follow templates/ literally)

- `CLAUDE.md` is always a one-line stub pointing to `AGENTS.md`.
- `AGENTS.md` is the mandatory entry point: a short "Must Read" list plus non-negotiable
  rules, at the project root.
- `.agents/README.md` is a flat index of everything under `.agents/` — the barrel `index.ts`
  equivalent for the docs tree. Every root doc and every subfolder README is linked from it.
  Keep it in sync whenever a conditional file (i18n, agent-surface, platform-adapter) is
  added or skipped.
- Topics that are genuinely one short doc stay flat in `.agents/*.md` (workflow, git-policy,
  documentation, mcp, testing-and-quality, agent-surface, refactoring, progress, i18n,
  accessibility).
- Topics that fan out into several small docs get a subfolder with its own `README.md` hub:
  `.agents/code-style/`, `.agents/architecture/`, `.agents/shared/`, `.agents/core/`,
  `.agents/decisions/`. This is a deliberate deviation from kikita-ui/-docs (which stayed
  fully flat) because this skill's spec calls for per-component and per-concern reuse docs
  that will keep growing after scaffolding.
- `.agents/shared/README.md` registers everything under `src/app/shared/`, split by
  dependency shape: `shared/ui/` (component/directive/pipe, Angular-decorated) and
  `shared/utilities/` (plain functions, zero Angular imports — enforced by an ESLint
  restricted-import rule). SCSS mixins, if SCSS was chosen, live in `styles/mixins/`, not
  under `shared/` at all. `.agents/core/README.md` registers app-wide singletons
  (services/guards/interceptors) under `src/app/core/`. Neither is named "components" —
  both cover more than components, hence the naming.
- `.agents/decisions/README.md` explains when a short ADR is required (layer direction,
  state ownership, routing strategy, public alias scheme) — always generated, starts with
  no ADR files. `AGENTS.md` links it under a conditional "for structural changes" group,
  not the always-read list — same pattern as the `shared/`/`core/` "for component work"
  group.
- All tracked file content — including JSDoc — is English only. No Cyrillic, no mojibake.
- All docs in `.agents/` must read like `angular-code-style.md` in kikita-ui: imperative,
  short, example-backed, ending in a review/verification checklist.
