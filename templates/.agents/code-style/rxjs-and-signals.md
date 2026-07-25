# RxJS vs Signals

- Signals are the default for reactive state. Prefer `signal()`, `computed()`,
  `rxResource()`/`httpResource()` over RxJS for state and data fetching.
- RxJS is still the right tool for one-shot mutations (POST/PATCH/PUT/DELETE) triggered by
  a user action, where a resource doesn't fit.
- Never call `.subscribe({ next, error })` for a mutation. Put error handling and cleanup
  in the pipe and subscribe with a plain callback:

```ts
this.userService
  .save(user)
  .pipe(
    catchError(() => {
      this.error.set('Save failed');
      return of(null);
    }),
    finalize(() => this.saving.set(false)),
  )
  .subscribe((result) => {
    if (result) this.saved.set(true);
  });
```

- Don't use a `Subject` as a state store — that's what `signal()` is for.
- Reading a reactive route param: use `toSignal(route.paramMap)`, never
  `route.snapshot.paramMap.get(...)` as a field initializer — it goes stale when the router
  reuses the component instance across a params-only navigation.
- Always clean up manual subscriptions with `takeUntilDestroyed()` if you can't avoid
  subscribing directly.

## Service Shape

- One responsibility per service — a service that both fetches user data and manages
  toast notifications is two services wearing one name.
- Expose `readonly` signals for state and plain methods for commands (`save()`,
  `refresh()`) — never expose a raw `WritableSignal` for external code to `.set()` directly.
- Use `effect()` only for actual side effects at a system boundary (syncing to
  `localStorage` via a platform adapter, logging, analytics) — not as a way to compute one
  piece of state from another. If you're tempted to `effect(() => this.b.set(f(this.a())))`,
  use `computed()` instead.
- Never copy one signal's value into another writable signal when `computed()` already
  expresses the relationship — that's a stale-state bug waiting to happen.

## Cross-Feature State

State that's genuinely shared across features (not just reused UI) lives in a `@Service`
under `core/`, exposing `readonly` signals per the Service Shape rules above — this is the
default. Do not reach for NgRx or a similar external store unless the project has grown
enough state, with enough cross-cutting update logic, that a plain signal-service can no
longer keep it manageable — that's a decision worth an ADR (see `../decisions/README.md`,
"state ownership"), not something to pull in speculatively at scaffold time.

## Review Checklist

- [ ] State is signals/computed/resource-based, not an RxJS store.
- [ ] Mutations use pipe-based error handling, not `.subscribe({ next, error })`.
- [ ] No `Subject` used as a state store.
- [ ] Reactive route params use `toSignal`, not a snapshot field initializer.
- [ ] Services expose readonly signals + command methods, not raw writable signals.
- [ ] No signal-copying where `computed()` would do; `effect()` used only for boundary
      side effects.
