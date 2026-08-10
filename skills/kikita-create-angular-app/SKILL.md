---
name: kikita-create-angular-app
description: Scaffold a new Angular project (latest stable) with a full .agents/ documentation tree, code style, MCP, and git policy pre-wired — retrofit that same .agents/ tree onto an existing Angular project this skill didn't create — or, in a project this skill already scaffolded/adopted, pull and merge upstream .agents/ doc updates. Use when the user asks to init/bootstrap/create a new Angular app, invokes /kikita-create-angular-app in an empty or near-empty directory, asks to add/generate/retrofit AGENTS.md or .agents/ docs onto an existing Angular project, or asks to update/sync/refresh the project's agent docs / .agents/ conventions in a project this skill previously touched.
---

# kikita-create-angular-app

Bootstraps a new Angular project and generates its `AGENTS.md` / `.agents/` documentation
tree so any AI agent (Claude Code, Codex, etc.) working in the project afterwards has a
complete, self-maintaining source of truth.

## 0. Mode detection (always do this first)

Three possible states for the target directory — check in this order:

1. **`.agents/.kikita-scaffold.json` present** → this project was already scaffolded or
   adopted by this skill. Follow `update.md` — do not re-run the questionnaire, do not
   re-scaffold, do not re-run `adopt.md`. `update.md` handles pulling the latest template,
   diffing since the recorded commit, and merging changes into the project's (possibly
   customized) `.agents/` files.
2. **Empty or near-empty directory** (no `package.json`, no `angular.json`, nothing but
   maybe a `.git/`/README the user just created) → fresh init. Continue with section 1
   below, then `plan.md`.
3. **Existing project, no scaffold record** (`package.json`/`angular.json` present, no
   `.agents/.kikita-scaffold.json`) → this is someone else's (or an older, pre-this-skill)
   Angular project. Do **not** run `plan.md` — it assumes an empty directory and will try to
   `ng new` over a real project. Follow `adopt.md` instead: it generates the `.agents/` tree
   against what's actually there, inferring questionnaire answers from the existing config
   rather than scaffolding new tooling, and asks before changing anything that already exists.

## 0.5 Preconditions (fresh init only)

- Confirm the working directory is empty or the user explicitly wants to init here.
- If the directory turns out to be non-empty when you check, stop and re-route to case 3
  above (`adopt.md`) instead of forcing the init path — don't ask the user to just "confirm
  it's fine to init here" when what's actually needed is the adopt flow.

## 1. Questionnaire (always ask before touching the filesystem)

Follow `plan.md` step by step from here. Do not skip the questionnaire. Do not skip MCP
install. When done, run through `checklist.md` before telling the user it's finished.

Ask the user, one round, all at once:

1. **CSS**: native CSS (custom properties), SCSS, or Tailwind CSS?
   - Tailwind is a structurally different branch, not a flavor of the other two: no
     `@layer {{PREFIX}}.*` scheme, no hand-written `:root` tokens, no SCSS mixins — see the
     `CSS=Tailwind` scaffold blocks in `templates/.agents/code-style/css-architecture.md`,
     `component-structure.md`, and `templates/.agents/architecture/folder-structure.md`.
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
- **Inclusion gates** — SSR, i18n, mandatory-JSDoc, UI-library, and CSS-engine answers don't
  fill a placeholder; they decide whether a whole file (or a `<!-- SCAFFOLD -->`-marked block
  inside one) is copied at all. A "no" answer (or, for CSS, the two non-chosen engines)
  means the file/block is deleted, not filled with an empty string. Gated files must still
  be linked from `AGENTS.md` / the relevant
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
