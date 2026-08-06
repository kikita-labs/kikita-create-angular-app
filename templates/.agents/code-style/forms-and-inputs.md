# Forms and Inputs

Angular Signal Forms (stable in v22, `@angular/forms/signals`) are the default for anything
that qualifies as a form — a named group of fields with validation, submission, or
cross-field rules. Reach for `[(ngModel)]` two-way binding only when there's no real "form"
to speak of. Never wire up a control by hand with `(change)`/`(input)` plus manual
`event.target` casting — that's reimplementing what `form()`/`[formField]` (or `ngModel`)
already does for free, worse.

```ts
// Wrong — manual event handling, no reason for it to exist
<input kuiCheckbox type="checkbox" [checked]="dontAskAgain()" (change)="onDontAskAgainChange($event)" />
protected readonly onDontAskAgainChange = (event: Event) => {
  this.dontAskAgain.set((event.target as HTMLInputElement).checked);
};
```

```ts
// Right — single standalone toggle, not part of a form: signal stays a signal
protected readonly dontAskAgain = signal(false);
```
```html
<input
  kuiCheckbox
  type="checkbox"
  [ngModel]="dontAskAgain()"
  (ngModelChange)="dontAskAgain.set($event)"
/>
```

`FormsModule`'s `[(ngModel)]` banana-in-box sugar desugars to
`[ngModel]="x" (ngModelChange)="x = $event"` — a plain reassignment, not a `.set()` call.
`ngModel` itself has no signal-aware two-way binding (unlike a component's own `model()`
input) — binding the sugar form directly to a `WritableSignal` either fails to type-check
or silently clobbers the signal reference on change (tracked upstream:
[angular/angular#61419](https://github.com/angular/angular/issues/61419),
[#57771](https://github.com/angular/angular/issues/57771)). Don't work around this by
demoting the field to a plain mutable property — expand the sugar by hand into
`[ngModel]` + `(ngModelChange)` instead, calling `.set()` explicitly. The control's state
stays a signal like everywhere else in the codebase, and `ngModelChange` already emits the
typed value (`boolean` for a checkbox) — no `event.target` cast needed either.

```ts
// Right — this is a form: Signal Forms owns it end to end
<input kuiCheckbox type="checkbox" [formField]="preferencesForm.dontAskAgain" />
```

## When to Use What

- **Signal Forms (`form()` + `[formField]`)** — any input that's part of a form: 2+ related
  fields, anything with validation, anything with a submit action, anything with
  cross-field or conditional (disabled/hidden/readonly) rules. This is the default; reach
  for it first.
- **`[(ngModel)]`** — a genuinely standalone control with no group, no validation, and no
  submit step around it (a single settings toggle, a filter checkbox next to a list). Don't
  build a one-field `form()` just to avoid `ngModel` here — that's the opposite mistake.
- **Never**: a manual `(change)`/`(input)` listener that reads `event.target` and casts it,
  for either of the above cases. If you're casting `event.target as HTMLInputElement`,
  something above should have been `[formField]` or `[(ngModel)]` instead.

## Building a Signal Form

`form()` takes a model **signal** and returns a `FieldTree` that mirrors the model's shape.
The model must be typed by a named interface — never an inline object type — placed per
`../architecture/folder-structure.md` (a feature-root `interfaces/` folder if shared by 2+
components in that feature, otherwise co-located in the owning component's own
`interfaces/` subfolder; promote to `shared/` only on genuine cross-feature reuse).

```ts
// interfaces/user-profile-form.interface.ts
export interface UserProfileFormModel {
  readonly name: string;
  readonly email: string;
  readonly age: number;
}
```

The schema function itself lives in `helpers/componentName.schema.ts` — see
`component-structure.md`'s folder structure section. Unlike `componentName.opener.ts`
(consumed by other components, so it stays a flat public sibling), a schema is only ever
consumed by its own component's `form()` call — exactly the "logic extracted out of the
component file" `helpers/` exists for.

Give the model a default-state constant in the component's own `constants/` subfolder
instead of inlining the initial object in the component (or the schema file). This keeps a
single source of truth to `.set()`/`reset()` back to later, instead of re-typing the
same literal object wherever the form needs resetting:

```ts
// constants/user-profile-form-default-state.const.ts
export const USER_PROFILE_FORM_DEFAULT_STATE: UserProfileFormModel = {
  name: '',
  email: '',
  age: 0,
};
```

```ts
protected readonly profileModel = signal<UserProfileFormModel>(USER_PROFILE_FORM_DEFAULT_STATE);

protected readonly profileForm = form(this.profileModel, profileFormSchema);
```

- Never seed the model with `null`/`undefined` for a field the template renders — use `''`
  for strings, `0` for numbers, `[]` for arrays. A field a form can't legitimately fill in
  yet belongs in a separate domain-model type, converted into the form model at the
  boundary (see "Domain Model vs Form Model" below), not `null` sprinkled through the form
  model itself.
- Fields are `FieldTree` nodes — call them as functions to read state:
  `profileForm.email().value()`, `profileForm.email().touched()`,
  `profileForm.email().errors()`. `profileForm.email.value()` (no call) is a mistake, not a
  shorthand — the uncalled node is the tree, not the state.
- Never `.set()` a field directly (`profileForm.email.set(x)`) — update the backing model
  signal instead; the form derives from it.
- In the template, bind the control with `[formField]`, not `[value]`/`[checked]` +
  a change handler:

```html
<input [formField]="profileForm.name" />
@if (profileForm.name().touched() && profileForm.name().invalid()) {
  @for (error of profileForm.name().errors(); track error.kind) {
    <p>{{ error.message }}</p>
  }
}
```

- Don't set `[disabled]`, `[readonly]`, or `min`/`max` as plain template attributes on a
  `[formField]`-bound control — express them as schema rules (`disabled()`, `readonly()`,
  `min()`/`max()` validators) instead; `[formField]` syncs those attributes from field state
  automatically, and a hand-set attribute fights it.
- Exception: a static `value` on a radio/checkbox input is required to identify which
  option it represents — that's not a state attribute, keep it.

## Schema and Validators

Pass a schema function as `form()`'s second argument. Extract it to a named, exported
function — never an inline arrow — so it's reusable and independently testable:

```ts
// helpers/user-profile-form.schema.ts
export function profileFormSchema(path: SchemaPath<UserProfileFormModel>) {
  required(path.name, { message: 'Name is required' });
  required(path.email, { message: 'Email is required' });
  email(path.email, { message: 'Enter a valid email address' });
  min(path.age, 18, { message: 'Must be 18 or older' });
}
```

- Built-in validators: `required`, `email`, `min`, `max`, `minLength`, `maxLength`,
  `pattern`. Only `required()` takes a `when` option directly — any other validator's
  conditional application goes through `applyWhen()`, not a `when` clause bolted onto the
  validator.
- Custom validation: `validate(path, ({ value, valueOf, state }) => ...)`, returning
  `{ kind, message }` or `undefined` (never `null` — `undefined` is the "no error" value).
  Read other fields with `valueOf(otherPath)` for dependency-tracked cross-field checks —
  never read a sibling field's raw signal value outside the context object.
- Async validation (uniqueness checks, server-side rules): `validateAsync(path, { params,
  factory, onSuccess, onError })`, backed by a `resource()`/`rxResource()`. `onError` is
  required — don't leave a rejected validation request unhandled.

### Reusable Validators — Don't Duplicate Across Forms

Never re-type the same `required`/`pattern`/cross-field logic in every form that happens to
share a field shape. Two composition tools cover this:

**A standalone validator function**, when the rule itself (not the whole field) repeats:

```ts
// shared/utilities/validators/us-zip-code.validator.ts
export function usZipCode(path: FieldPath<string>) {
  pattern(path, /^\d{5}$/, { message: 'Enter a 5-digit ZIP code' });
}
```

```ts
export function shippingAddressSchema(path: SchemaPath<AddressFormModel>) {
  required(path.street);
  usZipCode(path.zip);
}
```

**A reusable partial schema + `apply()`**, when a whole sub-shape of fields (and its rules)
repeats across otherwise-different forms — define it against a narrow, structurally-typed
interface covering just the shared fields, then `apply()` it into each concrete form's
schema:

```ts
// contact-info.schema.ts
export interface ContactInfo {
  readonly email: string;
  readonly phone: string;
}

export function contactInfoSchema(path: SchemaPath<ContactInfo>) {
  required(path.email);
  email(path.email);
  pattern(path.phone, /^\+?\d{7,15}$/);
}
```

```ts
export function supplierFormSchema(path: SchemaPath<SupplierFormModel>) {
  required(path.companyName);
  apply(path, contactInfoSchema); // SupplierFormModel structurally satisfies ContactInfo
}
```

- `applyWhen(path, ({ valueOf }) => condition, subSchema)` — apply a set of rules only when
  a condition holds (e.g. extra fields required only when a checkbox is checked). Don't
  simulate this by hand-toggling `required`'s `when` option across multiple validators.
- `applyEach(itemPath, (item) => ...)` — apply one schema to every item of an array field
  (single-argument callback only — no index parameter).

## Domain Model vs Form Model

When the data a form edits comes from an API and has optional/nullable fields the form
itself must never see as `null` (per "never seed with null" above), keep two interfaces: a
`*DomainModel` matching what the backend returns, and a `*FormModel` where every field the
template renders has a concrete default. Convert at the boundary with a plain mapping
function, and bridge the two with `linkedSignal()`:

```ts
protected readonly formModel = linkedSignal(() => toFormModel(this.domainModel()));
protected readonly editForm = form(this.formModel, editFormSchema);
```

Don't let `null`/`undefined` leak from the domain model into the form model just to avoid
writing the conversion function.

## Submission

```ts
protected onSubmit(): void {
  submit(this.profileForm, async () => {
    await firstValueFrom(this.profileApi.save(this.profileModel()));
  });
}
```

- `submit()`'s callback must be `async`/return a `Promise` — it isn't a fire-and-forget
  event handler.
- `submit()` marks every field touched before running validation, so error messages that
  depend on `touched()` show up correctly on a first submit attempt without a separate
  "mark all as touched" step.
- Gate the submit control on form state, not a hand-rolled flag:
  `[disabled]="profileForm().invalid() || profileForm().pending()"`.

## Review Checklist

- [ ] No manual `(change)`/`(input)` handler casting `event.target` where `[formField]` or
      `[(ngModel)]` belongs instead.
- [ ] Multi-field/validated/submittable input uses Signal Forms; `[(ngModel)]` is reserved
      for genuinely standalone controls with no group, validation, or submit step.
- [ ] A standalone toggle's state stays a signal — bound via expanded `[ngModel]` +
      `(ngModelChange)="sig.set($event)"`, never the `[(ngModel)]="sig"` banana-in-box
      sugar (it reassigns the property directly and doesn't call `.set()`) and never
      demoted to a plain mutable field just to dodge that limitation.
- [ ] Form model is a named interface (never inline), placed per
      `../architecture/folder-structure.md` — not `null`/`undefined` seeded on any field the
      template renders.
- [ ] Field state read by calling the node (`field().value()`), never the uncalled tree
      (`field.value()`); field values changed via the model signal, never `field.set()`.
- [ ] Schema is a named, exported function, not an inline arrow passed to `form()`.
- [ ] `[disabled]`/`[readonly]`/`min`/`max` expressed as schema rules, not plain template
      attributes, on any `[formField]`-bound control (radio/checkbox static `value`
      excepted).
- [ ] Repeated validation rule extracted to a standalone validator function; repeated
      sub-shape of fields extracted to a reusable schema applied via `apply()`/
      `applyWhen()`/`applyEach()` — no copy-pasted validator logic across forms.
- [ ] Custom `validate()` returns `undefined` for "valid", never `null`.
- [ ] `validateAsync()` has an `onError` handler.
- [ ] `submit()` callback is `async`; submit control disabled on `form().invalid() ||
      form().pending()`, not a separate hand-rolled flag.
