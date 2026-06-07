# figma-design-skills

Two composable [Claude Code](https://code.claude.com) skills for **design-to-code fidelity** — get the real design out of Figma, then prove the running app actually matches it.

Most "is it pixel-perfect?" checks fail the same two ways: the design is read off a **screenshot** (so a 4px gap or a 600→400 weight slips through), and "looks done" is treated as a **check** (no measured feedback loop). These skills fix both — one extracts exact values, the other verifies them by measurement.

```
   ┌─────────────────────────┐         ┌──────────────────────────┐
   │  figma-design-extract    │  spec   │  design-fidelity-verify  │
   │  read Figma → spec table │ ──────► │  measure app → PASS/FAIL │
   └─────────────────────────┘  table  └──────────────────────────┘
     exact tokens, not pixels             computed values, not glances
```

## The two skills

| Skill | One-liner |
|-------|-----------|
| **[figma-design-extract](skills/figma-design-extract)** | Pull exact tokens, sizes, and structure out of a Figma node via the Figma MCP into a build-ready **design-spec table** — `element \| property \| Figma exact value \| repo token \| source component`. Maps each Figma variable to *your* design tokens; logs drift instead of hardcoding. |
| **[design-fidelity-verify](skills/design-fidelity-verify)** | Run a bounded **vision + numeric loop** (~3 iterations) that reads the *actually rendered* values off the live app and walks every spec row to PASS/FAIL + delta. Works for **web** (`getComputedStyle`) and **mobile** (native view props via Argent or similar). |

They compose: `figma-design-extract` produces the spec table; `design-fidelity-verify` consumes it. Use either alone, or together for a full design→code→verified pipeline.

## The workflow

1. **Extract.** Share a Figma node URL. The extract skill reads structured values (`get_variable_defs`, `get_metadata`, `get_design_context`, `get_screenshot`) and produces a spec table mapped to your tokens. The screenshot is a *visual reference only* — never measured.
2. **Build.** Implement the component from the table, reusing existing primitives. Every value is a token reference, not a literal.
3. **Verify.** The verify skill runs the app, captures full-resolution, reads computed/native values, and compares each spec row to the live render — fixing the highest-severity drift first, bounded to ~3 iterations, and reporting any residuals honestly.

## Install

```
/plugin marketplace add jeltehomminga/figma-design-skills
/plugin install figma-design-skills@figma-design-skills
```

Both skills load on demand from their trigger phrases (e.g. a `figma.com` URL, or "verify the design"). You can also clone this repo and drop either `skills/<name>/` folder into your `~/.claude/skills/` (personal) or `.claude/skills/` (project).

## Requirements

- **Claude Code** (or any client that loads Agent Skills).
- **Figma MCP server** (Dev Mode MCP) — for `figma-design-extract`.
- **App-driving tooling** — for `design-fidelity-verify`:
  - **Web:** a browser/Playwright/Chrome MCP that can run JS in the page and screenshot.
  - **Mobile:** [Argent](https://github.com/software-mansion/argent) (recommended), [agent-device](https://github.com/callstackincubator/agent-device), or a lighter iOS-simulator MCP.

## Tool- and stack-agnostic

The skills name specific tools (Tailwind, NativeWind, Restyle, Playwright, Argent) only as **examples**. The principles are what matter:

- A value bound to a Figma variable maps to a **named token** in your code — never a hardcoded literal.
- Fidelity is **measured** (computed styles / native props), never eyeballed from a screenshot.
- Adapt the token syntax and the device tooling to whatever your repo already uses.

## License

[MIT](LICENSE) © Jelte Homminga
