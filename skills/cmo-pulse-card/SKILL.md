---
name: cmo-pulse-card
description: >
  Shared visual card component every CMOgpt reply renders or refreshes
  through — one template that scales from a thin no-finding touch, up
  through a single-metric diagnosis (hero only), up to the six-metric
  weekly health check. Read this whenever `cmogpt:cmo-output-conventions`
  or `cmogpt:cmo-health-check-card-design` says to render or refresh a
  reply — it holds the template file and the fill-and-publish procedure.
  Not a skill founders trigger by name. This includes a reply that only
  called one CMOgpt tool ad hoc (e.g. `about_my_account`, `get_diagnosis`)
  with no other CMOgpt skill invoked first — that tool call still owes the
  store's Pulse Card a render or refresh; there is no CMOgpt tool call
  this file doesn't apply to.
---

# CMOgpt — Pulse Card (shared component)

## What this is

Every CMOgpt reply — a finding, a raw data list, an open-ended discussion,
a conceptual explanation, or the full weekly digest — renders or refreshes
through this one card: verdict, headline metric, 0–6 supporting metrics,
the priority issue, one prescribed action, and up to four Go Deeper
follow-ups. `cmogpt:cmo-output-conventions`'s "When this applies" has no
exceptions — what changes per reply type is how much of this actually gets
filled in, not whether the card gets touched. The shape never changes;
only how many supporting metrics section 02 holds does, and section 02
drops out entirely below 2 (see "Fixed structure"). For a non-finding
reply, sections 02–05 can be thin or omitted — see
`cmogpt:cmo-output-conventions`'s "Non-finding replies."

`cmogpt:cmo-output-conventions` calls this with 0–1 supporting metrics for
a standard single-finding reply. `cmogpt:cmo-health-check-card-design`
calls this with up to 6 for the weekly digest. Neither of those files owns
the rendering — this one does, so a visual change is a one-file edit
instead of touching every terminal skill.

## Fixed structure

1. **Verdict** — store name, period, a status pill (good/watch/bad), and
   the hero number: the finding's headline metric, with its delta.
2. **Metrics** — 0 to 6 supporting tiles. **Omit this section entirely at
   0 or 1** — a single metric belongs in the hero, never duplicated into a
   lone tile. Use 2–6 only when there are genuinely that many supporting
   figures worth a glance.
3. **Priority** — the mechanism: what's actually driving the headline
   number, in plain language, with the real figures inline.
4. **Do this** — one prescriptive action with a target number. Same rule
   as before: never generic ("consider reviewing...").
5. **Go deeper** — up to 4 follow-up questions as clickable chips. See
   "Why Go Deeper copies instead of asking" below for what clicking one
   actually does.

## Procedure

1. **Decide the content in the calling skill, not here.** This file only
   renders. Follow the domain skill's own reasoning heuristics (e.g.
   `cmo-contribution-margin`'s new-vs-repeat logic, `cmo-health-check`'s
   six-indicator system read) to decide what leads, the verdict, and the
   prescription.

2. **Assemble one JSON object:**

   ```json
   {
     "store_name": "MAKU STUDIOS",
     "period_label": "Week ending Sept 14, 2026",
     "freshness_note": "Data current to Sept 14, 2026 — optional, omit if not stale",
     "verdict": { "label": "Growing profitably", "tone": "good" },
     "hero": {
       "value": "$59,231",
       "delta": "▲ 37.8% vs last week ($42,970)",
       "delta_tone": "up",
       "label": "Total sales, 7 days"
     },
     "tiles": [
       { "label": "AOV", "value": "$162.19", "delta": "▲ 15.8%", "tone": "up" }
     ],
     "priority": {
       "tone": "watch",
       "icon": "&#9873;",
       "html": "<b>Contribution margin</b> is the one metric under pressure — driven almost entirely by a single order."
     },
     "do_this": {
       "icon": "&#9673;",
       "html": "Hold budget flat at <b>$1,290/day</b> — confirm Sept 14 was a genuine one-order day before reacting."
     },
     "go_deeper": [
       "What's driving the 23% traffic surge this week?",
       "How is repeat ratio trending over 30 days?"
     ]
   }
   ```

   `tiles` is 0–6 entries; omit the key or pass `[]` for a hero-only card.
   `go_deeper` is 0–4 entries — the same follow-ups you'd otherwise pass
   through a tool call's own `go_deeper` argument; write them the same way
   (things the founder would actually ask next, not category labels).

3. **Fill the template.** Read
   `${CLAUDE_PLUGIN_ROOT}/skills/cmo-pulse-card/references/pulse-card-template.html`,
   copy it to a working file, and replace the single line
   `{{CARD_DATA}}` inside the `<script type="application/json"
   id="pulse-card-data">` block with the real JSON object from step 2
   (use a real JSON serializer — never hand-format the object as a
   string, and never leave `{{CARD_DATA}}` unreplaced).

4. **Publish it — update in place, never spawn a new one per finding.**
   - Title every card `<store_name> — Pulse Card` (stable across updates
     so this lookup keeps working).
   - Before the *first* publish in a conversation: call `Artifact` with
     `action: "list"` and look for an artifact already titled
     `<store_name> — Pulse Card`. If one exists, `read` it and republish
     to its `url`.
   - If this conversation already published one earlier in the same
     session, republish to that same `url` directly — do not look it up
     again.
   - Only publish without a `url` when neither of the above finds one —
     that is the one case where a new artifact gets created.
   - `favicon: "📈"` only on a genuinely first-ever publish for that
     store; omit it on every update.
   - Load `artifact-design` before the very first publish for a session,
     same as any other artifact.

5. **Chat reply stays short — do not drop it in favor of the card.**
   Write the normal two-line reply from `cmogpt:cmo-output-conventions`
   (headline + mechanism, then "Do this"), or the health-check verdict +
   priority line from its own skill. The card is the durable, shareable,
   revisit-able record; the two lines are the fast read for someone
   scanning chat history who never opens the side panel. After publishing,
   add one short sentence noting the card was created or updated — do not
   restate the metric grid or the go-deeper questions in chat text, those
   live only on the card.

## Field rules

- `delta_tone` / tile `tone`: `"up"` | `"down"` | `"flat"` — favorable
  direction is metric-specific (falling discount % is `"up"`/green;
  falling sales is `"down"`/red). Judge per metric, never a fixed
  up-is-green rule.
- `verdict.tone` / `priority.tone`: `"good"` | `"watch"` | `"bad"` — never
  invent confidence the data doesn't support. If the underlying numbers
  are noisy or a small-sample artifact, say so in `priority.html` and use
  `"watch"`, not `"bad"`.
- `do_this.html` must name a specific lever and a target number.
- `priority.html` and `do_this.html` render as HTML, not escaped text —
  you write these, so `<b>` around key figures is fine and expected; never
  pass founder-editable or tool-raw text through unescaped.

## Why Go Deeper copies instead of asking

A hosted Artifact page cannot post into the live conversation — that's a
platform boundary, not a bug to work around. Copy-to-clipboard is the
closest real equivalent: one click copies the question, the founder
pastes it into chat and sends it themselves, and it becomes a genuine new
turn with full memory once they do.

The thing that *does* post directly into the conversation today is the
connector's own native `ui://` card — the one rendered straight from a
tool call's `go_deeper` argument, outside this template entirely (see
`cmogpt:cmo-output-conventions`'s "Data and Go deeper" section for how
that still works on single tool calls that don't go through this card).
If that native mechanism is ever extended to carry this card's full
shape, this section should be rebuilt around it instead of
clipboard-copy — and this paragraph deleted, not left stale.

## Do not

- Do not publish a new Pulse Card artifact per finding — always update
  the one per store (step 4).
- Do not skip this for raw data lists, open-ended discussion, or
  conceptual explanations — `cmogpt:cmo-output-conventions`'s "When this
  applies" has no exceptions. Build the thinnest honest version (per its
  "Non-finding replies" section) instead of skipping the card.
- Do not put a lone metric into a 1-tile section 02 — fold it into the
  hero and omit section 02 entirely.
- Do not hand-write the card's HTML from scratch — always start from
  `references/pulse-card-template.html` so every card stays pixel-
  identical across skills.
