---
name: cmo-diagnose-contribution-margin
description: >
  Domain sub-skill for diagnosing contribution margin. Consumes the CONTRIB-MARGIN
  branch of the get_diagnosis() payload — discount %, COGS %, shipping subsidy %,
  marketing % — and reasons about why margin is moving, isolating new-business vs
  repeat economics. Use when the founder asks about profitability, "where is my
  profit going", discounting, or when get_diagnosis flags margin under pressure.
---

# CMOgpt — Contribution Margin (domain sub-skill)

## When to use
The founder asks about profitability, why profit is shrinking while sales grow,
discounting, shipping cost, or "am I actually making money" — or `cmo-diagnose-metrics`
or `cmo-health-check` has identified contribution margin as the lead finding and is drilling in.

## Inputs
Use the `CONTRIB-MARGIN` domain node from `get_diagnosis()`. Its children are the
margin drivers: `ORDER-DISC-PCT`, `COGS-PCT`, `SHIPPING-SUBSIDY-PCT`,
`MARKETING-PCT`. Each carries value, benchmark band, `gap_score`, `trend`,
`causal_proximity`, `priority_score`, and `segment_split`.

If you do not already have a fresh payload, call
`get_diagnosis(job_id, 'CONTRIBUTION-MARGIN', 30, 3)`.

## Domain reasoning heuristics
- **Margin is the centre of gravity.** Treat a margin problem as the most important
  commercial signal even if the founder asked about something else.
- **Discounting vs cost are different stories.** A discount-driven margin drop is a
  go-to-market problem (you traded margin for volume); a COGS- or shipping-driven
  drop is a unit-economics problem. The prescription differs — name which one.
- **New vs repeat is decisive here.** Use `segment_split`. Heavy first-order
  discounting to acquire new customers is a CAC problem in disguise; margin erosion
  in repeat business means loyalty that doesn't pay. Diagnose the right one.
- **Revenue can mask margin.** If sales are up but `CONTRIB-MARGIN` trend is down,
  that divergence is the finding — surface it even if unasked.
- **Read trend and gap together.** Margin at benchmark but falling fast is a
  forming problem; margin below benchmark but recovering is a managed one.

## Output
**What is happening** — margin headline (e.g. "margin is down ~13 pts on rising
new-customer discounting, not cost").
**Why** — the driver child that explains it, with its benchmark gap and trend, and
the segment it sits in.
**What to do next** — one prioritised action tied to the actual driver (e.g. cap
first-order discount depth, not "cut costs"). State a trade-off only if real.

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


## Do not
- Do not recommend "cut costs" generically when the driver is discounting.
- Do not report all four drivers — lead with the one that explains the move.
- Do not use benchmark numbers other than those returned by get_diagnosis().
