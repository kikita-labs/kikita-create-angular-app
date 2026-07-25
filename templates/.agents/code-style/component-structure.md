# Component Structure

## TypeScript

- Strict types. Avoid `any` — use `unknown` when a value is genuinely uncertain, then
  narrow it before use.
- Prefer inference when the type is obvious; don't annotate what TypeScript already knows.
- No broad casts (`as any`, `as unknown as X`) to paper over a weak type — fix the type
  instead.

## Visibility

- `public` — intentional class API, meant to be used from outside the component.
- `protected` — template-facing only; the template reads it, nothing outside the class
  should.
- `private` — internal implementation detail.
- Make injected dependencies, signals, inputs, outputs, and stable callbacks `readonly`
  unless reassignment is genuinely required.
- Signal APIs only: `input()`/`model()`/`output()`. The legacy `@Input()`/`@Output()`/
  `@ViewChild()`/`@ContentChild()` decorators are banned in new code — use `input()`,
  `model()`, `output()`, `viewChild()`, `contentChild()` instead.
- Host bindings/listeners use the `host` metadata object in `@Component`/`@Directive`, not
  the `@HostBinding()`/`@HostListener()` decorators.
- Forms use the Signal Forms API (`form()`) — not Reactive Forms (`FormGroup`/
  `FormControl`) and not template-driven forms (`ngModel`). If Signal Forms can't cover a
  specific need, stop and say so instead of silently falling back to Reactive Forms.
  - Validation constraints (`min()`, `max()`, `required()`, etc.) go in the form schema,
    never as native HTML attributes (`min`, `max`, `required`) on a `[field]`-bound
    element — Angular derives the DOM attributes and ARIA wiring from the schema; setting
    both duplicates the source of truth and can disagree with it.
  - Render field errors with `@if`, never `[hidden]` — hidden-but-present text isn't
    reliably picked up by assistive tech, and `@if` actually removes it from the DOM.
- Use `inject()` for dependencies, not constructor-parameter injection — see the member
  order below; `inject()` calls are their own ordered group, constructor stays for actual
  initialization logic.

## Service Decorator

- New injectable service classes use Angular's `@Service` decorator, not `@Injectable`.
- Use `@Injectable` only when a specific DI pattern requires it (e.g. a documented edge
  case `@Service` doesn't support) — and note why in a comment when you do.

## Member order

Blank line between every group; a group with nothing to show is skipped, not left as an
empty gap.

1. Inputs (`input()`/`model()`).
2. Outputs (`output()`).
3. `inject()` calls — always `private readonly`. If the template needs a value from an
   injected service, inject it `private`, then expose the needed slice further down as its
   own `protected readonly` (e.g. `protected readonly guildId = this.userService.guildId;`).
4. `protected readonly` plain fields, then `private readonly` plain fields.
5. `protected readonly signal()` fields, then `private readonly signal()` fields.
6. `protected readonly toSignal()` fields, then `private readonly toSignal()` fields.
7. `protected readonly linkedSignal()` fields, then `private readonly linkedSignal()`
   fields.
8. `protected readonly` resource (`resource()`/`rxResource()`/`httpResource()`) fields, then
   `private readonly` resource fields — prefer keeping the resource `private` and deriving a
   `computed()` (plus explicit loading/error state) from it instead of exposing the raw
   resource to the template.
9. `protected readonly form()` fields, then `private readonly form()` fields.
10. `protected readonly computed()` fields, then `private readonly computed()` fields.
11. `constructor()` — only when actually needed. Prefer extracting `effect()` bodies into a
    private method called from the constructor, instead of inlining the effect body.
12. Other lifecycle hooks, if unavoidable.
13. `protected readonly` arrow-function fields, if any (rare — prefer a method unless the
    template needs a bound reference).
14. `protected` methods, then `private` methods.

## Body formatting

- Group statements by purpose; blank line between groups, not inside one.
- Blank line before and after every `if` block.
- Collapse a single-statement `if` onto one line: `if (!user) return;`.
- Blank line before `return`.

```ts
function example() {
  const a = 1;
  const b = 2;

  if (!b) return;

  if (a > b) {
    doSomething();
    doSomethingElse();
  }

  return a + b;
}
```

## Folder structure

No `interface`, `type`, `enum`, or reusable `const` declared inline inside a component
file. If it needs to be exported for reuse, it goes in the matching subfolder:

```
componentName/
  interfaces/
    interface1.ts
    index.ts
  types/
    type1.ts
    index.ts
  constants/
    const1.ts
    index.ts
  enums/
    enum1.ts
    index.ts
  helpers/
    helper1.ts
    index.ts
  tokens/
    token1.ts        # every token ships with a provider function alongside it
    index.ts
  componentName.ts
  componentName.html
  componentName.{{CSS_EXT}}
```

- Decomposition is mandatory. Budgets: component/page TypeScript file ~150 lines target,
  200 hard-review threshold; template ~120 lines; stylesheet ~160 lines; a single function
  ~30 lines and one responsibility. Going over any of these means extract a helper,
  sub-component, or directive — don't just let it grow.
- Every subfolder gets a barrel `index.ts`.
- Component file names follow current Angular convention: no `.component` suffix
  (`user-card.ts`, not `user-card.component.ts`); name for what the thing is.

## Styling

- No hardcoded sizes or colors in component styles.
<!-- SCAFFOLD: keep this whole block only if CSS=native or CSS=SCSS -->
<!-- SCAFFOLD: keep this bullet only if a UI library was chosen -->
- Prefer {{UI_LIB}} primitives and its CSS variables first.
- Where no library covers it, use this project's own token scale — CSS custom properties
  (`var(--{{PREFIX}}-space-3)`, `var(--{{PREFIX}}-color-fg-muted)`) — never a raw px/hex
  value in a component stylesheet.
- Component styles live inside the `{{PREFIX}}.components` `@layer` — see
  `css-architecture.md` for the full layering rules.
<!-- SCAFFOLD: keep this whole block only if CSS=Tailwind -->
<!-- SCAFFOLD: keep this bullet only if a UI library was chosen -->
- Prefer {{UI_LIB}} primitives first; they carry their own Tailwind-compatible classes/vars.
- Where no library covers it, use Tailwind utility classes in the template, reading from the
  `@theme` scale in `styles/tailwind.css` — never a raw px/hex value or an arbitrary-value
  bracket (`w-[13px]`) as a substitute for a missing token; add the token to `@theme` instead.
- No inline `[style]`/`style="..."` and no per-component stylesheet for anything utilities
  already cover. A `componentName.css` file is only for the rare rule Tailwind utilities
  genuinely can't express (e.g. a keyframe animation) — see `css-architecture.md`.

## Review Checklist

- [ ] No inline interface/type/enum/const meant for reuse.
- [ ] Member order matches the list above.
- [ ] No `@Input`/`@Output`/`@ViewChild`/`@ContentChild` decorators; signal equivalents used.
- [ ] No `@HostBinding`/`@HostListener`; `host` metadata object used instead.
- [ ] No constructor-parameter injection; `inject()` used instead.
- [ ] Any form uses Signal Forms (`form()`), not `FormGroup`/`FormControl`/`ngModel`.
- [ ] New service classes use `@Service`, not `@Injectable`, unless a documented DI edge
      case requires the latter.
- [ ] No `any`, no broad casts; `unknown` + narrowing used where the type is uncertain.
- [ ] Component TS ~150 lines target / 200 hard limit, template ~120, stylesheet ~160,
      each function ~30 lines and one responsibility — decompose if over.
- [ ] No hardcoded size/color values.
- [ ] Every subfolder has a barrel `index.ts`.
