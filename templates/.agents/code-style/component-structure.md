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

Blank line between every group **and every subgroup** — including between the `protected`
half and `private` half of the same numbered group (e.g. between `protected readonly
signal()` fields and `private readonly signal()` fields). No two declarations from
different groups or subgroups ever sit on adjacent lines. A group/subgroup with nothing to
show is skipped, not left as an empty gap — never emit a blank line with nothing above or
below it.

```ts
readonly userId = input<string>();

readonly saved = output<void>();

private readonly http = inject(HttpClient);

protected readonly userService = inject(UserService);

protected readonly staticValues = STATIC_VALUES;

private readonly localOnlyValue = 'x';

protected readonly guildId = signal('');

private readonly draftId = signal('');

protected readonly user = toSignal(this.userService.user$);

private readonly rawFeed = toSignal(this.feedService.feed$);

protected readonly isValid = computed(() => this.user() != null);
```

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
- Exception: a run of consecutive single-line guard `if`s (same shape, one condition/return
  each, no other statements between them) stays tight — no blank line between them, only
  before the first and after the last. The tight run reads as one decision table, not
  separate blocks.
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

```ts
function statusToVariant(status: Status) {
  if (status === 'APPROVED') return 'success';
  if (status === 'REJECTED' || status === 'CANCELLED') return 'danger';
  if (status === 'IN_PROGRESS' || status === 'REPORT_PENDING') return 'warning';

  return 'info';
}
```

## Templates

- A signal (or any other call expression, e.g. a computed) read more than once in the same
  template gets bound to a local with `@let`, then every subsequent use reads the local, not
  the signal call again:

```html
@let userValue = user();

<h1>{{ userValue.name }}</h1>
<p>{{ userValue.email }}</p>
```

- A single use stays as a direct call — don't introduce `@let` pre-emptively.
- When the value being tested by `@if` is itself the value the block needs (e.g. a nullable
  signal read, checked for truthiness and then used), bind it with `@if (...; as x)` instead
  of re-reading the signal inside the block:

```html
@if (currentGuild.icon; as icon) {
  <kui-avatar [src]="icon" [name]="currentGuild.name" size="lg" shape="square" />
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
  componentName.opener.ts   # only for a dialog/drawer-style component, see below
```

- A dialog or drawer component opened imperatively ({{UI_LIB}}'s dialog/drawer service, if
  one was chosen) gets a flat `componentName.opener.ts` sibling exporting a single
  `injectXxx()` function — this is the component's public "how to open it" API, not an
  internal implementation detail, so it stays flat next to `componentName.ts` rather than
  inside `helpers/` (`helpers/` is for logic extracted out of the component file for
  size/decomposition reasons).

- Decomposition is mandatory. Budgets: component/page TypeScript file ~150 lines target,
  200 hard-review threshold; template ~120 lines; stylesheet ~160 lines; a single function
  ~30 lines and one responsibility. Going over any of these means extract a helper,
  sub-component, or directive — don't just let it grow.
- Every subfolder gets a barrel `index.ts`.
- Component file names follow current Angular convention: no `.component` suffix
  (`user-card.ts`, not `user-card.component.ts`); name for what the thing is.
- The class name drops the suffix too, not just the file: `export class UserCard`, not
  `export class UserCardComponent`. This matches the Angular CLI's own `ng generate` default
  since v20 (the `addTypeToClassName` schematic option, defaulted off) — the file-only
  reading is a common misconception, and the CLI's actual generated code is the source of
  truth here, not just the style guide prose. Same rule for directives/pipes/services: no
  `Directive`/`Pipe`/`Service` suffix on the class either.
  - Known tradeoff: dropping the suffix on both file and class can produce a namespace
    collision (e.g. `User` the entity type vs. `User` the component class, both imported in
    the same file). Resolve it with an import alias at the call site
    (`import { User as UserCard } from './user-card'`) — don't reintroduce the suffix
    project-wide just to dodge one collision.

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
- [ ] Signal/computed called 2+ times in one template bound via `@let` instead of repeated
      calls.
- [ ] Value tested and used inside `@if` bound via `@if (...; as x)` instead of re-reading
      it.
- [ ] Every subfolder has a barrel `index.ts`.
- [ ] Class name has no `Component`/`Directive`/`Pipe`/`Service` suffix, matching the file
      name convention — not just the file, the class too.
- [ ] Dialog/drawer component's `injectXxx()` opener is a flat `componentName.opener.ts`
      sibling, not tucked inside `helpers/`.
- [ ] Consecutive single-line guard `if`s of the same shape have no blank lines between
      them; blank line only before the first and after the last of the run.
