# Worked example — verifying the "Pricing card"

Continues the [`figma-design-extract` worked example](../../figma-design-extract/examples/worked-example.md). We built the Pricing card from its spec table; now we prove the running app matches — once on **web**, once on **mobile**.

The spec table (abridged):

```
element        | property | spec value          | repo token
---------------+----------+---------------------+----------------
Card container | radius   | 12                  | rounded-xl
Card container | padding  | 16                  | p-4
Plan name      | size/wt  | 18 / 600            | text-lg font-semibold
CTA            | bg       | #00B140 (primary/500)| bg-primary-500
CTA            | radius   | 8                   | rounded-lg
CTA            | height   | 48                  | h-12
Feature list   | gap      | 8                   | gap-2
```

---

## Web run (browser/Playwright MCP)

**B0 preflight:** dev server up, card visible on `/pricing`, devtools/JS-eval available. ✅
**B1 flow:** `navigate /pricing` → `waitFor [data-testid=pricing-card]`. Recorded.
**B2 capture:** screenshot at `deviceScaleFactor: 2`.
**B5 numeric pass** — ran `getComputedStyle` on each element:

```
element        | property | spec    | app (computed)      | delta | severity
---------------+----------+---------+---------------------+-------+---------
Card container | radius   | 12px    | 12px                | 0     | PASS
Card container | padding  | 16px    | 16px                | 0     | PASS
Plan name      | size     | 18px    | 18px                | 0     | PASS
Plan name      | weight   | 600     | 400                 | -200  | FAIL high
CTA            | bg       | #00B140 | rgb(0,177,64)=#00B140| 0    | PASS
CTA            | radius   | 8px     | 6px                 | -2    | FAIL med
CTA            | height   | 48px    | 48px                | 0     | PASS
Feature list   | gap      | 8px     | 8px                 | 0     | PASS
```

**B7 fix (iteration 1):** plan name was `font-medium` → change to `font-semibold`; CTA used `rounded-md` → `rounded-lg`. Replay flow, re-capture, re-measure → both PASS. 2 fixes, 1 iteration. Done.

---

## Mobile run (Argent, iOS simulator)

**B0 preflight:** simulator force-booted through Argent, app authenticated, native-devtools status checked → **injection not connecting** on this dev-client. Plan the RN/CDP + tree fallback.
**B1 flow:** launch app → tap "Plans" tab → wait for card. Recorded as an Argent flow.
**B2 capture:** `screenshot scale: 1.0`.
**B5 numeric pass** — native paint read blocked, so degrade gracefully:

- **Accessibility/view tree** gives frames → sizing/placement/gap measurable:

```
element        | property | spec   | app (frame) | delta | severity
---------------+----------+--------+-------------+-------+---------
Card container | width    | 343    | 343         | 0     | PASS
CTA            | height   | 48     | 44          | -4    | FAIL high
Feature list   | gap      | 8      | 8           | 0     | PASS
```

- **CDP inspect-element** confirms the CTA maps to `components/PricingCard.tsx:42`.
- **Residual (recorded, not hidden):** CTA `bg` hex, card `cornerRadius`, plan-name `font-weight` — *not measured numerically*, native injection unavailable. Checked qualitatively on the sharp `scale: 1.0` capture + source audit (`bg-primary-500`, `rounded-xl`, `font-semibold` are present in source).

**B7 fix (iteration 1):** CTA height was `h-11` (44) → `h-12` (48). Replay flow, re-measure → PASS.

**Report:** placement/sizing/gap PASS after 1 fix; 3 paint properties carried as an explicit residual with the reason. No false "pixel-perfect" claim.

---

## Takeaway

Same loop both platforms; only the capture + read mechanics differed (web `getComputedStyle`, mobile native frames + residual). The numeric pass caught a `-200` font-weight and a `-2px`/`-4px` radius/height drift that a vision-only check would have waved through.
