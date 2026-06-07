# Worked example — extracting a "Pricing card" from Figma

A concrete run of `figma-design-extract` against a fictional **Pricing card** node. The repo here uses Tailwind/NativeWind class names; swap in your own token names.

## Input

The designer shares:

```
https://www.figma.com/design/AbC123/Marketing?node-id=210-540
```

So the node id is `210:540`.

## Step 0 — resolve the node

`get_design_context("210:540")` returns the frame (no "node not found"), so the id is fresh. Scope every later call to `210:540`.

## Step 1 — structure (`get_design_context`, `disableCodeConnect: true`)

Tells us the card is a vertical auto-layout:

```
Card (auto-layout, vertical, gap 8, padding 16)
├─ Badge        ("Most popular")
├─ Plan name    (text)
├─ Price row    (auto-layout, horizontal, baseline)
│  ├─ Amount    (text)
│  └─ Period    (text, "/mo")
├─ Feature list (auto-layout, vertical, gap 8)
└─ CTA          (button, full width)
```

Structure only — we do **not** copy any pixel positions from this output.

## Step 2 — exact styles (`get_variable_defs`) — the source of truth

```
Card.fill            = var(--surface/0)
Card.cornerRadius    = 12
Card.padding         = 16
Card.gap             = 8
PlanName.font        = Heading/M  (18 / 600)
Amount.font          = Display/S  (32 / 700)
Period.fill          = var(--text/muted)
CTA.fill             = var(--primary/500)
CTA.cornerRadius     = 8
CTA.text             = Label/L (16 / 600), var(--text/on-primary)
```

The badge background came back with **no bound variable** → design smell. Note it, read the raw value with `get_metadata`, flag for the designer; don't bake the literal into a token.

## Step 3 — leftover measurements (`get_metadata`, scoped)

```
Badge.height = 24   (no token — one-off; flagged above)
CTA.height   = 48
```

## Step 4 — visual reference (`get_screenshot`)

Confirms the badge sits top-left, the price row aligns on the baseline, and the CTA is solid (not outline). Used to verify *what* we build — never measured.

## Step 6 — component reuse audit

Grep finds existing `<Card>`, `<Button>`, `<Badge>`, `<Text>` primitives. We reuse all four; no new components. The price row is plain auto-layout, no component needed.

## Output — the design-spec table

```
element          | property      | Figma exact value     | repo token / class        | source component
-----------------+---------------+-----------------------+---------------------------+------------------
Card container   | bg            | var(--surface/0)      | bg-white dark:bg-neutral-900 | <Card>
Card container   | radius        | 12                    | rounded-xl                | <Card>
Card container   | padding       | 16                    | p-4                       | <Card>
Card container   | gap           | 8                     | gap-2                     | <Card> (auto-layout)
Plan name        | font/size/wt  | Heading/M 18 / 600    | text-lg font-semibold     | <Text>
Amount           | font/size/wt  | Display/S 32 / 700    | text-3xl font-bold        | <Text>
Period           | color         | var(--text/muted)     | text-neutral-500          | <Text>
Feature list     | gap           | 8                     | gap-2                     | (auto-layout)
CTA              | bg            | var(--primary/500)    | bg-primary-500            | <Button primary>
CTA              | radius        | 8                     | rounded-lg                | <Button>
CTA              | height        | 48                    | h-12                      | <Button>
CTA label        | color         | var(--text/on-primary)| text-white                | <Button>
Badge            | height        | 24 (UNBOUND — flagged)| h-6 (+ drift note)        | <Badge>
```

Two findings logged alongside:
- **Drift:** if `--primary/500` resolves to `#00B140` in Figma but `#00A038` in the token package, add a row to `docs/design-drift.md` and keep using the `primary-500` token name.
- **Design smell:** Badge background + height have no bound variables — flagged for the designer.

This table is now the input to **`design-fidelity-verify`**.
