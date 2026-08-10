# Adopt Plan

Runs when the skill is invoked inside an existing Angular project that has no
`.agents/.kikita-scaffold.json` — a project this skill didn't scaffold. Generates the
`.agents/` documentation tree against what's actually there instead of assuming a blank slate.
Never runs `ng new`, never installs/replaces tooling the project already made its own choice
about (ESLint config, test runner, package manager) — this is a docs retrofit, not a
re-scaffold. If the project already has `.agents/.kikita-scaffold.json`, this is the wrong
file: use `update.md` instead.

## 1. Confirm scope before touching anything

This modifies a real, already-working project. Before writing a single file:

- Tell the user what you're about to do: generate `AGENTS.md` + `.agents/*` describing the
  project's actual current conventions, inferred from its config — not impose the skill's
  defaults over an established codebase.
- Ask explicitly if this is wanted, and whether it's fine to also install the pieces this
  skill considers non-negotiable but the project is missing entirely (Husky pre-commit hooks,
  `angular-mcp`) — offer these as opt-in additions, not silent installs, since a project that
  runs fine without them may have a reason.

## 2. Infer questionnaire answers from the existing project

Don't ask the full questionnaire from `SKILL.md` section 1 blind — read the project first and
only ask about what genuinely can't be determined by inspection:

- **CSS**: `angular.json` → `projects.*.architect.build.options.style` (`scss` vs `css`); if
  `css`, check for `tailwindcss`/`@tailwindcss/postcss` in `package.json` to tell native CSS
  from Tailwind.
- **UI library**: check `package.json` dependencies for `@kikita-labs/ui`, `@taiga-ui/*`,
  `primeng`; none found → "none".
- **Tests**: `package.json` devDependencies for `vitest`, `jasmine-core`/`karma`,
  `@playwright/test`; a `test`/`e2e` script existing with no matching runner dependency is a
  signal the project's test setup is incomplete or was removed — flag it, don't guess.
- **SSR**: `@angular/ssr` in dependencies, or `provideServerRendering`/`app.server.ts` present.
- **i18n**: `@angular/localize`, `@jsverse/transloco`, `@ngx-translate/core`, or similar in
  dependencies; record which one — do not assume it matches this skill's current fixed
  choice, and never silently swap it during a later update.
- **JSDoc policy**: sample a few exported symbols in `src/app/` — consistently documented
  means treat as "yes", otherwise "no". This one is a judgment call; say so when reporting it.
- **Prefix**: `angular.json` → `projects.*.prefix`.
- **Package manager**: whichever lockfile is present (`pnpm-lock.yaml` / `package-lock.json`
  / `yarn.lock`) — exactly one should exist; flag it if more than one does instead of picking.
- **Git policy**: ask — this isn't inferrable from the repo.

Report the inferred answers back to the user before generating anything, so they can correct
a wrong guess (e.g. i18n library detected but not actually in active use).

## 3. Generate the documentation tree

Follow `SKILL.md` section 2 / `plan.md` step 7's file list and placeholder-filling rules, using
the answers from step 2 instead of a fresh questionnaire. Differences from a fresh scaffold:

- Do not touch `angular.json`, `package.json` scripts, ESLint config, or any existing tooling
  file — describe what's actually configured in the generated docs, don't change it to match
  the template's defaults. If something the docs assume is missing entirely (e.g. no Prettier
  config at all), note it in the relevant doc rather than silently installing it, unless the
  user opted in during step 1.
- `.agents/code-style/*.md`, `.agents/architecture/*.md` etc. must describe the project's
  real folder structure and patterns as they exist today, not the template's example layout —
  read `src/app/` first (`shared/`, `core/`, `features/` or whatever's actually there) and
  adapt the doc content, don't paste the template verbatim.
- Skip `git init` — the project already has its history; just make sure the new `.agents/`
  files get committed in a normal commit once the user reviews them.

## 4. Write the scaffold record

Same shape as a fresh scaffold's `.agents/.kikita-scaffold.json` (`plan.md` step 10), plus a
marker that this was adopted, not scaffolded, and when:

```json
{
  "skill": "kikita-create-angular-app",
  "scaffoldedFromCommit": "<skill repo HEAD at adoption time>",
  "adopted": true,
  "adoptedAt": "{{DATE}}",
  "answers": { "...": "as inferred/confirmed in step 2" }
}
```

`scaffoldedFromCommit` being the *adoption*-time commit (not a real historical scaffold point)
is expected and correct — `update.md` only needs a starting point to diff forward from, it
doesn't need real history predating adoption.

## 5. Verify and hand back

Run the "Documentation" section of `checklist.md` (not "Setup"/"Tooling" — those assume a
fresh scaffold's exact toolchain, which adoption deliberately doesn't touch). Report what was
generated, what was inferred vs. confirmed by the user, and anything flagged as incomplete in
step 2 (missing test runner, ambiguous JSDoc coverage, multiple lockfiles) so the user can
follow up.
