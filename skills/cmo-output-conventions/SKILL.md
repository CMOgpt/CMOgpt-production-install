---
name: output-conventions
description: >
  Shared output-format and rendering-mode contract for every CMOgpt skill
  that presents a finding, diagnosis, or recommendation to the founder.
  Read this before presenting any reply that surfaces a finding — a
  metric read, a diagnosis, a recommendation, or a next step. Does not
  apply to raw data lists, open-ended discussion, or conceptual
  explanations, and has one documented exception for the weekly health
  digest (see "When this applies" below). Also defines a separate,
  always-applies rule against pasting raw tool output (JSON, error
  payloads, empty-result envelopes) into any reply — see "Never surface
  raw tool output" below. Centralizing this here means a rendering-mode or
  format change is a single-file edit instead of a 12-skill edit.
---

# CMOgpt — Output Conventions

This file is the single source of truth for *how* a CMOgpt reply is shaped
and rendered. Terminal skills (health check, diagnose-metrics, LTV, cohort,
budget, etc.) own *what* gets said — which metric leads, which verdict
applies, which threshold matters. This file owns *how it's packaged.*

---

## Current rendering mode

**`RENDERING_MODE: MARKDOWN_CARD`**

Update this line — and only this line, in most cases — when the rendering
picture changes. Everything below in "Which output do I emit" reads this
flag. Do not hardcode a rendering assumption inside any terminal skill;
they all defer to this file.

---

## When this applies

Use the format below for any reply that surfaces a finding: a metric read,
a diagnosis, a recommendation, a next-step suggestion.

Do **not** force this shape onto:
- A raw data list (recent orders, customer list, LTV decile table) — present
  as a plain list/table, no headline-mechanism framing needed.
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
2-4 metric card, it has its **own dedicated card design**, defined in
`cmogpt:cmo-health-check-card-design`.

Read that file — not this one — whenever presenting a health check result.
It covers: verdict badge, headline sales figure, six-metric grid, priority
issue callout, prescription, and go-deeper prompts, in both the plain-text
and (reserved) HTML forms, following the same `RENDERING_MODE` flag defined
here.

This is a deliberate, decided exception — not drift. If any other skill
starts to feel like it needs a similarly wider or dedicated format, don't
quietly copy `cmo-health-check`'s shape; give it its own design file the
same way, and add a one-line pointer to it here, so this file stays the
actual index of what's standard vs. what has its own dedicated design.

---

## Never surface raw tool output

This rule is not gated by "When this applies" above — it applies to every
reply, finding or not, in every rendering mode. A founder reads plain
sentences, not JSON. A tool's raw response — an error object, a status
payload, an empty-result envelope — is an implementation detail for you to
interpret, never text to paste into a reply.

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

## Field rules (apply regardless of rendering mode)

*(Does not apply to `cmo-health-check` — see documented exception above.)*

- **Headline states the fact AND the mechanism**, not just the fact.
  - Bad: "Sales are up 12% this week."
  - Good: "Sales up 12% — but the gain is coming from discounting, not loyalty."
- **2–4 supporting metrics max.** If more than 4 metrics matter, the lead
  driver hasn't been found yet — diagnose further before replying, don't
  list everything.
- **A "Do this" line is mandatory in every reply that surfaces a finding.**
  Distinct from the metrics. Names a specific lever and, where possible, a
  target number. Never generic ("consider looking into...").
  - Bad: "You may want to review your discounting strategy."
  - Good: "Do this: cap repeat-customer discounts back toward ~14% — where
    they sat last month — before this becomes their new price expectation."
- **"Go deeper" is secondary** — offered after the recommendation, never
  instead of it.
- State-color / emphasis (red/amber/green or bold/plain) maps to whether a
  number is bad/watch/good — never to metric category.
- Optimize for time-poor, mobile-first reading: front-load the headline,
  keep metrics scannable, one action, not a list of options.

---

## Which output do I emit

Check `RENDERING_MODE` above. This section applies to every skill using
the standard card shape. `cmo-health-check` reads the same flag but
applies it to its own dedicated shape defined in
`cmogpt:cmo-health-check-card-design` (not yet updated to the Markdown
card shape as of this change — tracked separately) — the mode below still
governs *which shape that dedicated design renders in*, it just doesn't
reshape into headline+metrics+do-this.

**`MARKDOWN_CARD`** *(current default)* → emit the Markdown card shape
below. This is plain CommonMark/GFM — bold, a table, a blockquote, emoji —
not raw HTML and not a custom fenced block a host has to specially parse
to render. It renders as a visually distinct card (bordered table,
highlighted blockquote) on every surface CMOgpt runs on today — claude.ai
web, Claude Desktop, Claude Code — because rendering standard Markdown in
an assistant reply is core, always-on behavior on those surfaces, not a
feature that needs confirming the way an MCP Apps `ui://` binding does.
(Assessed 2026-09-09; not separately lab-verified via
`ui-visualization-test` because that test targets `ui://` resource
rendering specifically, a different code path from how this skill's own
reply text renders.)

**`PLAIN_TEXT_DEFAULT`** *(legacy fallback)* → emit the flat plain-text
shape with no table or blockquote formatting. Fall back to this only if a
specific surface is ever confirmed to flatten or strip Markdown — not
observed on any CMOgpt surface so far.

**`HTML_CARD_CONFIRMED`** *(not active — reserved for when a confirmed
MCP Apps `ui://` resource binding is live)* → emit the HTML card shape
instead, on the specific surface(s) that binding covers. The most recent
`ui-visualization-test` run (2026-09-09) did not produce a valid PASS — the
test tool itself returned a malformed response rather than a `ui://`
resource, so it isn't evidence either way — this flag stays reserved
regardless. Until it is flipped, do not emit raw HTML in a normal chat
reply — it will not render as a card; it will show as plain or escaped
text, which is worse than the Markdown card below.

---

### Primary output: Markdown card

```
**{HEADLINE_FACT} — {HEADLINE_MECHANISM}**

| Metric | Reading |
|---|---|
| {METRIC_LABEL} | {METRIC_VALUE} ({METRIC_DELTA}) |
| {METRIC_LABEL} | {METRIC_VALUE} ({METRIC_DELTA}) |
<!-- 2-4 rows total -->

> 💡 **Do this:** {ONE_SPECIFIC_PRESCRIPTIVE_ACTION_WITH_TARGET_NUMBER}

**Go deeper:** {FOLLOWUP_QUESTION_1} · {FOLLOWUP_QUESTION_2} · {FOLLOWUP_QUESTION_3}
```

Prefix the headline with a single state emoji — 🟢 good / 🟡 watch / 🔴
bad — matching the overall read, same rule as before: state maps to
whether the finding is good/watch/bad, never to metric category. Only mark
an individual metric row the same way if its own state is clear from the
data; leave a row unmarked rather than guess when the underlying numbers
are themselves noisy or an artifact (e.g. a one-order week) — don't invent
confidence the data doesn't support.

**Example — good:**

```
🟡 **Sales up 12% — but the gain is coming from discounting, not loyalty.**

| Metric | Reading |
|---|---|
| Repeat share | -15% vs last month |
| Repeat discount depth | +31% |
| Conversion rate | 1.12% (below 1.8–4.5% benchmark) |

> 💡 **Do this:** Cap repeat-customer discounts back toward ~14% — where they sat last month — before this becomes their new price expectation.

**Go deeper:** How is repeat discounting trending? · What's driving low conversion? · Show my LTV by segment
```

Keep "Go deeper" as literal questions the founder can type back — no
click handler in a plain chat turn; there's no host to receive a click.

### Secondary output: plain text (fallback)

Use only when `RENDERING_MODE` above is `PLAIN_TEXT_DEFAULT`.

```
{HEADLINE_FACT} — {HEADLINE_MECHANISM}

{METRIC_LABEL}: {METRIC_VALUE} ({METRIC_DELTA})
{METRIC_LABEL}: {METRIC_VALUE} ({METRIC_DELTA})
{METRIC_LABEL}: {METRIC_VALUE} ({METRIC_DELTA})

**Do this:** {ONE_SPECIFIC_PRESCRIPTIVE_ACTION_WITH_TARGET_NUMBER}

Go deeper: {FOLLOWUP_QUESTION_1} · {FOLLOWUP_QUESTION_2} · {FOLLOWUP_QUESTION_3}
```

**Example — good:**

```
Sales up 12% — but the gain is coming from discounting, not loyalty.

Repeat share: -15% vs last month
Repeat discount depth: +31%
Conversion rate: 1.12% (below 1.8-4.5% benchmark)

**Do this:** Cap repeat-customer discounts back toward ~14% — where they sat last month — before this becomes their new price expectation.

Go deeper: How is repeat discounting trending? · What's driving low conversion? · Show my LTV by segment
```

Keep "Go deeper" as literal questions the founder can type back — no
`sendPrompt()`-style click handler in a plain chat turn; there's no host to
receive the click.

### Tertiary output: HTML card (reserved — see rendering mode above)

Use only when `RENDERING_MODE` above is `HTML_CARD_CONFIRMED` for the
current surface — a surface you control end-to-end, or a production
surface with a verified MCP Apps `ui://` resource binding (not just
emitting HTML text in a normal reply).

```html
<div style="background:var(--surface-2);border-radius:12px;border:0.5px solid var(--border);padding:0.875rem 1rem">
  <div style="display:flex;align-items:center;gap:8px;margin-bottom:10px">
    <div style="width:32px;height:32px;border-radius:50%;background:var(--bg-{STATE_COLOR});display:flex;align-items:center;justify-content:center">
      <i class="ti ti-{ICON}" style="color:var(--text-{STATE_COLOR})"></i>
    </div>
    <div style="font-size:14px;font-weight:500">{HEADLINE_FACT} — {HEADLINE_MECHANISM}</div>
  </div>

  <div style="display:flex;gap:8px;overflow-x:auto;margin-bottom:10px">
    <!-- repeat per metric, 2-4 total -->
    <div style="background:var(--surface-1);border-radius:999px;padding:6px 12px;font-size:12px;display:flex;align-items:center;gap:6px;white-space:nowrap">
      <i class="ti ti-{METRIC_ICON}" style="color:var(--text-{METRIC_STATE})"></i>
      {METRIC_LABEL} {METRIC_DELTA}
    </div>
  </div>

  <div style="background:var(--bg-accent);border-radius:var(--radius);padding:10px 12px;display:flex;gap:8px;align-items:flex-start">
    <i class="ti ti-bulb" style="color:var(--text-accent)"></i>
    <div style="font-size:13px"><strong>Do this:</strong> {ONE_SPECIFIC_PRESCRIPTIVE_ACTION_WITH_TARGET_NUMBER}</div>
  </div>

  <div style="margin-top:8px;font-size:12px;color:var(--text-muted)">
    Go deeper:
    <!-- repeat per follow-up, 2-3 total -->
    <button onclick="sendPrompt('{FOLLOWUP_PROMPT}')" style="font-size:12px;padding:5px 10px;border-radius:999px;background:var(--surface-1);border:0.5px solid var(--border)">{FOLLOWUP_LABEL}</button>
  </div>
</div>
```

No signup/CTA line on production surfaces — the founder's store is already
connected. `[OPEN: confirm whether any footer replaces it once HTML_CARD
mode is active — e.g. nothing, or a link to a paid-tier feature.]`

---

### Example — bad (reject this shape in any rendering mode)

```
This week your sales performance shows some interesting trends. Revenue increased
by 12% compared to last week, which is a positive sign. However, when we look
deeper at the data, we can see that this increase is partially driven by higher
discount rates among repeat customers...

[several more paragraphs of narrative]

Would you like me to look into your conversion rate further, or would you prefer
to see more detail on your LTV segments?
```

No specific action, no target number, closes with an open question instead
of a recommendation.

---

## Updating this file

When the rendering picture changes (e.g. a specific surface is confirmed to
strip Markdown, dropping back to `PLAIN_TEXT_DEFAULT`; or an MCP Apps
`ui://` binding is confirmed reliable in production, moving to
`HTML_CARD_CONFIRMED` — see `ui-visualization-test` for how to check that):

1. Flip `RENDERING_MODE` at the top of this file.
2. If moving to `HTML_CARD_CONFIRMED`, update field values in the HTML
   template if the design system changed.
3. Do **not** touch the 11 standard-shape terminal skills — they all defer
   here, so this is the only edit needed for them.
4. `cmo-health-check` reads this same flag but renders its own dedicated
   design — its templates live in `cmogpt:cmo-health-check-card-design`,
   not here. As of this change, that file has not yet been updated to add
   a Markdown card shape (still plain-text default there) — update it
   separately to bring health check in line, then keep its HTML template
   current too whenever the design system's field values change.

If a second skill ever needs its own exception the way `cmo-health-check`
does, add it as its own subsection under "When this applies," with the
same explicit reasoning (why the standard shape doesn't fit) — don't let
exceptions accumulate silently.
