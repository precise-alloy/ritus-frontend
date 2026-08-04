---
name: visual-verify
description: Use when a UI change needs objective, re-runnable verification — asserting an e2e/visual spec (e2e-spec.md) against a running app at the configured breakpoints, reporting PASS / FAIL / BLOCKED.
argument-hint: The e2e spec (a file or its routes) and the base URL to verify against.
user-invocable: true
---

# visual-verify

Assert an `e2e-spec.md` against a running app in a fresh context, and report PASS / FAIL / BLOCKED. You are given the
spec (a file or its routes) and the base URL; if either is missing, ask, then report BLOCKED if it is still absent
(never guess the dev-server port).

Targets are **local-only**, so `browser_evaluate` / `browser_run_code_unsafe` are enabled — use them to read computed
style (`getComputedStyle`), DOM/ARIA attributes (`getAttribute`), and the console. Never point this at a remote or
untrusted origin while they are on; `--isolated` + localhost-only origins are the remaining guardrails.

When starting visual-verify, create this TODO — **every item below, verbatim** — and mark each done as you complete it:

TODO:

```markdown
- [ ] Gate on required inputs (spec + base URL); report BLOCKED if either is missing.
- [ ] Read the spec and its breakpoints (each section's `viewport(s)`).
- [ ] Verify every `## <route>` section at each breakpoint.
- [ ] Handoff.
```

## Procedure

For every `## <route>` section, at each `viewport(s)` width (height 800px unless the section pins `<w>x<h>`):

1. `browser_resize` to the width, then `browser_navigate` to base URL + route — resize *before* navigating so
   responsive rendering and interaction paths load correctly for this breakpoint.
2. If the section's `preconditions` are not met (login redirect, missing seed data), report **BLOCKED** for that
   route — not FAIL; the environment, not the code, is unverifiable.
3. Run the `flow` steps in order (locate elements via `browser_snapshot`), checking each inline `assert` where it
   appears.
4. Check every `assertion` (and any `@ <width>` matching this breakpoint) with the strongest signal: the
   `browser_verify_*` tools, `browser_snapshot({boxes:true})` for geometry (bounding boxes within ±1px), and
   `browser_evaluate` / `browser_console_messages` for `style:` / `attribute:` / `focused:` (`document.activeElement`)
   / `console:`. Judge `paint:` by vision against the `reference:` image (or the requirement when there is none) —
   vision decides `paint:` only, and never overrides an objective result.
5. Save evidence: `browser_take_screenshot` to `.ritus/screenshot/<ticket>/<name>-<breakpoint>.png` (`mkdir -p` the
   dir first; `<ticket>` = the spec filename's `{ticket-id}` prefix, else its parent folder).

## Verdicts

- **PASS** — an objective signal confirms it (or, for `paint:`, vision matches the basis).
- **FAIL** — a signal contradicts it: the code is wrong.
- **BLOCKED** — the environment cannot run it: no base URL, unmet preconditions, an unreachable target, or no
  breakpoints. Never guess a port or assume `375`/`1280`.

## Handoff

- **Report:** your verdict - `VERIFY: PASS` or `VERIFY: FAIL` with gaps, or `VERIFY: BLOCKED` with the reason.
