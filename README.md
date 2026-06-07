# figma-design-skills

Two composable, tool-agnostic skills for **design-to-code fidelity** — get the real design out of Figma, then prove the running app actually matches it. Packaged as a [Claude Code](https://code.claude.com) plugin, and usable in **OpenAI Codex, Cursor, GitHub Copilot**, and any agent that reads `AGENTS.md` — see [Use with other agents](#use-with-other-agents).

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

## Use with other agents

> Cursor · OpenAI Codex · GitHub Copilot · Gemini CLI · Aider · Zed · and any agent that reads `AGENTS.md`.

The SKILL.md **bodies are plain, tool-neutral markdown** — the only Claude-specific part is auto-triggering via the `description`. To use a skill elsewhere, wire its body into your tool's instruction format. Every tool below speaks **MCP**, so the Figma / Playwright / Argent references work once you've configured those servers in that tool.

### Any tool — `AGENTS.md` (broadest reach)

This repo ships a root [`AGENTS.md`](AGENTS.md) that summarizes both skills and points to the full bodies. Tools that read `AGENTS.md` — Codex, Cursor, Copilot's coding agent, Gemini CLI, Aider, Jules, Zed, Windsurf, and 20+ others — pick it up automatically. Copy it (or the relevant parts) into your own project's `AGENTS.md`. *(VS Code Copilot: enable the `chat.useAgentsMdFile` setting.)*

### OpenAI Codex — native skills

Codex reads the **same `SKILL.md` format** from `.agents/skills/`. From your project:

```bash
mkdir -p .agents/skills
cp -r path/to/figma-design-skills/skills/* .agents/skills/
```

Invoke with `/skills` or `$figma-design-extract`, or let Codex match on the `description`.

### Cursor — project rule

Cursor uses `.cursor/rules/*.mdc`. Create one per skill — paste the skill body under MDC frontmatter:

```markdown
---
description: <paste the skill's description>
alwaysApply: false
---
<paste the body of skills/figma-design-extract/SKILL.md>
```

Leave `globs` empty and `@`-mention it (`@figma-design-extract`), or set `globs: "**/*.tsx"` to auto-attach on relevant files.

### GitHub Copilot — instructions file

Create `.github/instructions/figma-design-extract.instructions.md`:

```markdown
---
applyTo: "**"
---
<paste the body of the skill>
```

Honored in VS Code and by the Copilot cloud agent. For always-on guidance instead, append the body to `.github/copilot-instructions.md`.

### Configure the MCP servers (per tool)

The skills call MCP servers; each tool configures them separately:

| Tool | MCP config |
|------|-----------|
| Claude Code | `.mcp.json` / `.claude/mcp.json` |
| OpenAI Codex | `~/.codex/config.toml` — `[mcp_servers.<name>]` |
| Cursor | `.cursor/mcp.json` |
| GitHub Copilot | `.github/mcp.json` (Agent mode; root key `servers`) |

## Requirements

- **An agent** — Claude Code, OpenAI Codex, Cursor, GitHub Copilot, or any MCP-capable coding agent (see [Use with other agents](#use-with-other-agents)).
- **Figma MCP server** (Dev Mode MCP) — for `figma-design-extract`.
- **App-driving tooling** — for `design-fidelity-verify`:
  - **Web:** a browser/Playwright/Chrome MCP that can run JS in the page and screenshot.
  - **Mobile:** [Argent](https://github.com/software-mansion/argent) (recommended), [agent-device](https://github.com/callstackincubator/agent-device), or a lighter iOS-simulator MCP.

## Tool- and stack-agnostic

The skills name specific tools (Tailwind, NativeWind, Restyle, Playwright, Argent) only as **examples**. The principles are what matter:

- A value bound to a Figma variable maps to a **named token** in your code — never a hardcoded literal.
- Fidelity is **measured** (computed styles / native props), never eyeballed from a screenshot.
- Adapt the token syntax and the device tooling to whatever your repo already uses.

## Background — why measured fidelity (and where it's debated)

This approach reflects current Figma guidance and practitioner consensus, with the honest caveats:

- **Tokens/variables are the source of truth.** Figma: *"Figma knows which specific token is used, and can provide the name of that variable to the LLM."* ([Figma blog](https://www.figma.com/blog/introducing-figma-mcp-server/)). Reuse via Code Connect is *"the #1 way to get consistent component reuse."*
- **Don't measure from a screenshot.** Vision-language models are measurably weak at fine-grained pixel distances ([spatial-reasoning survey](https://arxiv.org/pdf/2510.25760)), so exact values come from structured data and the running app is *measured*, not eyeballed. The blind spot this closes is real — AI *"has no visibility into the final, rendered output of its own code"* ([Builder.io](https://www.builder.io/blog/figma-mcp-server)).
- **Bounded loop.** Self-correcting agents saturate fast: *"beyond the third attempt, new plans recycle prior tool sequences"* ([study](https://arxiv.org/pdf/2601.11637)) — hence the ~3-iteration cap.
- **"Pixel-perfect" is contested framing.** A credible camp argues the term is meaningless/harmful and the real goal is *"intentional fidelity to design decisions"* ([Smashing Magazine](https://www.smashingmagazine.com/2026/01/rethinking-pixel-perfect-web-design/), [Josh Comeau](https://www.joshwcomeau.com/css/pixel-perfection/)). These skills keep "pixel-perfect" only as a trigger phrase; the pass/fail unit is the **token**, not the literal pixel.
- **Complement, don't replace, visual regression.** Computed-style diffing verifies a *first build* (no baseline exists yet); tools like Chromatic / Percy / Applitools / Playwright `toHaveScreenshot` catch *ongoing* regressions. Use both.

## License

[MIT](LICENSE) © Jelte Homminga
