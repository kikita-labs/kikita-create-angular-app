# CSS Architecture

CSS engine: {{CSS}}.

<!-- SCAFFOLD: keep this whole block only if CSS=native or CSS=SCSS -->
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
- Declare the layer order once, early (e.g. in `src/styles/layers.{{CSS_EXT}}`, imported first),
  then author each file's rules inside its matching `@layer` block.
- CSS custom properties (design tokens) are the public styling contract — components read
  `var(--{{PREFIX}}-*)`, never a hardcoded size or color. See
  `component-structure.md` for the token-usage rule.
- Any grouped set of tokens that doesn't belong inline in the entrypoint gets its own file
  under `src/styles/`, imported once from the entrypoint — the pattern isn't limited to
  `layers.{{CSS_EXT}}`/mixins. Typographic tokens (font sizes, line-heights, weights, families) live
  in `styles/typography.{{CSS_EXT}}` as custom properties inside `{{PREFIX}}.base`, consumed
  via `var(--{{PREFIX}}-text-*)` — components never hand-roll `font-size`/`line-height` per
  instance. If a UI library was chosen and it ships a typography primitive, use that instead
  of these tokens — see `ui-library-usage.md`.
<!-- SCAFFOLD: keep only if SCSS was chosen -->
- SCSS is an authoring convenience (nesting, mixins, functions) — it compiles to the same
  layered CSS; it is not a separate runtime theme mechanism. Keep mixins under
  `src/styles/mixins/` with a barrel-style single entry import, same discipline as
  `index.ts` barrels for TS.
- One single entrypoint stylesheet imports every layer file, in layer-declaration order.
  Don't scatter ad-hoc `<style>` imports that bypass it.
- No inline styles (`[style]` binding or `style="..."` attribute) in templates. They bypass
  the layer cascade and the token contract, and can't be overridden by `@layer` rules. Use
  a class in the matching layer instead; if the value is truly dynamic and can't be a static
  class, bind a CSS custom property (`[style.--foo]`) and consume it via `var(--foo)` inside
  a real stylesheet rule.

## Review Checklist

- [ ] New styles land in the correct `@layer`, not unlayered.
- [ ] No `!important` used to fight another layer's specificity.
- [ ] No hardcoded size/color — goes through a `var(--{{PREFIX}}-*)` token.
- [ ] New style file is imported from the single entrypoint, in the right layer order.
- [ ] No `[style]`/`style="..."` in templates — dynamic values go through a CSS custom
      property, not an inline style.
- [ ] No hand-rolled `font-size`/`line-height` per component — typography goes through
      `styles/typography.{{CSS_EXT}}` tokens, or the UI library's typography primitive if
      one was chosen.

<!-- SCAFFOLD: keep this whole block only if CSS=Tailwind -->
- Styling is utility-first: compose Tailwind classes directly in templates instead of
  writing per-component stylesheets. Tailwind generates its own cascade layers
  (`@layer theme, base, components, utilities`) via the single `@import "tailwindcss"` in
  `src/styles/tailwind.css` — do not hand-roll a parallel `@layer {{PREFIX}}.*` scheme
  alongside it; there is one layer system, and it's Tailwind's.
- Design tokens live in the `@theme` block of `src/styles/tailwind.css`
  (`--color-{{PREFIX}}-fg-muted`, `--spacing-{{PREFIX}}-3`, etc.), not as hand-written
  `:root` custom properties — Tailwind turns each `@theme` entry into both a CSS variable
  and a utility class (`text-{{PREFIX}}-fg-muted`), so there is exactly one place a token
  is defined.
<!-- SCAFFOLD: keep only if a UI library was chosen -->
- {{UI_LIB}} owns its own `@theme`/class names — extend them in this project's `@theme`
  block rather than duplicating; don't fight the library's utilities with `!important`.
- Typography scale (font sizes, line-heights, weights) is added to `@theme` the same way as
  any other token (`--font-size-{{PREFIX}}-*`) and consumed via Tailwind's text utilities —
  never hand-rolled per component. If a UI library was chosen and ships a typography
  primitive, use that instead — see `ui-library-usage.md`.
- No hardcoded size/color values (px, hex, rgb literals) in templates or the rare
  component stylesheet — every value goes through a Tailwind utility backed by `@theme`,
  never an arbitrary-value bracket (`w-[13px]`, `text-[#333]`) as a substitute for a
  missing token; add the token to `@theme` instead.
- No inline styles (`[style]` binding or `style="..."` attribute) in templates — express
  the value as a utility class instead. If the value is genuinely dynamic and can't be a
  static class (e.g. a computed width), bind a CSS custom property (`[style.--foo]`) and
  consume it from a utility via an arbitrary-value reference (`w-[var(--foo)]`), not a raw
  literal.
- A per-component stylesheet is the exception, not the default — only for what Tailwind
  utilities genuinely can't express (e.g. a `@keyframes` block). When one exists, it still
  participates in Tailwind's own layers; don't declare a competing `@layer` order.

## Review Checklist

- [ ] No hand-rolled `@layer {{PREFIX}}.*` order — Tailwind's own layers are used as-is.
- [ ] New design tokens are added to the `@theme` block, not as ad-hoc `:root` variables.
- [ ] No hardcoded size/color and no arbitrary-value bracket standing in for a missing
      token.
- [ ] No `[style]`/`style="..."` in templates — dynamic values go through a bound CSS
      custom property consumed via an arbitrary-value utility, not an inline style.
- [ ] A per-component stylesheet exists only for what utilities can't express.
- [ ] No hand-rolled `font-size`/`line-height` per component — typography goes through
      `@theme` font tokens/text utilities, or the UI library's typography primitive if one
      was chosen.
