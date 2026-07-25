---
name: cmo-build-profit
description: >
  Growth quality assessment for Shopify brands. Use when the founder asks
  "should I spend more on ads", "is my growth healthy", "is my growth
  profitable", "can I scale my marketing", "is my growth sustainable", or
  any question about whether current growth justifies increasing marketing
  spend. Evaluates the source of growth, MER trap patterns, and payback
  cycle before answering. Does not run weekly health or MER root-cause
  diagnosis — routes those to cmo-health-check and cmo-diagnose-metrics.
---

# CMOgpt — Build Profit (Growth Quality)

## Purpose

This skill answers the growth quality question: is this business growing
in a way that compounds value, or is it buying revenue at the expense of
margin and retention? It is the required check before recommending any
increase in marketing spend.

---

## Triggers

Use this skill when the founder asks:
- "Should I spend more on ads?" / "Can I scale my marketing?"
- "Is my growth healthy?" / "Is my growth profitable?"
- "Is my growth sustainable?"
- "Am I growing the right way?"
- Any question about whether current growth justifies increasing spend

**Do not use for:**
- MER root-cause diagnosis ("why is my MER falling") → `cmo-diagnose-metrics` with metrics_code='MER'
- Weekly snapshot ("how's my store this week?") → `cmo-health-check`
- Setting a specific budget number → `cmo-set-marketing-budget`
- Weekly spend adjustment decision → `cmo-optimize-marketing-budget`

---

## Procedure

### Step 1 — Get session context

Call `about_my_account`. Extract `job_id` and `plan`.
Call `about_my_store`. Extract `store_name`, `growth_stage`, and `shopify_last_order_date`.

**Data freshness:** If `shopify_last_order_date` is more than 2 days old,
note it. Proceed; do not abort.

---

### Step 2 — Pull growth quality metrics

Call these in parallel:

```
get_diagnosis(job_id, 'CONTRIBUTION-MARGIN', 30, 3)
get_metric_history(job_id, 'REPEAT-RATIO', 30)
get_metric_history(job_id, 'MER', 30)
get_metric_history(job_id, 'CONTRIBUTION-MARGIN', 30)
get_metric_history(job_id, 'ORDER-DISC-PCT', 30)
get_benchmarks(job_id)
```

If the plan includes LTV metrics, also call:

```
get_metric_history(job_id, 'CONTRIBUTION-CAC-RATIO', 30)
get_metric_history(job_id, 'PAYBACK-CYCLE', 30)
get_metric_history(job_id, 'P2-DAYS-GAP', 30)
```

---

### Step 3 — Identify the source of growth

Determine which of the four sources is dominant (see **Growth Is Not One
Thing** below). This determines whether growth is an asset or a liability
before anything about spend is said.

---

### Step 4 — Check for MER trap patterns

Even if MER looks healthy, check both trap patterns (see **The MER Trap**
below). Always read MER and CONTRIBUTION-MARGIN together, and MER and
REPEAT-RATIO together.

---

### Step 5 — Evaluate the payback question

If CONTRIBUTION-CAC-RATIO and PAYBACK-CYCLE are available, apply the payback
check (see **The Payback Question** below). If CONTRIBUTION-CAC-RATIO < 1.0
and PAYBACK-CYCLE > 2, the answer to "should I spend more?" is no —
regardless of MER.

---

### Step 6 — Present the diagnosis

**What is happening** — one or two sentences on the quality of current growth.
Name the dominant growth source and whether it is compounding value.

**Why** — which signal (margin, repeat ratio, payback) explains the verdict.
Use `signed_gap` and trend from `get_diagnosis`. Localise to new vs repeat
where relevant.

**What to do next** — one specific action. If the founder asked "should I
spend more?", answer directly: yes or no, and the single reason. No hedging.

---

## Reasoning Framework

Apply this reasoning across Steps 3–5 above.

---

## Growth Is Not One Thing

Revenue growth can come from four sources with very different commercial implications:

- **More new customers** (SALES-COUNT-NB up) — good if CAC is sustainable and contribution margin on new orders is positive or recoverable within the payback cycle
- **Higher AOV** (AOV up) — almost always good; more revenue per acquisition cost
- **More repeat purchases** (REPEAT-RATIO up) — the highest-quality growth; no incremental acquisition cost
- **Discounting** (ORDER-DISC-PCT up, revenue up) — dangerous; inflating revenue while compressing margin

When a founder reports strong revenue growth, the first question is: which of these is driving it? The answer determines whether the growth is an asset or a liability.

---

## The MER Trap

MER (Marketing Efficiency Ratio) measures total revenue per dollar of marketing spend. It can look healthy while the business is deteriorating. The two most common failure patterns:

**Pattern 1 — MER holds, contribution margin falls.** Marketing is generating revenue efficiently, but COGS or discounting is consuming the margin. The business is growing unprofitably. Check MER-PCT against CONTRIBUTION-MARGIN simultaneously — if they are diverging, margin is being traded for revenue.

**Pattern 2 — MER holds, repeat ratio falls.** The marketing engine is replacing churning customers with new ones. Revenue looks stable but the customer base is weakening. Check REPEAT-RATIO trend alongside MER. If repeat ratio is falling, the business is on a treadmill — spending more to stand still.

---

## The Payback Question

Before recommending increased marketing spend, always check:

- **CONTRIBUTION-CAC-RATIO** — is contribution margin covering acquisition cost? Below 1.0 means each new customer is currently loss-making on first order.
- **PAYBACK-CYCLE** — how many orders are needed to recover CAC? If it is 3+ orders, the business is highly sensitive to churn after the first purchase.
- **P2-DAYS-GAP** — how quickly do customers return for a second purchase? A long gap combined with a high payback cycle means the business is chronically under-recovering CAC.

If CONTRIBUTION-CAC-RATIO < 1.0 and PAYBACK-CYCLE > 2, the answer to "should I spend more on marketing?" is: not until the unit economics of each new customer improve. More spend accelerates cash consumption without improving the business.

---

## New vs Repeat Balance

SALES-NB-RP-RATIO tells you the structure of growth. The right balance depends on the store's growth stage:

- Early-stage stores (< 2 years, low repeat base): higher NB% is expected and acceptable
- Growth-stage stores: repeat should be growing as a share — if it is not, retention is broken
- Mature stores: repeat should be the majority of revenue; heavy NB reliance at this stage means high churn

When repeat ratio is falling in a mature store, do not recommend more acquisition spend. Recommend fixing the retention economics first — the acquisition engine is filling a leaking bucket.

---

## The One Question That Matters

When a founder asks "is my growth healthy?", the answer has two parts:

1. Is contribution margin holding or improving as revenue grows? (Growth that compresses margin is not healthy growth.)
2. Is the repeat ratio holding or improving? (Growth that doesn't compound repeat customers is not sustainable.)

If both are yes: the growth is healthy. If either is no: name the problem before discussing the growth rate.
