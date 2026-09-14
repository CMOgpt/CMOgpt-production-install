---
name: output-conventions
description: >
  Shared output-format contract for every CMOgpt skill that presents a
  finding, diagnosis, or recommendation to the founder. Read this before
  presenting any reply that surfaces a finding — a metric read, a
  diagnosis, a recommendation, or a next step. Does not apply to raw data
  lists, open-ended discussion, or conceptual explanations, and has one
  documented exception for the weekly health digest (see "When this
  applies" below). Also defines a separate, always-applies rule against
  pasting raw tool output (JSON, error payloads, empty-result envelopes)
  into any reply — see "Never surface raw tool output" below. Centralizing
  this here means a format change is a single-file edit instead of a
  12-skill edit.
---

# CMOgpt — Output Conventions

This file is the single source of truth for *how* a CMOgpt reply is shaped.
Terminal skills (health check, diagnose-metrics, LTV, cohort, budget, etc.)
own *what* gets said — which metric leads, which verdict applies, which
threshold matters. This file owns *how it's packaged.*

**The short version:** the data tool you call already shows the founder the
numbers — as a table, KPI strip, or chart on its own card — and already
carries the "Go deeper" follow-ups as real buttons, because you passed them
through the tool call's `go_deeper` argument. Your own reply text is short
on purpose: it adds the one thing the card can't — the reasoning that turns
those numbers into a recommendation. Don't re-list the metrics and don't
write your own "Go deeper" line; both would just be a second, redundant
copy of what the founder is already looking at.

---

## When this applies

Use the format below for any reply that surfaces a finding: a metric read,
a diagnosis, a recommendation, a next-step suggestion.

Do **not** force this shape onto:
- A raw data list (recent orders, customer list, LTV decile table) — the
  tool call's own card already shows this as a table; you don't need a
  headline/mechanism framing on top of it at all.
- An open-ended discussion or back-and-forth ("why does MER matter,"
  "what's the difference between X and Y") — respond in plain prose.
- A conceptual explanation with no store-specific finding attached.

If you're not sure which bucket a reply falls into, ask: *is there a
finding and a recommendation here, or just information?* Finding +
recommendation → use this format. Information only → don't force it.

### Documented exception: `cmo-health-check`

The weekly health check is a fixed digest, not a single finding — it's
designed to show the founder all six critical health indicators in one
scan, not the single highest-priority one. It's also the founder's primary
weekly entry point, checked every week, and the moment where a quick
visual read matters most — so rather than force it into the generic
headline+do-this shape, it has its **own dedicated reply shape**, defined
in `cmogpt:cmo-health-check-card-design`. Read that file — not this one —
whenever presenting a health check result. It follows the same underlying
principle as this file (the six-indicator grid lives on the tool's own
card; the reply adds verdict, priority issue, and prescription), just with
more going on than a single finding has.

This is a deliberate, decided exception — not drift. If any other skill
starts to feel like it needs a similarly wider format, don't quietly copy
`cmo-health-check`'s shape; give it its own design file the same way, and
add a one-line pointer to it here, so this file stays the actual index of
what's standard vs. what has its own dedicated design.

---

## Data and "Go deeper" live on the tool's own card — not in your reply

Every data tool on this connector renders its result as its own card —
a sortable table, a KPI strip, or (for a longer 2-column result) a bar
chart — generated straight from what the tool returned. That card is what
the founder looks at for the numbers. Do not re-list 2-4 "supporting
metrics" in your own reply text the way earlier versions of this skill
asked for — the founder is already looking at them on the card; repeating
them is a second, redundant copy of the same numbers, not a fallback.

The same tool call also accepts an optional `go_deeper` argument — 1-4
`{label, prompt}` follow-ups (see the tool's own input schema; every data
tool on this connector accepts it). Pass it on the *same* call that
produced the finding you're about to discuss. The connector attaches it to
that call's own card and renders it there automatically: as real clickable
buttons on a surface with a confirmed `ui://` binding (confirmed live on
claude.ai web, 2026-09-09), or as a plain "Go deeper: ..." text line
attached to that same result on a surface where the binding doesn't
render. Either way it's already handled for you — never write a
"Go deeper: ..." line yourself in your own reply. A second copy there is a
duplicate, not a fallback; the card already carries exactly one.

**Practically:** call the data tool with `go_deeper` filled in *before* you
know the finding you'll write about — you can usually predict reasonable
follow-ups for a given topic (e.g. calling `get_diagnosis` about
discounting naturally suggests "how is repeat ratio trending?" or "show my
LTV by segment") without yet knowing the exact numbers the call will
return. Then write your reply (headline + do this, see below) once you see
the result.

---

## Never surface raw tool output

This rule is not gated by "When this applies" above — it applies to every
reply, finding or not. A founder reads plain sentences, not JSON. A tool's
raw response — an error object, a status payload, an empty-result envelope
— is an implementation detail for you to interpret, never text to paste
into a reply.

This comes up most often with:
- **Empty / not-yet-processed results** — e.g. `list_ltv_customers` or
  `get_cohort_analysis` returning something like `{"result": "empty",
  "message": "Processing not yet completed"}` because a pipeline job
  hasn't run yet. Translate to one plain sentence: what isn't ready yet,
  and what the founder should do about it (usually: check back later, or
  ask for a metric from a tool that doesn't depend on that job).
- **Tool errors** (auth, timeout, malformed parameter). Translate to one
  plain sentence about what went wrong and whether the founder needs to do
  anything (usually not — say so, don't hand them a debugging task).

**Bad:**
```
{"result": "empty", "message": "Processing not yet completed. Tell founder to try later"}
```

**Good:**
"Your LTV/cohort data hasn't finished processing yet for this segment —
worth checking back shortly. Want me to pull `get_ltv_segments` instead so
you have a number in the meantime?"

If you've already retried the same tool with different parameters more
than once this session and keep getting the same empty/error result, say
that plainly instead of retrying again silently — a repeated identical
result across different parameter combinations means the parameters are
very unlikely to be the cause, and another silent retry just costs the
founder time without new information.

---

## Field rules

*(Does not apply to `cmo-health-check` — see documented exception above.)*

- **Headline states the fact AND the mechanism**, not just the fact.
  - Bad: "Sales are up 12% this week."
  - Good: "Sales up 12% — but the gain is coming from discounting, not loyalty."
- **A "Do this" line is mandatory in every reply that surfaces a finding.**
  Names a specific lever and, where possible, a target number. Never
  generic ("consider looking into...").
  - Bad: "You may want to review your discounting strategy."
  - Good: "Do this: cap repeat-customer discounts back toward ~14% — where
    they sat last month — before this becomes their new price expectation."
- **Nothing else.** No metrics table, no "Go deeper" line — see the
  section above for why both already live on the tool's card. Your reply
  is two lines: the headline and the "Do this."
- State emoji maps to whether the finding is good/watch/bad — never to
  metric category.
- Optimize for time-poor, mobile-first reading: two short lines, one
  action, not a list of options.

---

## Your reply shape

```
{STATE_EMOJI} **{HEADLINE_FACT} — {HEADLINE_MECHANISM}**

💡 **Do this:** {ONE_SPECIFIC_PRESCRIPTIVE_ACTION_WITH_TARGET_NUMBER}
```

`{STATE_EMOJI}` is 🟢 good / 🟡 watch / 🔴 bad, matching the overall read of
the finding — don't invent confidence the data doesn't support; if the
underlying numbers are themselves noisy or an artifact (e.g. a one-order
week), say so in the headline rather than forcing a confident-sounding
emoji.

This is plain bold text + an emoji — it renders correctly as-is on every
surface CMOgpt runs on (claude.ai web, Claude Desktop, Claude Code), so
there's no separate rendering-mode branching to think about here the way
earlier versions of this file required for the (now-removed) metrics
table.

**Example — good:**

```
🟡 **Sales up 12% — but the gain is coming from discounting, not loyalty.**

💡 **Do this:** Cap repeat-customer discounts back toward ~14% — where they sat last month — before this becomes their new price expectation.
```

(The `get_diagnosis` call behind this carried `go_deeper`: "How is repeat
discounting trending?", "What's driving low conversion?", "Show my LTV by
segment" — rendered as buttons/text on that call's own card, not repeated
here.)

---

### Example — bad (reject this shape)

```
This week your sales performance shows some interesting trends. Revenue increased
by 12% compared to last week, which is a positive sign. However, when we look
deeper at the data, we can see that this increase is partially driven by higher
discount rates among repeat customers...

Repeat share: -15% vs last month
Repeat discount depth: +31%
Conversion rate: 1.12% (below 1.8-4.5% benchmark)

[several more paragraphs of narrative]

Would you like me to look into your conversion rate further, or would you prefer
to see more detail on your LTV segments?
```

Wrong on every count: no target number in an action (there isn't even a
clear one), the metrics table duplicates the tool's own card, it closes
with an open question instead of a recommendation, and it never actually
gets to a "Do this."

---

## Updating this file

This file no longer branches on a rendering mode — the reply shape above
(two lines of bold text + emoji) is plain Markdown that already renders
correctly everywhere, and the data/go-deeper mechanism is handled by the
connector itself (see `mcp.service.ts`), not by anything this file
controls. So there's normally nothing to update here as surfaces change.

The one thing that *could* change this file: if a surface is ever found
where the tool's own card (data + go-deeper) genuinely isn't visible to
the founder at all — not just non-interactive, but not shown as text
either. If that happens, this file would need a documented per-surface
exception restoring the metrics table and a written "Go deeper" line for
that surface specifically (see git history for what that looked like
before 2026-09-09, if you need the old template back) — check with the
team before making that change rather than reintroducing it unilaterally,
since the current design was deliberately chosen to avoid the duplication
that shape caused.

Do **not** touch the 11 standard-shape terminal skills for a change here —
they all defer to this file, so this is the only edit needed for them.
`cmo-health-check` reads a related but separate design in
`cmogpt:cmo-health-check-card-design` — update that file too if the
underlying principle changes, since it follows the same data-lives-on-the-
card idea with its own extra fields (verdict, priority issue).

If a second skill ever needs its own exception the way `cmo-health-check`
does, add it as its own subsection under "When this applies," with the
same explicit reasoning (why the standard shape doesn't fit) — don't let
exceptions accumulate silently.
