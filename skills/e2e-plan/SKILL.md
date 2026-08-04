---
name: e2e-plan
description: Use when UI outcomes need an objective, re-runnable e2e/visual spec (e2e-spec.md) for visual-verify to assert — given the route(s), what each must show or do, and the breakpoints to check.
argument-hint: The route(s), the outcomes to verify for each, the breakpoints (widths), and where to write the spec.
user-invocable: true
---

# e2e-plan

Write an `e2e-spec.md` that `visual-verify` can run. It is a faithful, objective restatement of the outcomes you are
given — never a new requirement. If a required input is missing (routes, per-route outcomes, breakpoints), ask for it; if it is still not
given, stop and report BLOCKED. The output path is optional — default to `./e2e-spec.md`. Never assume breakpoints.

When starting e2e-plan, create this TODO — **every item below, verbatim** — and mark each done as you complete it:

TODO:

```markdown
- [ ] Gate on required inputs (routes, per-route outcomes, breakpoints); report BLOCKED if any is missing.
- [ ] Gather preconditions, the output path, and the UI surfaces to cover.
- [ ] Read `templates/e2e-spec.md` for the format.
- [ ] Write the spec.
- [ ] Get user approval, then hand off to visual-verify.
```

## Format

One `## <route>` section per surface; routes are app-relative (the base URL is supplied at run time). See
`templates/e2e-spec.md` for a complete example. Each section has:

- `name` — a short label for the surface.
- `preconditions` — the auth role + data the outcomes assume, or `none`.
- `viewport(s)` — the widths to check; height defaults to 800px (pin `<w>x<h>` only when an outcome depends on height).
- `reference:` *(optional)* — a committed mockup image, the basis for that section's `paint:` check.
- `flow:` *(optional)* — numbered interaction steps, re-run from a clean load per viewport, with inline `assert`
  checkpoints.
- `assertions:` — the outcomes to check at the end. Write each as the strongest objective form the browser can read:
  visible / text / `value:` / `attribute:` / `style:` (computed) / `console:` / `focused:` (which element holds focus)
  / geometry (size, alignment, containment, centering). Reserve `paint:` for a look-and-feel judgment (focus glow,
  gradient, "matches the mockup") that no value comparison can pin. Scope one to a breakpoint with `@ <width>`.

Derive every assertion and precondition from the stated outcomes. If an outcome does not pin a browser-checkable
result, do not invent one.

## Handoff

- **Report:** the written spec path and the routes it covers, or BLOCKED (required inputs still missing after asking,
  or outcomes too vague to pin an objective assertion).
- **Approval gate:** show the spec to the user, wait for approval, and revise on request before handing off to
  visual-verify.
