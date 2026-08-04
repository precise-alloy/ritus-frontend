<p align="center">
  <img src="https://github.com/precise-alloy/.github/raw/main/images/ritus-frontend-banner.svg" alt="Ritus Banner" width="2142" style="max-width: 100%; height: auto;" height="500">
</p>

<div align="center">

# Ritus Frontend

</div>

Browser visual-verification plugin. It bundles **Playwright MCP** (cross-platform: Claude Code + GitHub Copilot CLI)
and ships two skills — `e2e-plan` (writes an e2e/visual spec from the outcomes you want checked) and `visual-verify`
(asserts that spec against the running app) — for **objective, re-runnable** browser verification, with no CLI test
command required.

---

## Installation

Both platforms read the plugin manifest's `"mcpServers": ".mcp.json"` and auto-activate the server on install (MCP
precedence: plugins load last-wins, deduped by server name). One `.mcp.json` serves both.

### Standalone

Install just this plugin — no `ritus` core required.

**Claude Code:**

1. Add marketplace
2. Install Ritus

```text
/plugin marketplace add precise-alloy/ritus-frontend
/plugin install ritus-frontend
```

**GitHub Copilot CLI:**

1. Add the marketplace
2. Fetch the plugin manifest
3. Install Ritus

```text
/plugin marketplace add precise-alloy/ritus-frontend
/plugin marketplace browse ritus-frontend-marketplace
/plugin install ritus-frontend@ritus-frontend-marketplace
```

### Alongside ritus

Install `ritus` as well — same marketplace.

Claude Code:

```text
/plugin marketplace add precise-alloy-marketplace
/plugin install ritus
/plugin install ritus-frontend
```

GitHub Copilot CLI:

```text
/plugin marketplace add precise-alloy/ritus
/plugin install ritus@precise-alloy-marketplace
/plugin install ritus-frontend@precise-alloy-marketplace
```

## Two ways to run

Everything the plugin needs — the two skills, the spec format, and the pinned Playwright MCP server — ships inside it.
That lets it run entirely on its own, and also slot into a larger workflow when one is present. The skills are
identical in both modes; only where their inputs come from, and who triggers them, differs.

|  | **Standalone** | **Alongside [ritus](https://github.com/precise-alloy/ritus)** |
| --- | --- | --- |
| Install | Just this plugin | This plugin + `ritus` core |
| Who triggers it | You invoke `e2e-plan` / `visual-verify` directly | ritus dispatches them per ticket |
| Inputs | You give the route(s), base URL, outcomes, and breakpoints | Derived from ritus's review doc / task files / `PROJECT_CONTEXT` |
| Spec path | Wherever you point `e2e-plan` (default `./e2e-spec.md`) | `docs/tasks/{branch-slug}/{ticket-id}-e2e-spec.md` |
| Result | You read the PASS / FAIL / BLOCKED report | The ticket gate runs `visual-verify` before `pr-review` |

The skills carry no ritus knowledge of their own; ritus supplies the wiring (dispatch, inputs, gate) only when it is
installed.

## What it adds

- A version-pinned `playwright` MCP server (`.mcp.json`), auto-activated on install via the manifest `mcpServers`
  field — no manual `/mcp add`.
- The `e2e-plan` skill — writes an `e2e-spec.md` from the routes and the outcomes to verify. Standalone: invoke it with
  the routes + outcomes + breakpoints. Alongside ritus: dispatched after `task-generation`, it derives the spec from
  the approved review document for the human to review.
- The `visual-verify` skill — drives the app and asserts the spec. Standalone: invoke it with a spec + base URL.
  Alongside ritus: dispatched once per ticket before `pr-review`; browser tools live in this skill only.

## Usage (standalone)

Plan, then verify:

1. **Plan** — invoke `e2e-plan` with the route(s), the outcomes to check, and the breakpoints. It writes an
   `e2e-spec.md` (one `## <route>` section per surface) for you to review.

   > Plan an e2e spec for `/account/orders`: a signed-in customer with ≥1 order sees their order list; check at 640px
   > and 1024px. Write it to `./e2e-spec.md`.

2. **Verify** — start the app, then invoke `visual-verify` with the spec and the base URL. It drives the running app
   and reports PASS, a specific failing assertion, or BLOCKED.

   > Run visual-verify on `./e2e-spec.md` against `http://localhost:3000`.

Alongside ritus, both steps are dispatched automatically — you don't invoke them by hand.

## Version pinning

`.mcp.json` pins `@playwright/mcp` to a specific version — never `@latest`. Pinning keeps the tool surface stable:
`browser_run_code_unsafe`/`browser_evaluate` can move in/out of the default caps across releases, and a silent
change would weaken the `visual-verify` allow-list. Bump the pin deliberately.

## Hard requirement (no fallback)

Alongside ritus, a ticket with an `e2e-spec.md` REQUIRES this plugin plus a reachable target: the UI gate is an
objective `visual-verify` check with no human/deferred fallback. Without the plugin (or a reachable target, or
configured breakpoints), the gate cannot be satisfied and `visual-verify` reports BLOCKED. Non-UI ritus work is
unaffected.

## Breakpoints are project-derived

Alongside ritus, viewports come from `docs/PROJECT_CONTEXT.md` § Responsive breakpoints (detected by `repo-scan`);
standalone, from the spec's `viewport(s)` or you. `visual-verify` resizes to each configured breakpoint; with none it
reports BLOCKED — it never assumes `375px`/`1280px`.
