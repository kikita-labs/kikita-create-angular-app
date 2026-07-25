# Core

Registry of `src/app/core/` singletons: app-wide services, guards, and interceptors that
exist exactly once for the whole app. Not a dumping ground for "things that didn't fit
elsewhere" — see `../architecture/folder-structure.md` for the boundary between `core/` and
`shared/`.

## Registry

| Name | Kind | Path | Doc | Summary |
| --- | --- | --- | --- | --- |
| _(none yet)_ | | | | |

Kind is one of: `service`, `guard`, `interceptor`.

## Adding an entry

1. Build it under `src/app/core/`.
2. Add a row to the table above.
3. If it has a non-trivial API or wiring (e.g. an interceptor's behavior, a service's
   public methods), create `.agents/core/<name>.md`. Simple guards/services can rely on the
   table row alone.

This applies every time a core singleton is added, changed, or removed — see
`../documentation.md`.

## Review Checklist

- [ ] Table has no stale or missing entries.
- [ ] Nothing here that's actually feature-scoped or reusable-but-not-singleton — that
      belongs in `../shared/README.md` instead.
