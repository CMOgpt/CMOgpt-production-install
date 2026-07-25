---
name: cmo-set-marketing-budget
description: >
  Sets a data-grounded marketing budget for Shopify brands using the
  five-step MER method: target sales → contribution margin → max spend % →
  implied MER → monthly budget. Triggers when the founder asks "what should
  my marketing budget be", "how much should I spend on ads", "set my
  marketing budget", "what MER should I target", "how do I convert MER into
  a budget", or any question about calculating or setting a marketing or
  advertising budget. Uses live store data (contribution margin, current MER,
  growth stage) to produce a specific budget number, not a generic range.
---

# CMOgpt — Set Marketing Budget

## Purpose

The optimal marketing budget is the amount a brand can spend to maximise
growth while protecting contribution margin, cash flow, and target MER.

This skill builds that number using a five-step method:
1. Start with target sales
2. Read contribution margin before marketing
3. Set a target contribution margin after marketing
4. Calculate maximum marketing spend as a % of sales
5. Convert to a monthly budget and an implied MER target

> **Scope note:** In this skill, "marketing" means advertising spend —
> Meta, Google, TikTok, and paid acquisition. CMOgpt distinguishes marketing
> from advertising more broadly (see `/cmo-explain-marketing-advertising`),
> but most founders quote marketing when they mean paid advertising, and this
> skill uses that convention.

---

## Triggers

Use this skill when the founder asks:
- "What should my marketing budget be?"
- "How much should I spend on ads?"
- "Set my marketing budget" / "Help me work out my ad budget"
- "What MER should I target?" / "What is a good MER for my business?"
- "How do I turn a MER target into a budget?"
- "Am I spending too much or too little on marketing?"
- "What percentage of revenue should I spend on ads?"
- Any question about calculating, setting, or validating a marketing budget

---

## Procedure

### Step 1 — Get store and account context

Call `about_my_account`. Extract:
- `job_id` — required for all tool calls
- `plan` — determines metric access

Call `about_my_store`. Extract:
- `store_name` — for the budget output header
- `growth_stage` — determines budget logic and MER range (see Step 4)
- `category` — informs seasonality notes and margin expectations
- `shopify_last_order_date` — data freshness check

If the connector is not installed, you can still run this skill using
founder-supplied inputs. Ask for: target monthly sales, estimated
contribution margin before marketing, and target contribution margin after
marketing.

---

### Step 2 — Pull live metrics

Call these in parallel:

```
get_metric_history(job_id, 'MER', 30)
get_metric_history(job_id, 'CONTRIBUTION-MARGIN', 30)
get_metric_history(job_id, 'SALES-AMT', 30)
get_marketing_budget(job_id)
get_my_targets(job_id)
get_benchmarks(job_id)
```

From the results, extract:
- **Current MER** — 30-day average and recent trend
- **Contribution margin** — before marketing (gross margin minus COGS,
  discounts, freight, fulfilment, payment fees, returns)
- **Current monthly revenue run-rate** — to anchor the target sales number
- **Current daily marketing budget** — from `get_marketing_budget`
- **MER benchmark for their industry** — from `get_benchmarks`
- **Existing MER target** — from `get_my_targets` if set

If `CONTRIBUTION-MARGIN` is not available or returns low_confidence, tell
the founder and ask them to estimate it manually using the cost structure
breakdown in Step 3.

---

### Step 3 — Establish contribution margin before marketing

Before setting a budget, the founder must know how much margin is available
to fund it. Walk through the cost structure if live data is not available:

```
Item                              % of Sales   Example ($250k sales)
────────────────────────────────────────────────────────────────────
Sales                               100%           $250,000
Product / COGS                      -35%           -$87,500
Discounts                           -10%           -$25,000
Freight / fulfilment                -10%           -$25,000
Payment fees                         -2%            -$5,000
Returns allowance                    -5%           -$12,500
Variable customer service            -1%            -$2,500
────────────────────────────────────────────────────────────────────
Contribution margin before mktg      37%            $92,500
```

If live `CONTRIBUTION-MARGIN` data is available, use that number. If the
founder knows their own margin estimate, use that. State clearly which
source you are using.

---

### Step 4 — Set the target MER based on margin

A brand with high margin can afford a lower MER (spend more aggressively).
A brand with thin margin needs a higher MER (spend more conservatively).

Use this table to set the safe MER range:

| Contribution Margin Before Marketing | Safe MER Range     |
|--------------------------------------|--------------------|
| 60%+                                 | 2.5 – 3.5          |
| 50–60%                               | 3.0 – 4.0          |
| 40–50%                               | 4.0 – 5.0          |
| 30–40%                               | 5.0 – 7.0          |
| Under 30%                            | 7.0+ or pause spend|

Cross-reference with:
- **Industry benchmark MER** from `get_benchmarks` — where does their
  category typically operate?
- **Growth stage** from `about_my_store`:

| Stage          | Budget Logic                                                 |
|----------------|--------------------------------------------------------------|
| Launch         | Small test budget; focus on creative, offer, conversion rate |
| Early traction | Spend only where CAC and MER are stable                      |
| Growth         | Budget by target MER and contribution margin                 |
| Scale-up       | Balance new customer acquisition and retention               |
| Mature         | Budget by marginal MER, contribution profit, and cash flow   |

A startup may temporarily accept a lower MER if it is deliberately buying
learning — creative testing, audience discovery, market proof. The word
"deliberately" matters.

> **Bad reason to accept low MER:** "Meta needs more budget."
> **Good reason:** "We are testing 20 new creatives this month and expect
> lower efficiency while we learn."

Choose a **specific target MER** for this founder — not a range. State your
reasoning: why this number, given their margin, stage, and benchmark.

---

### Step 5 — Build the budget model

Use the worked model to calculate a monthly budget:

```
Input                                    Founder's Numbers
────────────────────────────────────────────────────────────
Target monthly sales                     [from Step 1 or ask]
Contribution margin before marketing     [from Step 3]
Target contribution margin after mktg   [ask founder, or suggest]
────────────────────────────────────────────────────────────
Maximum marketing spend %                = CM before − CM target
Monthly marketing budget                 = Sales × Marketing spend %
Implied target MER                       = Sales ÷ Budget
Daily marketing budget                   = Monthly budget ÷ trading days
```

**Worked example ($250k target, 45% CM before, 20% CM target):**

```
45% − 20% = 25% available for marketing
$250,000 × 25% = $62,500 monthly budget
$250,000 ÷ $62,500 = 4.0 MER
$62,500 ÷ 30 = $2,083/day
```

Present the founder's own numbers in this format. Always show the
calculation steps — do not just output the final number.

---

### Step 6 — MER ↔ spend % conversion

Many founders flip between MER and spend-as-a-percentage-of-revenue. Show
them the translation so they can sanity-check:

| Marketing Spend as % of Sales | Equivalent MER |
|-------------------------------|----------------|
| 10%                           | 10.0           |
| 15%                           | 6.7            |
| 20%                           | 5.0            |
| 25%                           | 4.0            |
| 30%                           | 3.3            |
| 35%                           | 2.9            |
| 40%                           | 2.5            |

If the founder says "we spend 25% of revenue on ads" → MER is 4.0.
If they say "our MER is 5" → they are spending 20% of revenue.

Always translate between the two when the founder uses one of them — they
may not know their MER is equivalent to what they already know as a spend
percentage.

---

### Step 7 — Refine by season and trading context

A monthly budget is a starting point. Adjust it for the trading rhythm:

| Period             | Budget Approach                                          |
|--------------------|----------------------------------------------------------|
| Quiet months       | Efficiency focus, creative testing, list growth          |
| Product launch     | Higher spend, creative push, acquisition priority        |
| Sale period        | Higher spend only if margin supports the discounting     |
| BFCM / Christmas   | Aggressive but tightly controlled; protect CM floor      |
| Post-sale period   | Shift to retention, replenishment, winback               |
| Low stock period   | Reduce acquisition spend; do not buy customers you       |
|                    | cannot fulfil                                            |

For **fashion ecommerce** specifically, also factor in:
- New collection drop calendar
- Inventory depth and size availability
- Sell-through targets and discounting calendar
- Gross margin by product (not just overall)
- Return rate by category
- Seasonality of category demand

If `category` from `about_my_store` is fashion, beauty, or lifestyle, add
a one-paragraph note on the seasonal factors most relevant to their category.

---

### Step 8 — Present the budget output

Present the output in this structure:

```
MARKETING BUDGET — [store_name]
────────────────────────────────────────────────────────────
Target monthly sales:             $XXX,XXX
Contribution margin before mktg:  XX%
Target contribution after mktg:   XX%
────────────────────────────────────────────────────────────
Maximum marketing spend:          XX% of sales
Monthly marketing budget:         $XX,XXX
Daily marketing budget:           $X,XXX/day
Implied MER target:               X.Xx
────────────────────────────────────────────────────────────
Current MER (30-day):             X.Xx  [↑/↓ vs benchmark]
Industry MER benchmark:           X.Xx
````

Follow with:
- One sentence on whether the implied MER is within their safe range
- One sentence on the current MER vs target gap (if live data is available)
- One seasonal or stage-specific note if relevant

Close with: "To track performance against this budget weekly, use
`/cmo-optimize-marketing-budget`."

---

## Constraints

- Always show the full calculation — not just the output number. Founders
  need to understand the logic to own the decision.
- Never set a budget that implies a MER below 2.5 without an explicit
  discussion of the margin risk.
- If contribution margin data is unavailable or low-confidence, ask the
  founder for their estimate before proceeding. Do not assume a margin.
- Do not recommend spending more to fix a falling MER. Diagnose the
  conversion or efficiency problem first.
- If the founder's current MER is already below their safe range, flag this
  before building the budget model — they may need to cut, not add, spend.

---

## What this skill is not

This skill sets the budget number. It does not manage weekly spend
adjustments based on live performance — that is `/cmo-optimize-marketing-budget`.
It does not explain the difference between marketing and advertising — that
is `/cmo-explain-marketing-advertising`.
