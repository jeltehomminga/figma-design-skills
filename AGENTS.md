# AGENTS.md

Guidance for any coding agent working in — or with — this repository.

This repo ships two composable, tool-agnostic skills for **design-to-code fidelity**. The summaries below work as passive guidance in any agent; the full step-by-step instructions live in the skill files and should be opened when a task matches.

## The two skills

- **Extract — `skills/figma-design-extract/SKILL.md`.** Use when implementing UI from a Figma node (a `figma.com` URL, "build this design", "match the Figma"). Read exact values via the Figma MCP (`get_variable_defs` for tokens, `get_metadata` for sizes, `get_design_context` for structure; `get_screenshot` is a visual reference only — never measured). Map every value to a repo design token; never hardcode. Output a design-spec table.
- **Verify — `skills/design-fidelity-verify/SKILL.md`.** Use when checking a built screen ("verify the design", "is this pixel-perfect", "design QA"). Measure the *rendered* values (web: `getComputedStyle`; mobile: native view props via Argent or similar at `scale: 1.0`), walk every spec-table row to PASS/FAIL + delta, fix highest-severity first, bounded to ~3 iterations, and report residuals honestly.

## Core rules (both skills)

- Tokens/variables are the source of truth. **Never measure from a screenshot.**
- Fidelity means the **right token**, not the literal pixel.
- Reuse existing components before scaffolding new ones.
- Prove a match by **measuring** rendered values, not glancing; cap the fix loop at ~3 iterations and report residuals honestly.

## Requirements

- Figma MCP (extract); a browser/Playwright MCP (web verify) or Argent / agent-device (mobile verify).
- MCP servers are configured per tool — see the README section "Use with other agents".

## Full detail

Open the relevant `SKILL.md` above. Cross-tool install steps are in [`README.md`](README.md).
