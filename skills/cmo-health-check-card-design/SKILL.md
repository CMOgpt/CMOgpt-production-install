---
name: cmo-health-check-card-design
description: >
  Dedicated visual card design for cmo-health-check specifically. This is
  not the generic finding card defined in `cmogpt:output-conventions` — it
  has its own shape because health check is the founder's primary weekly
  entry point and needs to support a quick, high-visual-impact decision,
  not a single finding. Read this instead of the generic card template
  whenever presenting a health check result.
---

# CMOgpt — Health Check Card Design

## Why this is separate from the standard card

The standard `output-conventions` card is built for one finding: a
headline, a mechanism, 2-4 metrics, one action. Health check carries more
than that by design — six to seven weekly indicators, a priority issue,
a prescription, and a verdict — because it's meant to answer "is my
business okay this week?" in one glance, not drill into one thing. Forcing
it into the generic shape would either cut real information or blow past
the 4-metric cap. So it gets its own layout, following the same design
system and field-quality rules as the generic card (mandatory action, no
vague prescriptions, mobile-first), just arranged differently.

## Content model

Every health check card has exactly these parts, in this order:

1. **Store + week context** — store name, week-ending date, data freshness note if stale.
2. **Verdict** — exactly one of: growing profitably / growing with risk / needs attention. Rendered as a colored badge, sentence case.
3. **Headline metric** — total sales, as a large number with its week-over-week delta. This is the one number a founder should absorb in half a second.
4. **Supporting metric grid** — the remaining six indicators (AOV, discount %, marketing spend, MER, contribution margin, repeat ratio), each with value and delta.
5. **Priority issue** — the single metric most likely to compound if unaddressed, called out separately with real numbers. Not folded into the grid — it needs its own visual weight.
6. **Prescription ("Do this")** — one to two concrete actions with a target number. Never generic.
7. **Go deeper** — 2-3 follow-up prompts.

Verdict color mapping:
- Growing profitably → `success` role (green)
- Growing with risk → `warning` role (amber)
- Needs attention → `danger` role (red)

## Primary output: plain text

This is the default on every surface today (see `RENDERING_MODE` in
`output-conventions`). Same content model as above, laid out for a chat
message rather than a card.

```
{STORE_NAME} — week ending {DATE}
{DATA_FRESHNESS_NOTE, only if stale}

Verdict: {GROWING PROFITABLY | GROWING WITH RISK | NEEDS ATTENTION}

Total sales: {VALUE} ({DELTA} vs last week)

{METRIC_LABEL}: {VALUE} ({DELTA})
{METRIC_LABEL}: {VALUE} ({DELTA})
{METRIC_LABEL}: {VALUE} ({DELTA})
{METRIC_LABEL}: {VALUE} ({DELTA})
{METRIC_LABEL}: {VALUE} ({DELTA})
{METRIC_LABEL}: {VALUE} ({DELTA})

Priority issue: {ONE_OR_TWO_SENTENCES_WITH_REAL_NUMBERS}

Do this: {ONE_TO_TWO_CONCRETE_ACTIONS_WITH_TARGET_NUMBERS}

Go deeper: {FOLLOWUP_1} · {FOLLOWUP_2} · {FOLLOWUP_3}
```

**Worked example (Maku The Label, week ending Aug 8, 2026):**

```
MAKU The Label — week ending Aug 8, 2026
Data current to: Aug 8, 2026

Verdict: NEEDS ATTENTION

Total sales: $42,260 (-16.0% vs last week)

AOV: $154.32 (-12.9%)
Discount %: 14.2% (-8.7pts)
Marketing spend: $7,744 (-11.8%)
MER: 5.46x (-4.8%)
Contribution margin: 40.6% (+15.2pts)
Repeat ratio: 21.0% (-7.9pts)

Priority issue: Traffic is falling on a fast, accelerating trend — the
top-ranked driver behind this week's sales drop.

Do this: Restore about $500 of the $1,032 weekly budget cut to test
traffic recovery, while holding discounting near 14.2%.

Go deeper: How is repeat ratio trending over 30 days? · What's driving
the traffic decline specifically? · Show my LTV by segment
```

## Secondary output: HTML card (reserved — see rendering mode in output-conventions)

Only emit this when `output-conventions`' `RENDERING_MODE` is
`HTML_CARD_CONFIRMED` for the current surface.

```html
<div style="background: var(--surface-2); border-radius: 12px; border: 0.5px solid var(--border); padding: 1rem 1.25rem;">

  <div style="display: flex; align-items: center; justify-content: space-between; margin-bottom: 12px;">
    <div>
      <p style="font-weight: 500; font-size: 15px; margin: 0;">{STORE_NAME}</p>
      <p style="font-size: 12px; color: var(--text-muted); margin: 2px 0 0;">Week ending {DATE} &middot; data current to {DATA_DATE}</p>
    </div>
    <div style="display: flex; align-items: center; gap: 6px; background: var(--bg-{VERDICT_ROLE}); color: var(--text-{VERDICT_ROLE}); font-size: 12px; padding: 5px 10px; border-radius: var(--radius); white-space: nowrap;">
      <i class="ti ti-{VERDICT_ICON}" style="font-size: 15px;" aria-hidden="true"></i>
      {VERDICT_LABEL_SENTENCE_CASE}
    </div>
  </div>

  <div style="border-top: 0.5px solid var(--border); padding-top: 12px; margin-bottom: 12px;">
    <p style="font-size: 13px; color: var(--text-secondary); margin: 0 0 2px;">Total sales</p>
    <div style="display: flex; align-items: baseline; gap: 8px;">
      <span style="font-size: 24px; font-weight: 500;">{SALES_VALUE}</span>
      <span style="font-size: 13px; color: var(--text-{SALES_DELTA_ROLE});">{SALES_DELTA} vs last week</span>
    </div>
  </div>

  <div style="display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 8px; margin-bottom: 12px;">
    <!-- repeat per supporting metric, exactly 6 -->
    <div style="background: var(--surface-1); border-radius: var(--radius); padding: 10px 12px;">
      <p style="font-size: 12px; color: var(--text-secondary); margin: 0 0 2px;">{METRIC_LABEL}</p>
      <p style="font-size: 16px; font-weight: 500; margin: 0;">{METRIC_VALUE}</p>
      <p style="font-size: 12px; color: var(--text-{METRIC_DELTA_ROLE}); margin: 2px 0 0;">{METRIC_DELTA}</p>
    </div>
  </div>

  <div style="background: var(--bg-warning); border-radius: var(--radius); padding: 10px 12px; display: flex; gap: 8px; align-items: flex-start; margin-bottom: 8px;">
    <i class="ti ti-flag" style="color: var(--text-warning); font-size: 16px; margin-top: 1px;" aria-hidden="true"></i>
    <div style="font-size: 13px; color: var(--text-warning);"><span style="font-weight: 500;">Priority issue:</span> {PRIORITY_ISSUE_TEXT}</div>
  </div>

  <div style="background: var(--bg-accent); border-radius: var(--radius); padding: 10px 12px; display: flex; gap: 8px; align-items: flex-start;">
    <i class="ti ti-bulb" style="color: var(--text-accent); font-size: 16px; margin-top: 1px;" aria-hidden="true"></i>
    <div style="font-size: 13px;"><span style="font-weight: 500;">Do this:</span> {PRESCRIPTION_TEXT}</div>
  </div>

  <div style="margin-top: 10px; display: flex; flex-wrap: wrap; gap: 6px;">
    <!-- repeat per follow-up, 2-3 total -->
    <button onclick="sendPrompt('{FOLLOWUP_PROMPT}')" style="font-size: 12px;">{FOLLOWUP_LABEL} &#8599;</button>
  </div>

</div>
```

**Verdict → role/icon mapping for the template above:**

| Verdict | `{VERDICT_ROLE}` | `{VERDICT_ICON}` |
|---|---|---|
| Growing profitably | `success` | `trending-up` |
| Growing with risk | `warning` | `alert-triangle` |
| Needs attention | `danger` | `alert-triangle` |

**Metric delta role:** `success` if the change is favorable for that
specific metric (e.g. discount % falling, contribution margin rising),
`danger` if unfavorable, `text-secondary` (no role color) if roughly flat.
Favorable direction depends on the metric — falling discount % is good,
falling sales is bad — so this is a per-metric judgment, not a fixed
up-is-green rule.

**Layout notes:**
- Supporting metric grid is capped at 2 columns even though there are 6
  items (3 rows of 2) — matches the mobile column cap; do not go to 3
  columns.
- The verdict badge sits top-right, opposite the store name, so it's the
  first color a founder's eye lands on — this is the "quick decision"
  signal the redesign was for.
- Priority issue and prescription are visually distinct blocks (different
  role colors) so they don't blur into one long paragraph — a founder
  should be able to tell "what's wrong" from "what to do" without reading
  closely.

## What did not change

The underlying business logic in `cmo-health-check` — which metric leads,
the six-indicator system read, the verdict thresholds, the priority-issue
selection rules — is unchanged by this design. This file only defines how
that output is packaged and rendered; the diagnosis logic still lives in
`cmo-health-check`'s own procedure.
