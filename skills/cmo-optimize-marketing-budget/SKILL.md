---
name: cmo-optimize-marketing-budget
description: >
  Weekly and daily marketing spend review for Shopify brands. Monitors MER,
  contribution margin, and sales trend against the founder's budget plan and
  recommends a specific spend adjustment: increase, hold, reduce, or cut.
  Triggers when the founder asks "should I increase my ad spend", "how is
  my marketing performing this week", "adjust my marketing budget", "is my
  MER on track", "should I cut spend", "review my ad budget", or any
  question about whether current marketing spend is working and what to
  change. Produces one recommendation with a specific dollar amount, not a
  range of options.
---

# CMOgpt — Optimize Marketing Budget

## Purpose

This skill is the weekly operational rhythm for marketing spend. It reads
live MER, sales trend, and contribution margin against the founder's budget
plan, applies a five-condition decision framework, and issues one specific
recommendation: increase, hold, reduce, or cut — with a number attached.

The job is not to explain the strategy. It is to tell the founder what to
do with their budget this week, and why.

---

## Triggers

Use this skill when the founder asks:
- "Should I increase my ad spend?"
- "How is my marketing performing this week?"
- "Adjust my marketing budget" / "Review my ad spend"
- "Is my MER on track?" / "My MER is down — what should I do?"
- "Should I cut spend?" / "Can I scale up spend?"
- "Weekly marketing review"
- Any question about whether to change current spend levels

---

## Core principle: weekly signal, not daily noise

Shopify revenue is volatile day to day. A single bad day does not mean
spend should be cut. A single good day does not mean spend should be
scaled. This skill reads weekly trends — not daily movements — before
issuing any recommendation to change spend.

> Never cut spend after one bad day.
> Never scale spend after one good day.
> Weekly trend is the signal. Daily movement is the noise.

The only exception: if MER is deeply negative and contribution margin is
being destroyed (the "cut quickly" condition), act immediately regardless
of day count.

---

## Procedure

### Step 1 — Get store and account context

Call `about_my_account`. Extract:
- `plan` — determines metric access

Call `about_my_store`. Extract:
- `store_name` — for the review header
- `growth_stage` — informs spend aggressiveness
- `shopify_last_order_date` — data freshness check

**Data freshness check:** If `shopify_last_order_date` is more than 2 days
before today, note: "Data is current to [date] — yesterday may not yet be
included." Proceed; do not abort.

---

### Step 2 — Pull the five key metrics

Call these in parallel:

```
get_metric_history('MER', 14)
get_metric_history('SALES-AMT', 14)
get_metric_history('CONTRIBUTION-MARGIN', 14)
get_metric_history('BUDGET-DAILY', 14)
get_metric_history('ORDER-DISC-PCT', 14)
get_marketing_budget()
get_my_targets()
get_benchmarks()
```

From the results, calculate for each metric:
- **This week** — 7-day average (days 1–7)
- **Last week** — 7-day average (days 8–14)
- **Direction** — improving, stable, or deteriorating
- **vs target** — from `get_my_targets`
- **vs benchmark** — from `get_benchmarks`

Also extract:
- **Current daily budget** — from `get_marketing_budget`
- **MER target** — from `get_my_targets` if set; otherwise derive from the
  contribution margin (see `/cmo-set-marketing-budget` logic)

---

### Step 3 — Apply the spend decision framework

Read the five metrics as a system, then apply the framework in order.
Stop at the first condition that matches.

#### Condition 1 — Scale up
**MER above target AND sales growing week-on-week**

The marketing is working. The business is growing within safe economics.
Increase spend 10–20%.

Calculate the increase in dollar terms:
```
New daily budget = Current daily budget × 1.10 to 1.20
Monthly impact   = New daily budget × 30
```

State the upper bound only if contribution margin has headroom above the
target. Do not scale past the CM floor set in `/cmo-set-marketing-budget`.

---

#### Condition 2 — Hold
**MER on target AND sales stable (±5% week-on-week)**

Performance is on plan. Hold spend at current level.

Note any early-warning signals even while holding:
- Is ORDER-DISC-PCT creeping up? (Efficiency may be hiding behind discounts)
- Is CONTRIBUTION-MARGIN compressing even if MER looks healthy?
- Is the MER trend flat but approaching the bottom of the safe range?

If any of these are present, flag them as "watch items" even on a Hold
decision. They may trigger a Reduce next week if they persist.

---

#### Condition 3 — Investigate (week 1 below target)
**MER below target for 1 week**

Do not cut spend yet. Diagnose first.

Check in this order:
1. **Creative fatigue** — is the same creative running for 3+ weeks without
   a refresh? Frequency rising while CTR falling is the tell.
2. **Offer weakness** — is the discount level or free-shipping threshold
   no longer competitive? Check ORDER-DISC-PCT trend.
3. **Conversion rate** — is traffic landing but not converting? MER can
   fall even with good spend if the site or landing page is broken.
4. **AOV** — is revenue per order falling? A lower AOV means the same
   spend buys less revenue, which compresses MER mechanically.

Issue a diagnosis, not a spend cut. Tell the founder what to fix this week.
If the issue is diagnosable from the data, name it specifically. If it
requires deeper investigation, route to `/cmo-diagnose-metrics`.

---

#### Condition 4 — Reduce (week 2 below target)
**MER below target for 2 consecutive weeks AND the Condition 3 issues have
not been resolved**

Reduce spend 15–25%. Do not cut to zero — some spend maintains presence
and customer flow while the issue is fixed.

Calculate the reduction in dollar terms:
```
New daily budget = Current daily budget × 0.75 to 0.85
Monthly saving   = (Current − New) × 30
```

Restate the diagnosis from Condition 3. The reduction buys time to fix the
underlying problem. It is not the fix itself.

---

#### Condition 5 — Cut quickly
**MER below target AND contribution margin is negative (or approaching zero)**

This is the only condition that overrides the weekly-trend rule. If the
brand is actively losing money on every marketing dollar, speed matters.

Cut spend significantly — recommend a 40–60% reduction or to a minimum
maintenance level, whichever is higher.

```
Minimum maintenance spend = enough to keep core retargeting and
brand search live; not enough to continue prospecting at scale
```

State clearly: this is not a performance optimisation. The brand is spending
into negative contribution. The priority is to stop the bleeding, then
diagnose.

Route to `/cmo-diagnose-metrics` on CONTRIBUTION-MARGIN immediately after the
spend cut recommendation.

---

### Step 4 — Check for overrides before issuing the recommendation

Before finalising the recommendation, check for context that overrides the
standard framework:

**Override: deliberate test period**
If the founder has noted they are running a creative test, audience
experiment, or launch phase — a temporary low MER is expected. Confirm
whether this is still within the test window before recommending a cut.
Ask: "Are you still in a planned test period, or has this run past the
expected learning window?"

**Override: seasonal spike**
If BFCM, Christmas, or a major sale period is within 2 weeks, do not
reduce acquisition spend on a single week of soft MER. Note the seasonal
context and hold, unless Condition 5 applies.

**Override: recent stock or fulfilment issue**
If there is a known stock-out, fulfilment delay, or product issue, MER
may be falling because orders cannot be completed — not because ads are
failing. Spending into a fulfilment problem amplifies losses. In this
case, reduce to maintenance spend regardless of MER.

---

### Step 5 — Translate the recommendation into a specific action

The output must contain a specific dollar amount. Not a range. Not a
direction. A number.

```
Current daily budget:     $X,XXX/day
Recommendation:           [Increase / Hold / Reduce / Cut]
New daily budget:         $X,XXX/day
Change:                   +/− $XXX/day  (+/− XX%)
Monthly budget impact:    +/− $XX,XXX/month
```

State the one-line reason: which condition triggered it and what metric
drove the decision.

If the recommendation is Investigate (Condition 3), the output is the
diagnosis and the specific thing to fix — not a spend number change.

---

### Step 6 — Present the weekly spend review

**Header:**
```
MARKETING SPEND REVIEW — [store_name] — week ending [date]
```

**Five-metric summary:**
```
Metric                 This week   Last week   Change    vs Target
───────────────────────────────────────────────────────────────────
MER                    X.Xx        X.Xx        ↑/↓ X%    [↑/↓/=]
Sales                  $XX,XXX     $XX,XXX     ↑/↓ X%    —
Contribution Margin    XX%         XX%         ↑/↓ Xpp   —
Daily Budget           $X,XXX      $X,XXX      ↑/↓ X%    —
Discount %             XX%         XX%         ↑/↓ Xpp   —
```

**Decision:**
State the condition number and the recommendation in one sentence.

Example:
> "MER has been below your 4.0 target for two consecutive weeks and the
> creative refresh from last week has not yet moved the needle. Condition 4:
> reduce spend."

**Action:**
The specific budget change with numbers.

**Watch items (if any):**
One or two early-warning signals to monitor next week, even on a Hold.

---

## Present results
Before presenting results, read `cmogpt:output-conventions` for the
current output format and follow it — a two-line reply (headline +
mechanism, then a mandatory "Do this" with a target number), since the
data itself and the "Go deeper" follow-ups already live on the tool
call's own card.

Use the selection logic above to decide *what* leads and what the
recommendation is; `output-conventions` governs *how* it's shaped and
rendered. Do not duplicate formatting rules here — if the shared
convention ever needs a skill-specific exception, flag it for review
rather than overriding it locally.

---

## Diagnosis rules when MER is below target

When Condition 3 applies, check these in order before recommending a fix:

| Signal                                | What it means                                        |
|---------------------------------------|------------------------------------------------------|
| MER down, spend held, orders down     | Conversion or traffic quality problem — not spend    |
| MER down, spend up, revenue flat      | Efficiency falling; creative or audience fatiguing   |
| MER down, ORDER-DISC-PCT up           | Discounts buying revenue, not marketing working      |
| MER down, AOV down                    | Mix shift or bundle/upsell breakdown                 |
| MER down, CONTRIBUTION-MARGIN stable  | Revenue problem, not margin crisis — diagnose first  |
| MER down, CONTRIBUTION-MARGIN down    | Both efficiency and margin deteriorating — escalate  |

For any of these combinations, the prescription differs. Do not issue a
generic "fix your creative" recommendation without knowing which signal is
present.

---

## Tone and constraints

- Issue one recommendation. Not a list of options. The founder wants to
  know what to do, not what they could do.
- Always attach a dollar amount. "Reduce spend" means nothing without
  "reduce to $1,400/day from $1,850/day."
- Do not recommend increasing spend to fix a falling MER. Diagnosis first.
- Do not use percentage adjustments alone — translate everything into
  dollar amounts the founder can enter into their ad platform today.
- Keep the full review readable in under 3 minutes.

---

## Routing

- To understand how the current budget was set → `/cmo-set-marketing-budget`
- To understand MER and marketing spend concepts → `/cmo-explain-marketing-advertising`
- To diagnose a specific underperforming metric in depth → `/cmo-diagnose-metrics`
- For full weekly business health check → `/cmo-health-check`
