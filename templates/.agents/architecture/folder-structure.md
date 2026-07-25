# Folder Structure

```
src/
  app/
    core/            # app-wide singletons: guards, interceptors, root services
      platform/      # SSR-safe browser-API adapters, if SSR was chosen — see platform-adapter.md
    shared/
      ui/            # reusable components/directives/pipes — see .agents/shared/README.md
      utilities/     # framework-agnostic helper functions — no Angular imports
    features/
      <feature-name>/
        <component-name>/
          interfaces/
          types/
          constants/
          enums/
          helpers/
          tokens/
          <component-name>.ts
          <component-name>.html
          <component-name>.{{CSS_EXT}}
    app.routes.ts
    app.config.ts
  styles/
<!-- SCAFFOLD: keep the next line only if CSS=native or CSS=SCSS -->
    layers.css       # @layer order declaration, imported first
<!-- SCAFFOLD: keep the next line only if CSS=SCSS -->
    mixins/          # SCSS mixins/functions — see css-architecture.md
<!-- SCAFFOLD: keep the next line only if CSS=Tailwind -->
    tailwind.css     # @tailwind/@import directives + @theme tokens, imported first
  environments/      # non-secret, per-target build config (API base URL, feature flags)
```

- `environments/` holds non-secret, build-time config that differs per target (dev/prod
  API base URL, feature-flag defaults) — verify the current Angular CLI's actual
  mechanism for this (`fileReplacements` in `angular.json`, or its current equivalent)
  rather than assuming an older pattern still applies. Real secrets (API keys, tokens)
  never go here or anywhere tracked — they come from environment variables / CI secrets
  at deploy time, injected outside the repo. See `.env`/`.env.*` handling in
  `../git-policy.md`.

- `app.routes.ts` and `app.config.ts` stay directly under `src/app/`, next to the root
  component — this is the Angular CLI's own default layout and what `ng generate`
  schematics expect. Don't nest them under a `bootstrap/` folder; that adds indirection
  with no benefit for a single-app project (it would only start paying off in a multi-app
  monorepo, which this skill doesn't support — see `SKILL.md`).
- `shared/` splits by dependency shape, not just "reusable": `ui/` holds anything with an
  Angular decorator (component, directive, pipe) — template-facing, DI-aware. `utilities/`
  holds plain functions/classes with zero Angular imports — anything under `core/`,
  `shared/ui/`, or `features/` can depend on a utility, but a utility never depends back on
  Angular. Don't blend the two into one folder; the import direction is exactly what an
  ESLint restricted-import rule needs to enforce automatically (see
  `../testing-and-quality.md`).
- SCSS mixins/functions (if SCSS was chosen) live in `styles/mixins/`, not under
  `app/shared/` — they're not TypeScript and aren't part of the `shared/ui`/`shared/
  utilities` import-boundary story.
<!-- SCAFFOLD: keep the next line only if CSS=Tailwind -->
- Design tokens (if Tailwind was chosen) live in the `@theme` block of `styles/tailwind.css`,
  not scattered across component files — see `css-architecture.md`.
- A feature owns its own components; nothing under `features/<x>/` is imported by another
  feature directly — promote it to `shared/ui/` or `shared/utilities/` first, with a doc
  entry, if it needs reuse.
- `core/` is for things that exist exactly once for the whole app (auth interceptor, root
  error handler) — not a dumping ground for "things that didn't fit elsewhere".

## Review Checklist

- [ ] New shared component/directive/pipe lives in `shared/ui/`, new framework-agnostic
      helper lives in `shared/utilities/` — not mixed.
- [ ] Nothing under `shared/utilities/` imports from `@angular/*`.
- [ ] `app.routes.ts`/`app.config.ts` stay at `src/app/`, not moved into a `bootstrap/`
      folder.
- [ ] Nothing added to `core/` that isn't a genuine app-wide singleton concern.
- [ ] No secret value committed under `environments/` or anywhere else tracked — secrets
      come from env vars/CI secrets, never from a file in the repo.
