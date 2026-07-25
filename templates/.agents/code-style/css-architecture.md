# CSS Architecture

CSS engine: {{CSS}}.

- Use native CSS `@layer` to order cascade priority explicitly, instead of relying on
  source order or specificity fights.
<!-- SCAFFOLD: keep only if a UI library was chosen -->
- {{UI_LIB}} owns its own layers — do not fight them with `!important`; add project layers
  after the library's in the layer order.
- Minimum layer set for this project:

```css
@layer {{PREFIX}}.base, {{PREFIX}}.components, {{PREFIX}}.utilities;
```

- `{{PREFIX}}.base` — resets, global tokens/custom properties, base element styles.
- `{{PREFIX}}.components` — per-component styles.
- `{{PREFIX}}.utilities` — small single-purpose overrides, used sparingly.
- Declare the layer order once, early (e.g. in `src/styles/layers.css`, imported first),
  then author each file's rules inside its matching `@layer` block.
- CSS custom properties (design tokens) are the public styling contract — components read
  `var(--{{PREFIX}}-*)`, never a hardcoded size or color. See
  `component-structure.md` for the token-usage rule.
<!-- SCAFFOLD: keep only if SCSS was chosen -->
- SCSS is an authoring convenience (nesting, mixins, functions) — it compiles to the same
  layered CSS; it is not a separate runtime theme mechanism. Keep mixins under
  `src/styles/mixins/` with a barrel-style single entry import, same discipline as
  `index.ts` barrels for TS.
- One single entrypoint stylesheet imports every layer file, in layer-declaration order.
  Don't scatter ad-hoc `<style>` imports that bypass it.

## Review Checklist

- [ ] New styles land in the correct `@layer`, not unlayered.
- [ ] No `!important` used to fight another layer's specificity.
- [ ] No hardcoded size/color — goes through a `var(--{{PREFIX}}-*)` token.
- [ ] New style file is imported from the single entrypoint, in the right layer order.
