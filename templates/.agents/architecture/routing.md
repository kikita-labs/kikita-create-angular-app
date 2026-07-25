# Routing

- Route paths and route names are defined once as an enum, never repeated as string
  literals across the codebase.

```ts
export enum AppRoute {
  Home = 'home',
  UserProfile = 'user/:id',
}
```

```ts
export const routes: Routes = [
  { path: AppRoute.Home, loadComponent: () => import('./home/home') },
  { path: AppRoute.UserProfile, loadComponent: () => import('./user-profile/user-profile') },
];
```

- Navigation calls (`router.navigate`, `routerLink`) reference the enum, not a raw string.
- Lazy-load features with `loadComponent`/`loadChildren` — no eagerly-imported feature
  routes in `app.routes.ts`.

## Review Checklist

- [ ] No raw route-path string literals outside the enum definition.
- [ ] New feature routes are lazy-loaded.
