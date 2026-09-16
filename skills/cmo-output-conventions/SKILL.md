---
name: output-conventions
description: >
  Shared output-format contract for every CMOgpt skill that presents a
  finding, diagnosis, or recommendation to the founder. Read this before
  presenting any reply that surfaces a finding — a metric read, a
  diagnosis, a recommendation, or a next step. Every such finding renders
  through the shared Pulse Card (`cmogpt:cmo-pulse-card`), hero-only for a
  single metric. Does not apply to raw data lists, open-ended discussion,
  or conceptual explanations, and has one documented exception for the
  weekly health digest (see "When this applies" below). Also defines a
  separate, always-applies rule against pasting raw tool output (JSON,
  error payloads, empty-result envelopes) into any reply — see "Never
  surface raw tool output" below. Centralizing this here means a format
  change is a single-file edit instead of a 12-skill edit.
---

# CMOgpt — Output Conventions

This file is the single source of truth for *how* a CMOgpt reply is shaped.
Terminal skills (health check, diagnose-metrics, LTV, cohort, budget, etc.)
own *what* gets said — which metric leads, which verdict applies, which
threshold matters. This file owns *how it's packaged.*

**The short version:** every finding renders as a Pulse Card — a
persistent, visual artifact defined and built in `cmogpt:cmo-pulse-card` —
plus a short two-line chat reply that gives the fast read for someone
scanning history who never opens the card. Your own reply text stays
short on purpose: the card carries the number(s); your two lines carry the
reasoning that turns those numbers into a recommendation. Don't re-list
metrics or write a "Go deeper" line in your reply text — both live on the
card.

---

## When this applies

Use the format below for any reply that surfaces a finding: a metric read,
a diagnosis, a recommendation, a next-step suggestion.

Do **not** force this shape onto:
- A raw data list (recent orders, customer list, LTV decile table) — the
  tool call's own card already shows this as a table; you don't need a
  headline/mechanism framing, and no Pulse Card either.
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
visual read matters most — so it has its **own dedicated content spec**,
defined in `cmogpt:cmo-health-check-card-design`. Read that file — not
this one — whenever presenting a health check result. It renders through
the same Pulse Card as everything else (see below), just with up to 6
supporting metrics instead of 0–1.

This is a deliberate, decided exception for *content* (what leads, the
six-indicator system read, the verdict thresholds) — not for rendering.
Both files defer to `cmogpt:cmo-pulse-card` for how the result actually
gets built and published. If any other skill starts to feel like it needs
its own content spec the way `cmo-health-check` does, give it one the same
way, and add a one-line pointer to it here.

---

## Every finding renders as a Pulse Card

Every data tool on this connector still renders its own result as a card
too — a sortable table, a KPI strip, or a bar chart, generated straight
from what the tool returned, and still worth passing `go_deeper` on (see
below) since a surface with a confirmed `ui://` binding renders those as
real clickable buttons that post directly into the conversation. That
tool-level card is a useful bonus, but it is **not** the finding's primary
presentation anymore — the Pulse Card is.

For every finding, follow the procedure in `cmogpt:cmo-pulse-card`: build
the content (this file's field rules below govern quality), assemble the
card's JSON, fill the template, and publish/update the one Pulse Card
artifact for that store. Read that file for the full mechanics — it is
the single source of truth for the *rendering*, the same way this file is
the single source of truth for the *reasoning*.

**Still pass `go_deeper` on the underlying tool call too** — 1–4
`{label, prompt}` follow-ups, same as before. It's low-cost and gives a
bonus native card for anyone on a `ui://`-capable surface. But the
follow-ups a founder is actually meant to use are the Pulse Card's Go
Deeper chips — see `cmogpt:cmo-pulse-card`'s "Why Go Deeper copies instead
of asking" for why those work differently (copy-to-clipboard, not a live
button) and why that's a deliberate, documented platform limit, not an
oversight.

---

## Never surface raw tool output

This rule is not gated by "When this applies" above — it applies to every
reply, finding or not. A founder reads plain sentences, not JSON. A tool's
raw response — an error object, a status payload, an empty-result envelope
— is an implementation detail for you to interpret, never text to paste
into a reply, and never something to put into a Pulse Card's JSON either.

This comes up most often with:
- **Empty / not-yet-processed results** — e.g. `list_ltv_customers` or
  `get_cohort_analysis` returning something like `{"result": "empty",
  "message": "Processing not yet completed"}` because a pipeline job
  hasn't run yet. Translate to one plain sentence: what isn't ready yet,
  and what the founder should do about it (usually: check back later, or
  ask for a metric from a tool that doesn't depend on that job). Don't
  build a Pulse Card around an empty result — say so in plain text instead.
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

*(Content rules below apply everywhere, including `cmo-health-check`; only
the *rendering* mechanics differ there — see the documented exception
above.)*

- **Headline states the fact AND the mechanism**, not just the fact.
  - Bad: "Sales are up 12% this week."
  - Good: "Sales up 12% — but the gain is coming from discounting, not loyalty."
- **A "Do this" is mandatory in every reply that surfaces a finding.**
  Names a specific lever and, where possible, a target number. Never
  generic ("consider looking into...").
  - Bad: "You may want to review your discounting strategy."
  - Good: "Do this: cap repeat-customer discounts back toward ~14% — where
    they sat last month — before this becomes their new price expectation."
- **Chat reply is two lines only.** No metrics table, no "Go deeper"
  line in your own text — both live on the Pulse Card. Your reply is the
  headline, the "Do this," and (once the card is published or updated) one
  short sentence noting it.
- Verdict/state maps to whether the finding is good/watch/bad — never to
  metric category. Don't invent confidence the data doesn't support; if
  the underlying numbers are noisy or an artifact (e.g. a one-order week),
  say so in the headline and use "watch," not a falsely confident "good"
  or "bad."
- Optimize for time-poor, mobile-first reading: two short lines in chat,
  one action, not a list of options — the card carries the rest.

---

## Your reply shape

```
{STATE_EMOJI} **{HEADLINE_FACT} — {HEADLINE_MECHANISM}**

💡 **Do this:** {ONE_SPECIFIC_PRESCRIPTIVE_ACTION_WITH_TARGET_NUMBER}
```

Then publish or update the Pulse Card per `cmogpt:cmo-pulse-card`, and add
one short sentence noting it (e.g. "Card updated above.") — do not paste
the artifact URL unless the founder asks for it.

`{STATE_EMOJI}` is 🟢 good / 🟡 watch / 🔴 bad, matching the overall read of
the finding, and matches the `tone` you set on the Pulse Card's `verdict`
or `priority` field — the two should never disagree.

**Example — good:**

```
🟡 **Sales up 12% — but the gain is coming from discounting, not loyalty.**

💡 **Do this:** Cap repeat-customer discounts back toward ~14% — where they sat last month — before this becomes their new price expectation.

Card updated above with the full breakdown and follow-up questions.
```

(The Pulse Card behind this carries `go_deeper`: "How is repeat discounting
trending?", "What's driving low conversion?", "Show my LTV by segment" —
as clipboard-copy chips on the card, not repeated here. The `get_diagnosis`
call also carried the same three as its own `go_deeper` argument, for the
bonus native card.)

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
clear one), the metrics table duplicates the Pulse Card, it closes with an
open question instead of a recommendation, and it never actually gets to
a "Do this" — and no Pulse Card was built at all.

---

## Updating this file

This file owns the *reasoning* rules (field quality, what "Do this" must
contain, when the format does and doesn't apply). The *rendering* — the
card's markup, tokens, and publish/update mechanics — lives entirely in
`cmogpt:cmo-pulse-card`. A visual change (new field, different layout,
different Go Deeper mechanism) is a `cmo-pulse-card`-only edit; it should
almost never require touching this file. A reasoning change (what counts
as a valid "Do this," when the two-line shape doesn't apply) is a
this-file-only edit; it should almost never require touching
`cmo-pulse-card`. If a change seems to need both, that's worth a second
look — it usually means the two are tangled somewhere they shouldn't be.

Do **not** touch the 11 standard-shape terminal skills for a change here —
they all defer to this file, so this is the only edit needed for them.
`cmo-health-check` reads a related but separate content spec in
`cmogpt:cmo-health-check-card-design` — update that file too if the
underlying content principle changes; both it and this file defer to
`cmogpt:cmo-pulse-card` for rendering.

If a second skill ever needs its own content exception the way
`cmo-health-check` does, add it as its own subsection under "When this
applies," with the same explicit reasoning (why the standard shape doesn't
fit) — don't let exceptions accumulate silently.
