# Folder Structure

```
public/             # served at the site root as-is (favicon.ico, robots.txt, manifest) —
                     # no import path, referenced by absolute URL (e.g. /favicon.ico)
src/
  assets/            # static files referenced from code/templates, served under /assets —
                      # images grouped by domain (assets/<domain>/), not dumped flat
  app/
    core/            # app-wide singletons: guards, interceptors, root services
      guards/        # route guards (CanActivateFn, etc.)
      interceptors/  # HttpInterceptorFn functions
      services/      # app-wide singleton services (@Service, providedIn: root)
      platform/      # SSR-safe browser-API adapters, if SSR was chosen — see platform-adapter.md
    shared/
      ui/            # reusable components/directives/pipes — see .agents/shared/README.md
      utilities/     # framework-agnostic helper functions — no Angular imports
    features/
      <feature-name>/
        <feature-name>.routes.ts   # feature's own routes — see routing.md
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

- `public/` vs `src/assets/` — two different jobs, both wired in `angular.json`'s `assets`
  config (verify the current Angular CLI's actual copy paths rather than assuming):
  - `public/` is for files the browser or a crawler expects at a fixed root URL —
    `favicon.ico`, `robots.txt`, a web manifest. Nothing in app code imports from here by
    path; it's referenced by absolute URL only.
  - `src/assets/` is for static files the app references from code or templates —
    images, locale JSON if i18n is used. Group by domain under `assets/<domain>/` rather
    than dumping every file flat into `assets/`.

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
- A feature is a business domain (e.g. `user-profile`, `cart`, `missions`), not a UI
  component — it's the unit that owns a URL sub-tree and a chunk of the domain, and it can
  hold several components inside it. Don't create a `features/<x>/` folder per component;
  group components that serve one domain under one feature folder instead.
- A feature owns its own components; nothing under `features/<x>/` is imported by another
  feature directly — promote it to `shared/ui/` or `shared/utilities/` first, with a doc
  entry, if it needs reuse.
- `core/` is for things that exist exactly once for the whole app (auth interceptor, root
  error handler) — not a dumping ground for "things that didn't fit elsewhere". Split by
  kind, same reasoning as the `shared/ui`/`shared/utilities` split: `guards/`,
  `interceptors/`, `services/` (plus `platform/` if SSR). Don't leave singleton files flat
  directly under `core/` once there's more than one of a kind — one guard and one service
  in two different flat files is still "two kinds", not "not enough to bother splitting".
- One responsibility per service (see `code-style/rxjs-and-signals.md`, "Service Shape") —
  a service that wraps HTTP calls for three unrelated domains is three services wearing one
  name. Split a growing HTTP-client service by domain (e.g. `user-api.service.ts`,
  `missions-api.service.ts`) once it's doing more than one thing, even though every one of
  them still lives in `core/services/` — this is a file-organization split, not a
  layer-direction change, so it doesn't need an ADR by itself.

## Review Checklist

- [ ] New shared component/directive/pipe lives in `shared/ui/`, new framework-agnostic
      helper lives in `shared/utilities/` — not mixed.
- [ ] Nothing under `shared/utilities/` imports from `@angular/*`.
- [ ] `app.routes.ts`/`app.config.ts` stay at `src/app/`, not moved into a `bootstrap/`
      folder.
- [ ] Feature with its own routes has `<feature-name>.routes.ts` inside its folder — see
      `routing.md` — not routes inlined in `app.routes.ts`.
- [ ] Nothing added to `core/` that isn't a genuine app-wide singleton concern; guards,
      interceptors, and services each live in their own `core/<kind>/` subfolder.
- [ ] No service mixes more than one unrelated responsibility — split by domain instead.
- [ ] No secret value committed under `environments/` or anywhere else tracked — secrets
      come from env vars/CI secrets, never from a file in the repo.
- [ ] Root-URL files (favicon, robots.txt, manifest) go in `public/`; code/template-
      referenced static files go in `src/assets/`, grouped by domain, not dumped flat.
