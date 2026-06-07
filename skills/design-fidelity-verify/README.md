# design-fidelity-verify

> Prove the running app matches its design spec by **measuring** rendered values — not by glancing at a screenshot.

This is the **second** of two composable skills. It consumes the design-spec table from [`figma-design-extract`](../figma-design-extract) and runs a bounded **vision + numeric feedback loop** (~3 iterations) that walks every spec row to PASS/FAIL + delta, for **both web and mobile**.

## What it does

- **Preflight** the infra gates (env up, data shape, inspection tooling, build current, auth) before rebuilding.
- **Capture full-res** (web dpr ≥ 2 / mobile `scale: 1.0`) — blurry capture is the #1 reason drift slips through.
- **Numeric pass** — read the *actually rendered* values and compare to the spec:
  - **Web:** `getComputedStyle` + `getBoundingClientRect` via a browser/Playwright/Chrome MCP → [`references/verify-web.md`](references/verify-web.md)
  - **Mobile:** native view props (backgroundColor, bounds, cornerRadius, font) via **Argent** or similar at `scale: 1.0` → [`references/verify-mobile.md`](references/verify-mobile.md)
- **Record navigation as a replayable flow** so every re-verify measures the same state.
- **Report residuals honestly** when a measurement channel is blocked, instead of claiming a false pass.

The two reference files are loaded **on demand** — a web project never pulls the mobile mechanics and vice versa. The method is **tool-agnostic**; the reference files name specific MCPs only as examples.

## Use it

Say "verify the design", "is this pixel-perfect", "check this against Figma", or run it right after building a screen with `figma-design-extract`. See [`SKILL.md`](SKILL.md) for the loop and [`examples/worked-example.md`](examples/worked-example.md) for a web + mobile run.

> **Complementary, not a replacement for visual regression.** The numeric pass verifies a *first build* (no baseline exists yet); for *ongoing* protection, snapshot the approved render as a baseline for a visual-regression tool (Playwright `toHaveScreenshot`, Chromatic, Percy, Applitools). A cheap structural check — Playwright `toMatchAriaSnapshot` (role/name/order) — pairs well with the element-mapping step.
