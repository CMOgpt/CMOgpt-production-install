---
name: cmo-health-check-card-design
description: >
  Content spec for `cmo-health-check` specifically — what the weekly digest
  says and in what order. This is not the generic finding shape defined in
  `cmogpt:output-conventions`; health check shows all six critical
  indicators in one scan, not a single finding. Both this file and the
  standard shape render through the shared `cmogpt:cmo-pulse-card`
  component — read this file for *what* the health check says, and
  `cmogpt:cmo-pulse-card` for how that becomes the actual card.
---

# CMOgpt — Health Check Card Design

## Why this is separate from the standard finding

The standard `output-conventions` reply is built for one finding from one
tool call: a headline, a mechanism, one action. Health check carries more
than that by design — six to seven weekly indicators pulled from six
separate tool calls, a priority issue, a prescription, and a verdict —
because it's meant to answer "is my business okay this week?" in one
glance, not drill into one thing. That's a *content* difference (what the
finding contains), not a rendering one — both this file and the standard
shape publish through the same `cmogpt:cmo-pulse-card` component, just
with a different number of supporting tiles (health check: up to 6;
standard finding: 0–1).

## Content model

Every health check produces exactly these parts, in this order — this is
what goes into the Pulse Card JSON (see `cmogpt:cmo-pulse-card` for the
exact schema and the publish procedure):

1. **Store + period context** → `store_name`, `period_label`
   ("Week ending {date}"), and `freshness_note` if the data is stale (more
   than 2 days behind today — see `cmo-health-check`'s procedure).
2. **Verdict** → `verdict.label`, one of: Growing profitably / Growing
   with risk / Needs attention. `verdict.tone`: good / watch / bad
   respectively.
3. **Headline metric** → `hero`: total sales, with its week-over-week
   delta. This is the one number a founder should absorb in half a second.
4. **Supporting metric grid** → `tiles`: the six indicators (AOV, discount
   %, marketing spend, MER, contribution margin, repeat ratio), each with
   value and delta and a per-metric `tone` (favorable direction is
   metric-specific — falling discount % is `"up"`/green, falling sales is
   `"down"`/red).
5. **Priority issue** → `priority`: the single metric most likely to
   compound if unaddressed, called out with real numbers. Use `tone:
   "watch"` or `"bad"` depending on severity — see `cmo-health-check`'s own
   priority-selection rules (contribution margin as centre of gravity,
   trend-can-outrank-position, etc.) for *which* metric leads; this file
   only says where it goes in the card.
6. **Prescription** → `do_this`: one to two concrete actions with a target
   number. Never generic.
7. **Go deeper** → `go_deeper`: 2–4 follow-up questions the founder would
   naturally ask next, as an array — these render as clickable chips on
   the card (see `cmogpt:cmo-pulse-card`'s "Why Go Deeper copies instead of
   asking").

Verdict → tone mapping:
- Growing profitably → `good` (🟢)
- Growing with risk → `watch` (🟡)
- Needs attention → `bad` (🔴)

## Chat reply

After building the content above, publish/update the Pulse Card per
`cmogpt:cmo-pulse-card`'s procedure, then reply in chat with just the
verdict and the priority issue — not the full grid, not the go-deeper
questions, those live on the card:

```
{VERDICT_EMOJI} **{STORE_NAME} — {VERDICT_LABEL}.** {ONE_LINE_ON_TOTAL_SALES_AND_ITS_DELTA}

💡 **Do this:** {PRESCRIPTION}

Card updated above with all six indicators and this week's follow-ups.
```

**Worked example (MAKU STUDIOS, week ending Sept 14, 2026):**

```
🟢 **MAKU STUDIOS — growing profitably.** Total sales up 37.8% to $59,231.

💡 **Do this:** Hold marketing budget flat at $1,290/day — confirm Sept 14 was a genuine one-order day before reacting to the contribution-margin dip it caused.

Card updated above with all six indicators and this week's follow-ups.
```

The Pulse Card behind that reply carries all six tiles (AOV, discount
rate, marketing spend, MER, contribution margin, repeat ratio), the fuller
priority explanation, and the go-deeper chips — none of that is repeated
in the chat text.

## What did not change

The underlying business logic in `cmo-health-check` — which metric leads,
the six-indicator system read, the verdict thresholds, the priority-issue
selection rules — is unchanged by this file. This file only defines what
the health check's *content* is and the order it's assembled in; the
diagnosis logic still lives in `cmo-health-check`'s own procedure, and the
actual card markup and publish mechanics live in `cmogpt:cmo-pulse-card`.
