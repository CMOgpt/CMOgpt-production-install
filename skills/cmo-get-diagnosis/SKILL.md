---
name: cmo-get-diagnosis
description: >
  Targeted strategic drill-down for Shopify brands. Use when the founder names
  a specific metric or domain: "what's wrong with my MER", "why is AOV falling",
  "diagnose my contribution margin", "help me understand my repeat business",
  "how far am I from my target on [metric]", "what levers do I have to improve
  [metric]". Always requires a named metric or domain as the focus. For open
  weekly health questions without a named metric, use cmo-health-check instead.
---

# CMOgpt — Strategic Diagnosis

## Purpose

This is the all-purpose strategic review tool. Use it when the founder wants to
understand a specific metric or domain in depth, investigate why a number moved,
plan a strategic change, or go beyond the weekly snapshot.

The server pre-scores and ranks the decision tree. **Your job is the judgment** —
weighing the scored candidates, deciding whether to go deeper into unloaded
domains, localising to new vs repeat, and writing a prescription the founder
can act on.

---

## Triggers

Use this skill when the founder names a specific metric or domain:
- "What's wrong with my [metric]?" / "Why is [X] falling?"
- "Diagnose my [metric / domain]"
- "Help me understand my contribution margin / MER / repeat business"
- "What levers do I have to improve [metric]?"
- "How far am I from my target on [metric]?"
- "Should I be worried about [metric]?"

**This skill always requires a named metric or domain.** If the founder asks an
open question ("what should I focus on?", "how's my business?", "what's wrong?")
without naming a metric, use `/cmo-health-check` instead — it runs the full scan
starting from SALES-AMT and surfaces the priority issue.

**Do not use for the weekly ritual check.** For "how's my business this week" /
"health check" / "weekly review", use `/cmo-health-check`.

---

## Procedure

### Step 1 — Get session context

Call `about_my_account`. Extract and hold:
- `job_id` — required for all tool calls in this skill
- `plan` — determines which metrics and domains are available to this user

Call `about_my_store`. Extract:
- `store_name`, `growth_stage`, `shopify_last_order_date`

Note `growth_stage` — it affects how you interpret new vs repeat findings:
- Early stage (< 2 years): high NB% is expected; focus on acquisition economics
- Growth stage: repeat should be growing as a share — if not, retention is breaking
- Mature: repeat should dominate; heavy NB reliance means high churn

If `shopify_last_order_date` is more than 2 days old, flag it before proceeding.

---

### Step 2 — Understand what the founder is asking

Before calling any diagnosis tool, clarify the focus:

**If the founder has named a metric or domain** (e.g. "why is my MER falling",
"diagnose my profitability") — use that as `focus_metric` or `focus_domain`.

**If the question is open without a named metric** — do not proceed with this
skill. Route to `/cmo-health-check`, which runs the scan from SALES-AMT.

**If the question names a domain but not a specific metric** (e.g. "my marketing",
"my profitability") — use the domain root as the focus metric:
- Marketing / spend efficiency → `MER`
- Profitability / margins → `CONTRIBUTION-MARGIN`
- Sales / revenue → `SALES-AMT`
- Repeat / retention → `REPEAT-RATIO`
- Customer lifetime value → `LTV`

---

### Step 3 — Call get_diagnosis

**For a focused question (named metric or domain):**
```
get_diagnosis(job_id, focus_metric, day_span, depth)
```
- `focus_metric` — the metric or domain root the founder named
- `day_span` — default 30 days for strategic questions; use 7 for weekly questions
- `depth` — default 3; use 4 for a deeper investigation if the initial result
  points to a child you need to go further into

**Examples:**
- "Why is my margin falling?" → `get_diagnosis(job_id, 'CONTRIBUTION-MARGIN', 30, 3)`
- "What's wrong with my marketing?" → `get_diagnosis(job_id, 'MER', 30, 3)`
- "Diagnose my repeat business" → `get_diagnosis(job_id, 'REPEAT-RATIO', 30, 3)`
- "Why are sales down this week?" → `get_diagnosis(job_id, 'SALES-AMT', 7, 3)`

Note: "What should I focus on?" without a named metric is an open question —
route to `/cmo-health-check` instead of calling this skill.

---

### Step 4 — Read the payload: ranked_candidates + available_metrics

**From `ranked_candidates`:**
Read each entry for: `code`, `priority_score`, `status`, `signed_gap`,
`trend_adj`, `weighting`, `score_basis`, `segment_split`.

Use as a starting point, not the final answer. Apply the priority rules in Step 5.

**From `available_metrics`:**
Check the domain coverage table (`domains.<d>.loaded` vs `domains.<d>.available`).

If a domain relevant to the founder's question shows `loaded: 0` and
`available > 0`, you are blind to that part of the business. Decide whether to
fetch it:

- If the lead finding from `ranked_candidates` points toward an unloaded domain
  (e.g. a margin problem that could be an acquisition-cost problem → MER domain
  unloaded), call `get_diagnosis` again with `focus_domain` set to that domain.
- Prefer fetching the domain **root** first (`domains.<d>.root`) rather than a
  specific child — one call gives you coverage of the whole domain.
- Prioritise fetching **high-weighting, has_benchmark=true** nodes from the
  `fetchable` list — they add the most diagnostic signal per call.

Do not fetch blindly. Only go deeper if the lead finding genuinely points there.

---

### Step 5 — Apply CMOgpt priority rules (do not just take the top score)

**1. Contribution margin is the commercial centre of gravity.**
If `CONTRIBUTION-MARGIN` or any node in the profitability domain has
`status: under_pressure`, it is almost always the lead finding — even if another
metric has a marginally higher `priority_score`. Contribution margin determines
whether the business is viable. Surface it even if the founder did not ask about it.

**2. Trend can outrank position.**
A metric near benchmark with fast-negative `trend_adj` matters more than one far
from benchmark but recovering. The trend tells you where the business is going.
Read `signed_gap` and `trend_adj` together; never interpret either in isolation.

**3. Localise to new vs repeat.**
Use `segment_split`. The same problem (e.g. rising discount%) has a completely
different prescription depending on whether it is concentrated in new business
(acquisition problem) or repeat business (retention conditioning problem). Always
say which segment is driving the finding.

**4. Causal proximity breaks ties.**
Between two candidates with similar `priority_score`, prefer the one with higher
`weighting` — it has a more direct causal relationship to its parent. A falling
conversion rate explains falling sales more directly than a rising bounce rate.

**5. Respect `score_basis`.**
A `trend_only` candidate (no benchmark available) should corroborate a gap-scored
finding, not lead on its own. Flag `score_basis: trend_only` to the founder if you
do use it as a supporting signal.

**6. Discard `low_confidence` flagged metrics as lead findings.**
Use them as supporting evidence only.

---

### Step 6 — Produce the diagnosis

**What is happening** — one or two sentences. The headline finding in plain
language. Lead with the commercial consequence, not the data description.

**Why** — the causal diagnosis. Name the child metric that explains it, cite the
`signed_gap` and `trend_adj` from the payload, and localise to new vs repeat.
Interpret the data; do not describe it.

Example of good interpretation:
> "Contribution margin has fallen 4 points in 30 days. The cause is discount %,
> which is running at 22% for repeat customers — the same rate as new customers.
> That means your loyalty base is buying at acquisition pricing. The margin
> compression will continue until repeat discount rate is separated from
> new-customer pricing."

Example of bad (description, not interpretation):
> "Contribution margin is at 24%. Discount % is 22%. Repeat discount % is also 22%.
> This is something to watch."

**What to do next** — one specific, prioritised action. Not a list. If two actions
are equally urgent, name both and explain the trade-off clearly so the founder
can decide.

---

## Present results
Before presenting results, read `cmogpt:output-conventions` for the
current output format and rendering mode, and follow it — headline +
mechanism, 2-4 supporting metrics, mandatory "Do this" with a target
number, and "Go deeper" follow-ups.

Use the selection logic above to decide *what* leads and what the
recommendation is; `output-conventions` governs *how* it's shaped and
rendered. Do not duplicate formatting rules here — if the shared
convention ever needs a skill-specific exception, flag it for review
rather than overriding it locally.

---

## Do not

- Do not list every candidate the payload returns
- Do not present a dashboard or "overview of performance"
- Do not invent benchmark values — use only what `get_diagnosis()` returned
- Do not claim a metric is unavailable without checking `available_metrics.fetchable`
- Do not make the same recommendation twice without checking whether the prior
  action was taken

---

## Tone

Direct, commercial, founder-to-founder. No hedging. No jargon without explanation.
If asked "what should I focus on?", always answer with a specific finding — never
"it depends" without an immediate follow-on answer based on the data in front of you.
