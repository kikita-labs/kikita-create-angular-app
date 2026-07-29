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
  { path: UserProfileRoute.Root, loadComponent: () => import('./user-profile/user-profile') },
  { path: UserProfileRoute.Edit, loadComponent: () => import('./user-profile-edit/user-profile-edit') },
];
```

```ts
// app.routes.ts
export enum AppRoute {
  Home = 'home',
  UserProfile = 'user',
}

export const routes: Routes = [
  { path: AppRoute.Home, loadComponent: () => import('./features/home/home') },
  { path: AppRoute.UserProfile, loadChildren: () => import('./features/user-profile/user-profile.routes').then((m) => m.USER_PROFILE_ROUTES) },
];
```

- Navigation calls (`router.navigate`, `routerLink`) reference the enum, not a raw string.
- Lazy-load features with `loadComponent`/`loadChildren` — no eagerly-imported feature
  routes in `app.routes.ts`.
- `app.routes.ts` stays a thin top-level map of feature entry points; it never grows to
  contain a feature's internal sub-routes — those belong in that feature's own
  `<feature-name>.routes.ts`.

## Review Checklist

- [ ] No raw route-path string literals outside the enum definition.
- [ ] New feature routes are lazy-loaded.
- [ ] A feature with more than one route (or nested routes) has its own
      `<feature-name>.routes.ts`, referenced from `app.routes.ts` via `loadChildren` — not
      inlined there.
