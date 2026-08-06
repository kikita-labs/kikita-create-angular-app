# Routing

- Route paths and route names are defined once as an enum, never repeated as string
  literals across the codebase.

```ts
export enum AppRoute {
  Home = 'home',
  UserProfile = 'user/:id',
}
```

- Every feature under `features/<feature-name>/` owns its routes in a sibling
  `<feature-name>.routes.ts` — `app.routes.ts` never lists a feature's internal paths
  directly, it only points at that file via `loadChildren`.

```ts
// features/user-profile/user-profile.routes.ts
export enum UserProfileRoute {
  Root = '',
  Edit = 'edit',
}

export const USER_PROFILE_ROUTES: Routes = [
  { path: UserProfileRoute.Root, loadComponent: () => import('./pages/user-profile-view/user-profile-view') },
  { path: UserProfileRoute.Edit, loadComponent: () => import('./pages/user-profile-edit/user-profile-edit') },
];
```

Every component a route above loads lives under `pages/<page-name>/` — see
`architecture/folder-structure.md` for the full shape. A non-routed component the feature
uses internally (e.g. a card rendered inside `user-profile-view`) is never imported here;
it lives under `components/<component-name>/`, a sibling kind-folder of `pages/`, wired up
by the page component itself, not by the router.

```ts
// features/home/home.routes.ts
export enum HomeRoute {
  Root = '',
}

export const HOME_ROUTES: Routes = [
  { path: HomeRoute.Root, loadComponent: () => import('./pages/home/home') },
];
```

```ts
// app.routes.ts
export enum AppRoute {
  Home = 'home',
  UserProfile = 'user',
}

export const routes: Routes = [
  { path: AppRoute.Home, loadChildren: () => import('./features/home/home.routes').then((m) => m.HOME_ROUTES) },
  { path: AppRoute.UserProfile, loadChildren: () => import('./features/user-profile/user-profile.routes').then((m) => m.USER_PROFILE_ROUTES) },
];
```

Even a single-route feature like `home` above still gets its own `<feature-name>.routes.ts`
and its entry component still lives under `pages/` — there is no exception for a feature
with just one route, on either count. `app.routes.ts` never calls `loadComponent` directly
on a feature's page; it only ever points at that feature's routes file via `loadChildren`.
This costs one small file per feature, but keeps every feature uniform (no "does this one
have a routes file or not" lookup) and means adding a second route to a feature later never
requires touching `app.routes.ts` — only that feature's own routes file. Angular's own
routing guide recommends this for the same reason: a route config that's minimal today
tends to grow, and a routing file for every feature is one less thing to restructure later.

- Navigation calls (`router.navigate`, `routerLink`) reference the enum, not a raw string.
- Lazy-load features with `loadChildren` — no eagerly-imported feature routes in
  `app.routes.ts`, and no `loadComponent` in `app.routes.ts` pointing at a feature's page
  directly.
- `app.routes.ts` stays a thin top-level map of feature entry points, each one a
  `loadChildren` pointer; it never grows to contain a feature's internal sub-routes — those
  belong in that feature's own `<feature-name>.routes.ts`.

## Review Checklist

- [ ] No raw route-path string literals outside the enum definition.
- [ ] New feature routes are lazy-loaded.
- [ ] Every feature under `features/<feature-name>/` — including a single-route one — has
      its own `<feature-name>.routes.ts`, referenced from `app.routes.ts` via `loadChildren`.
      `app.routes.ts` never calls `loadComponent` on a feature's page directly.
