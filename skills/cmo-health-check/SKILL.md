---
name: cmo-health-check
description: >
  Weekly business health review for Shopify brands. Auto-triggers when the
  founder asks "how is my store doing", "run my weekly review", "health check",
  "how are my numbers this week", "is my business growing", or any weekly
  performance question. Calls get_diagnosis starting from SALES-AMT (7 days,
  3 levels deep) and cross-checks the six critical health indicators via
  get_metric_history. Produces a single health verdict: GROWING PROFITABLY,
  GROWING WITH RISK, or NEEDS ATTENTION.
---

# CMOgpt — Weekly Health Check

## Purpose

The weekly health check is a fixed ritual, not a flexible analysis. It answers
one question: **is the business healthier this week than last week?**

It starts from total revenue, lets the diagnosis engine surface what is driving
it, and cross-checks the six metrics that together determine whether growth is
real and profitable. The output is a verdict, a priority issue, and one action —
not a report.

---

## Triggers

Use this skill when the founder asks:
- "How is my store doing this week?"
- "Run my weekly review" / "weekly health check"
- "How are my numbers?" / "How's the business?"
- "What should I focus on this week?"
- "Is my business growing profitably?"
- Any question about overall weekly store performance or health

---

## Procedure

### Step 1 — Get session context

Call `about_my_account`. Extract and hold:
- `plan` — determines which metrics and domains are available

Call `about_my_store`. Extract:
- `store_name` — for the review header
- `shopify_last_order_date` — data freshness check
- `growth_stage` — informs how to weight new vs repeat findings

**Data freshness check:** If `shopify_last_order_date` is more than 2 days before
today, tell the user: "Data is current to [shopify_last_order_date] — the most
recent trading day may not yet be included." Then proceed; do not abort.

If either tool returns an error or the connector is not installed, tell the user:
"To use CMOgpt, connect your store at https://cjmap.cmogpt.io/signup." Then stop.

---

### Step 2 — Run the sales diagnosis

Call `get_diagnosis('SALES-AMT', 7, 3)`.

- `'SALES-AMT'` — start from total revenue, the natural weekly entry point
- `7` — look back 7 days (current trading week)
- `3` — drill 3 levels deep into the revenue decision tree

From the response, read:
- `ranked_candidates` — pre-scored metrics explaining the SALES-AMT result this
  week. Use as a starting point, not the final answer. Apply the re-weighting
  rules in Step 4 before choosing the lead finding.
- `available_metrics.domains` — check coverage. If `repeat` or `mer` domains show
  `loaded: 0` but `available > 0`, you may have an undiagnosed system problem.
  Note this for the user if the diagnosis warrants going deeper.
- `notes` — any `low_confidence` flags. Do not lead with a low-confidence metric.

---

### Step 3 — Pull the six critical health indicators

Call `get_metric_history` for each of these metrics, lookback 14 days (gives you
this week and last week for comparison):

```
get_metric_history('AOV', 14)
get_metric_history('BUDGET-DAILY', 14)
get_metric_history('CONTRIBUTION-MARGIN', 14)
get_metric_history('MER', 14)
get_metric_history('ORDER-DISC-PCT', 14)
get_metric_history('REPEAT-RATIO', 14)
```

These six give you the context the diagnosis engine may not surface if its top
candidates sit elsewhere in the tree. You are explicitly checking the metrics
that determine whether growth is sustainable — independent of what the scoring
engine ranked first.

---

### Step 4 — Read the system, not the individual metrics

Before presenting any finding, read all six history metrics as a system. The
combination tells the real story.

**Healthy growth — all of these hold:**
- Sales flat or up, contribution margin stable or improving
- ORDER-DISC-PCT flat or falling
- BUDGET-DAILY stable relative to revenue
- MER stable or improving, REPEAT-RATIO stable or improving
→ Growth quality is good. Say so. Identify what is working and where to invest next.

**Warning: buying revenue**
- Sales up AND ORDER-DISC-PCT up AND BUDGET-DAILY up AND CONTRIBUTION-MARGIN down
→ Revenue growth is being purchased with discounts and spend. The economics are
weakening. This is the most common trap for Shopify founders.

**Warning: demand weakness**
- Sales down AND MER falling (spend held, orders fell) AND CONTRIBUTION-MARGIN under pressure
→ Demand is softening. Problem is conversion, offer, or traffic quality — not ad spend.
Do not recommend increasing budget.

**Warning: retention risk**
- Sales flat or up AND CONTRIBUTION-MARGIN acceptable AND REPEAT-RATIO declining
→ Acquisition may be working but the customer base is not sticking. Loyalty and
post-purchase problem, not a marketing problem.

---

### Step 5 — Apply CMOgpt priority rules to the diagnosis

When choosing the lead finding from `ranked_candidates`:

1. **Contribution margin is centre of gravity.** If `CONTRIBUTION-MARGIN` or any
   profitability node has `status: under_pressure`, it is almost always the lead
   finding — even if another metric has a marginally higher `priority_score`.

2. **Trend can outrank position.** A metric near benchmark but with fast-negative
   `trend_adj` matters more than one far from benchmark but recovering.

3. **Localise to new vs repeat.** Use `segment_split` from the payload. A problem
   in new business has a different prescription than the same problem in repeat
   business. Always say which.

4. **Respect `score_basis`.** A `trend_only` candidate (no benchmark) should
   corroborate a gap-scored finding, not lead on its own.

5. **Discard `low_confidence` metrics as lead findings.** Use them as supporting
   evidence only.

---

### Step 6 — Present the review

**Header:**
```
WEEKLY HEALTH CHECK — [store_name] — week ending [date]
Data current to: [shopify_last_order_date]
```

**Six-metric summary** — current week vs prior week, direction of change:
```
Metric                     This week    Last week    Change
───────────────────────────────────────────────────────────
Total Sales                $XX,XXX      $XX,XXX      ↑ +X%
AOV                        $XX.XX       $XX.XX       ↓ -X%
Discount %                 X.X%         X.X%         ↑ +X%
Marketing spend (daily)    $XX.XX       $XX.XX       → flat
MER                        X.Xx         X.Xx         ↓ -X%
Contribution Margin        XX%          XX%          ↓ -X%
Repeat Ratio               XX%          XX%          ↑ +X%
```

**Priority issue** — the single metric most likely to compound if unaddressed.
State it plainly with the actual numbers. Do not soften.

Example:
> "Your priority issue this week is discount %, which has risen from 12% to 19%.
> At your current contribution margin of 28%, this is the single fastest lever
> compressing profitability."

**Prescription** — one to two specific, concrete actions. Use actual numbers.
No generic advice.

**Health verdict** — close with exactly one of:

> **GROWING PROFITABLY** — Revenue moving in the right direction, margin holding
> or improving, repeat ratio stable. Good week. Scale what is working.

> **GROWING WITH RISK** — Revenue up but economics weakening. [Name the specific
> compression point.] Address this before it compounds.

> **NEEDS ATTENTION** — [Metric] is out of range in a way that will compound if
> not addressed this week. [Name the priority action.]

---

## Diagnosis rules by metric (reference)

**Sales down** — do not default to "spend more." Check: conversion rate, AOV,
traffic volume, stock availability, seasonality — in that order.

**Contribution margin falling** — check: ORDER-DISC-PCT rising, BUDGET-DAILY
rising relative to orders, COGS shift, fulfilment cost creep.

**ORDER-DISC-PCT rising** — check: is there a campaign running (expected)? Is it
becoming structural (discounting every week = new price floor)?

**MER falling** — check: order volume down while spend held? Conversion falling?
Creative fatiguing? Traffic quality shift (country, bot traffic)?

**REPEAT-RATIO falling** — check: post-purchase experience, replenishment email
timing, product satisfaction signals (returns, reviews).

**AOV falling** — check: mix shift to cheaper items, increased discounting,
bundles/upsells not working, free shipping threshold too low, high-value items
out of stock.

---

## Tone and constraints

- Write like a commercially experienced advisor, not a BI dashboard
- Be direct — do not soften findings to the point they lose meaning
- Use the actual numbers from the data; never generalise
- The review should be readable in under 3 minutes
- One priority issue, one action — not a list of ten
- Never recommend increasing ad spend as the first response to falling sales
- Never give the same prescription two weeks running without checking if the
  prior action was taken

---

## What this skill is not

This is a fixed weekly ritual with a fixed structure. It is not a flexible
analytical tool. If the founder wants to go deeper on a specific metric, domain,
or strategic question, use `/cmo-diagnose-metrics` for a targeted review.
