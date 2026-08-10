---
name: ui-visualization-test
description: >
  Periodic, low-stakes test of whether MCP Apps `ui://` resource rendering
  is working in the current surface (claude.ai web, Claude app, Claude
  Desktop). Trigger when the founder or developer asks to "test UI
  rendering," "check if the visual card works," "run the visualization
  test," "test MCP apps," or similar. Not a business skill — produces no
  store insight, only a pass/fail signal on whether the host renders the
  card.
---

# CMOgpt — UI Visualization Test

## Purpose

There's no changelog entry that reliably announces "third-party MCP Apps
`ui://` rendering now works." The only trustworthy signal is empirical:
call a minimal test tool and look at what actually shows up. This skill
runs that check on demand so it can be repeated every few weeks without
reconstructing the procedure each time.

## Procedure

1. Call `CMOgpt:ui_render_test` (the dedicated dummy tool — see
   `ui-render-test-tool-spec.md` for what it returns; if this tool isn't
   installed yet, stop and tell the founder it needs to be added to the
   MCP server first).
2. Look at exactly what appears in the response, and classify it as one of:

   - **PASS** — a styled card renders inline: verdict badge, metric grid,
     priority issue callout, prescription callout, all visually formatted
     per the design in `cmogpt:cmo-health-check-card-design`. This means
     the `ui://` resource actually rendered in an iframe.
   - **FAIL — fallback placeholder** — instead of the card, a plain-text
     message appears along the lines of "this tool call rendered an
     interactive widget in the chat" with no visible card. This is the
     exact failure mode reported in the open MCP Apps rendering bugs
     (`modelcontextprotocol/ext-apps#671`, `anthropics/claude-ai-mcp#61`)
     — the protocol exchange succeeded but the host didn't render the
     iframe.
   - **FAIL — tool error** — the tool call itself errors out (auth,
     connector not installed, tool not found). Different failure mode from
     the above — this means the test tool isn't reachable at all, not that
     rendering specifically failed.

3. Report the result in this exact shape, so results are comparable across
   test runs over time:

```
UI rendering test — {DATE} — {SURFACE: claude.ai web / Claude app / Claude Desktop}

Result: {PASS | FAIL - fallback placeholder | FAIL - tool error}

{One or two sentences on exactly what appeared, in plain language.}

{If FAIL: nothing further needed from you — this is expected until Anthropic
resolves the underlying rendering issue. Re-test in a few weeks.}
{If PASS: this is a real, meaningful change — flag it to the team before
enabling HTML_CARD_CONFIRMED in output-conventions, since a single pass
on one surface doesn't confirm it holds across all surfaces.}
```

4. Do not draw conclusions beyond the single test run. One PASS on
   claude.ai web doesn't mean Claude Desktop or the mobile app also
   render correctly — if the founder wants full coverage, offer to repeat
   the same procedure and ask which surface they're testing on, since
   there's no way for this skill to detect that automatically.

## What this skill does not do

- It does not modify `RENDERING_MODE` in `output-conventions` — that's a
  deliberate, manual decision the team makes after enough consistent PASS
  results, not something this test should trigger automatically.
- It does not test any business logic, metric accuracy, or diagnosis
  quality — the dummy data in the test tool is fixed and fictional
  precisely so this stays a pure rendering check, decoupled from whether
  the analytics pipeline is working.
- It is not a substitute for checking the upstream bug threads directly —
  those threads may explain *why* a test failed even when this skill can
  only tell you *that* it failed.

## Suggested cadence

Every 2-4 weeks, or immediately after any Anthropic announcement that
touches MCP Apps, connectors, or Claude's rendering pipeline. Re-testing
more often than that is unlikely to produce new information, since this is
an infrastructure fix on Anthropic's side, not something that changes
based on your own usage.
