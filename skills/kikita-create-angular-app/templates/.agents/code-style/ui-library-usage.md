<!-- SCAFFOLD: keep this whole file only if a UI library was chosen -->
# UI Library Usage

- Build with {{UI_LIB}}'s primitives first — buttons, inputs, dialogs, layout primitives,
  typography. Reaching for a raw `<button>` or a hand-rolled dialog when {{UI_LIB}} already
  ships one defeats the reason it's in the project.
- Reuse order, most specific wins:
  1. An existing `shared/ui/` component that already wraps a {{UI_LIB}} primitive for this
     project's needs (see `../shared/README.md`) — reuse it instead of wrapping the same
     primitive again.
  2. {{UI_LIB}}'s own primitive, used directly.
  3. A hand-built component — only when neither of the above covers the case.
- Styling a {{UI_LIB}} primitive is layout only — spacing, sizing, positioning, via this
  project's tokens/layers (see `css-architecture.md`) — never its visual identity (color,
  border, shadow, typography). Re-skinning a primitive's look defeats the point of using a
  shared library. Override the library's own visual styling only when the user explicitly
  asks for that.
- Typography: if {{UI_LIB}} ships a typography primitive ({{UI_LIB_TYPOGRAPHY_COMPONENT}} —
  e.g. kikita-ui's `kuiText`), use it for all text content instead of a bare tag with
  hand-rolled `font-size`/`line-height`. Give it the correct semantic tag/role for the
  content it wraps — a heading is a heading, a paragraph is a paragraph — never default to
  `div`/`span` for text just because it's the path of least resistance. See
  `../accessibility.md`.
- If no UI library was chosen, this project's own typography tokens
  (`styles/typography.{{CSS_EXT}}`, see `css-architecture.md`) are the equivalent — use
  those tokens/classes on real semantic tags, not one-off font styles per component.

## Review Checklist

- [ ] New UI reaches for an existing `shared/ui/` wrapper, then {{UI_LIB}}'s own primitive,
      before a hand-built component.
- [ ] Styles applied to a {{UI_LIB}} primitive are layout-only, not a re-skin of the
      library's visual identity — unless the user explicitly asked for that.
- [ ] Text content uses the library's typography primitive (or this project's typography
      tokens if no library was chosen) with the correct semantic tag — not `div`/`span`/`p`
      picked arbitrarily.
