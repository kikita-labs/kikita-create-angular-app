# Accessibility

- Prefer native HTML semantics before ARIA (`<button>` not `<div role="button">`).
- Use Angular CDK or Angular Aria primitives (focus trap, live announcer, focus origin,
  listbox/combobox/menu behaviors) for complex interactive widgets instead of hand-rolling
  keyboard/focus handling.
- Every interactive element is reachable and operable by keyboard alone.
- Touch targets are at least 44x44px on touch viewports.
- Don't claim a component is "fully accessible" without actually checking it — screen
  reader pass, keyboard-only pass, and color-contrast check (WCAG 2.2 AA, 4.5:1 for text).
- Responsive: components must work from a 320px viewport up through desktop, without
  horizontal scroll on the page body.

## Review Checklist

- [ ] Keyboard-only pass done on new interactive UI.
- [ ] Color contrast meets WCAG 2.2 AA.
- [ ] No horizontal scroll introduced at 320px width.
- [ ] Native semantics used before reaching for ARIA.
