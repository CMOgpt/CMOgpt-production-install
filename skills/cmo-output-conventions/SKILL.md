---
name: output-conventions
description: >
  Shared output-format and rendering-mode contract for every CMOgpt skill
  that presents a finding, diagnosis, or recommendation to the founder.
  Read this before presenting any reply that surfaces a finding — a
  metric read, a diagnosis, a recommendation, or a next step. Does not
  apply to raw data lists, open-ended discussion, or conceptual
  explanations, and has one documented exception for the weekly health
  digest (see "When this applies" below). Centralizing this here means a
  rendering-mode or format change is a single-file edit instead of a
  12-skill edit.
---

# CMOgpt — Output Conventions

This file is the single source of truth for *how* a CMOgpt reply is shaped
and rendered. Terminal skills (health check, diagnose-metrics, LTV, cohort,
budget, etc.) own *what* gets said — which metric leads, which verdict
applies, which threshold matters. This file owns *how it's packaged.*

---

## Current rendering mode

**`RENDERING_MODE: PLAIN_TEXT_DEFAULT`**

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
`cmogpt:cmo-health-check-card-design` — the plain-text/HTML distinction
below still governs *whether that dedicated design renders as styled HTML
or plain text*, it just doesn't reshape into headline+metrics+do-this.

**`PLAIN_TEXT_DEFAULT`** → always emit the plain-text shape below. This is
correct on every surface CMOgpt runs on today — native chat (desktop,
browser, mobile), Claude Code, Cowork. It doesn't depend on the host
rendering HTML, so it's the safe default when you're unsure which surface
you're on.

**`HTML_CARD_CONFIRMED`** *(not active yet — reserved for when a confirmed
MCP Apps `ui://` resource binding is live)* → emit the HTML card shape
instead, on the specific surface(s) that binding covers. Until this flag is
flipped, do not emit raw HTML in a normal chat reply — it will not render
as a card; it will show as plain or escaped text, which is worse than the
plain-text template below.

---

### Primary output: plain text

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

### Secondary output: HTML card (reserved — see rendering mode above)

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

### Example — bad (reject this shape in either rendering mode)

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

When the rendering picture changes (e.g. an MCP Apps `ui://` binding is
confirmed reliable in production — see cmo-primary for what "confirmed"
means and how to test it):

1. Flip `RENDERING_MODE` at the top of this file.
2. Update field values in the HTML template if the design system changed.
3. Do **not** touch the 11 standard-shape terminal skills — they all defer
   here, so this is the only edit needed for them.
4. `cmo-health-check` reads this same flag but renders its own dedicated
   design — its HTML template lives in `cmogpt:cmo-health-check-card-design`,
   not here, so update that file's HTML block too if the design system's
   field values changed. Flipping the flag here still activates it; you
   just have two HTML templates to keep current instead of one.

If a second skill ever needs its own exception the way `cmo-health-check`
does, add it as its own subsection under "When this applies," with the
same explicit reasoning (why the standard shape doesn't fit) — don't let
exceptions accumulate silently.
