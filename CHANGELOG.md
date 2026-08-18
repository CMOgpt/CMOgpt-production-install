# CMOgpt Connector — Changelog

All notable changes to the **CMOgpt Connector** plugin are documented here.

---

## [1.44] — 2026-08-18

### Manifest — missing skills and tool registration

- `plugin.json` — added 6 skills that existed on disk but were not declared
  in the `skills` list: `cmo-adjust-marketing-spend`,
  `cmo-contribution-margin`, `cmo-get-diagnosis`,
  `cmo-health-check-card-design`, `cmo-output-conventions`,
  `ui-visualization-test`. This was the root cause of
  `ui-visualization-test` not being found at runtime — the skill folder
  existed but was never loaded into the plugin
- `plugin.json` — added `ui_render_test` to the `tools` list (dummy
  diagnostic tool used only by `ui-visualization-test` to verify MCP Apps
  `ui://` rendering); backing implementation added to the MCP server tool
  DB

### Manifest

- Bumped `version` to `1.44`
- `cmo-version` — updated to report `1.44`, build `2608181500`, release
  date `2026-08-18`

---

## [1.43] — 2026-08-09

### new feature - Visual cards 

- `cmo-primary` — added visual card, summarize findings, concise answers 

---

## [1.42] — 2026-08-07

### Skills — `cmo-primary` visual output overhaul

- Added a "Business topics" rule: after any reply that surfaces a finding
  (not a plain data lookup), close with exactly one next-step suggestion
  grounded in a metric/domain not yet explored this session; don't repeat a
  domain already offered and declined
- Added a "Visual output convention" section describing the host's compact
  visual cards (KPI/metric tiles, ranked step lists, clickable prompt
  chips) and the prose rules around them: concise/mobile-first/scannable
  replies, a 1-2 sentence prose lead-in with no numbers, 2-4 metric cards
  colored by state (not category) for the "Why", ranked step lists instead
  of numbered paragraphs for multi-action plans, chips/buttons instead of
  numbered prose menus, and a 2-4 sentence prose budget before the CTA
- Added the `cmo-card` fenced-block output format as the mandatory default
  shape for every finding-bearing reply on the production plugin: a single
  JSON object per reply with `headline`, `state`, up to 4 `metrics`,
  `doThis`, and up to 3 `goDeeper` chip prompts. Documented field rules
  (headline states fact + mechanism, not just the fact; `doThis` must name
  a specific lever and target number; plain prose stays for the lead-in and
  single-action replies with no metrics to show)
- Removed the "What you are not" boundary section (dashboard/reporting-tool
  disclaimer), superseded by the new card-based output convention

### Manifest

- Bumped `version` to `1.42`
- `cmo-version` — updated to report `1.42`, build `260807`, release date
  `2026-08-07`

---

## [1.41] — 2026-08-03

### Bug fixes — tool-name drift against `plugin.json`

- `cmo-primary` — corrected `get_ltv_segment` (singular) → `get_ltv_segments` (plural),
  matching the tool declared in `plugin.json` and the live SP

## [1.4] — 2026-07-24

### Bug fixes — tool-call parameter drift (root cause of `list_ltv_customers` failing/looping on claude.ai)

- Removed `job_id` from every example tool call across 8 skills (`cmo-diagnose-metrics`,
  `cmo-health-check`, `cmo-build-profit`, `cmo-optimize-marketing-budget`,
  `cmo-set-marketing-budget`, `cmo-explain-marketing-advertising`,
  `cmo-diagnose-contribution-margin`, `cmo-router`) — `job_id` is resolved server-side and
  is not a real parameter on any of these tools, so instructing the model to pass it
  produced malformed calls
- `get_diagnosis` — corrected documented params from `day_span`/`focus_metric` to the real
  schema `date_range`/`metrics_code`
- `cmo-primary` — fixed wrong param names against live tool schemas: `upsert_marketing_spend`
  (`platform_name` → `platform_code`), `upsert_target` (`target_name`/`target_value` →
  `metrics_name`/`metrics_value`), `get_metrics_manifest` (`metrics_domain` →
  `business_domain`)
- `cmo-primary` — removed fully-filled-in example tool calls (e.g.
  `list_ltv_customers(segment="lapsed", ...)`) that the model was pattern-matching and
  echoing back literally instead of asking the founder; replaced with valid-literal-set
  statements only, plus explicit no-paraphrasing rules (no "ascending"/"descending" for
  `list_order`, no title-case for `segment` — the underlying SP requires exact lowercase
  `active/lapsed/dormant/churned` and exact `ASC`/`DESC`)
- `cmo-primary` — added a new top-of-file rule: every tool call must carry a real
  `skill_name`. Root cause of the `list_ltv_customers` bug was tool calls issued with
  `skill_name = "none"` when the founder's wording (e.g. literally typing a tool name)
  didn't match any `cmo-router` trigger — none of the ask-before-calling/valid-value
  guardrails live outside skill content, so an unattributed call bypassed all of them
  regardless of wording. Added reverse-mapping requirement: resolve to the skill that
  documents the requested tool when phrasing doesn't match the router's intent table
- `cmo-primary` — added explicit "never guess, never scan" rule: model had been trying
  multiple parameter combinations (`segment="all"`, decile as string vs int,
  `is_break_even` as `"true"/"false"`, and reusing `get_ltv_segment`'s time-window labels
  as `list_ltv_customers` segment values) instead of asking the founder once for a valid
  literal. Confirmed via direct SP test that clean literal params return data — the empty
  results were a calling-behavior bug, not a data/environment issue
- De-duplicated the resulting guardrail text: full valid-value lists now live in
  `cmo-analyze-customer-profitability`, with `cmo-primary` and the skill-routing map
  reduced to a short pointer plus the core rule, keeping `cmo-primary`'s net growth to
  ~+18% instead of +28%

### Skills — cleanup

- Deleted `cmo-diagnose-growth/` from the `skills/` folder. It was left on disk as a
  byte-identical duplicate of `cmo-build-profit` (differing only in the `name:` field)
  after the 1.3 changelog said it had been deleted — it never actually was. It was already
  absent from `plugin.json`'s `skills[]`, so this is a repo-cleanliness fix, not a
  behavior change

### Known issues — not fixed this release (tracked for Andrew, backend-side)

- Tool-name/param mismatches between skill docs and the live DB tool definitions
  (`dw_1pd_app.ref_claude_connector_definition`) are still being reconciled: `get_ltv_segment`
  vs `get_ltv_segments` naming, `list_ltv_customers` param spelling disagreeing between
  `cmo-primary` and `cmo-analyze-customer-profitability`, and `list_recent_orders`/
  `get_ltv_distribution` needing new SQL params before they can accept the arguments the
  skills already document. Decision this round: skill docs are the source of truth and the
  DB is being updated to match, rather than editing the skills again
- `get_segment_migration` — active production tool with no skill coverage; also missing a
  required `segment` param on its schema, so no skill-doc fix is possible until the schema
  is updated
- `get_trading_week` and `get_business_domain` — both instructed in `cmo-primary`, but the
  backing stored procedures may not be deployed (`1pd_SQL` has a `-- not built yet` comment
  on both). Needs confirmation before continuing to expose them as always-available tools

### Follow-up — 2026-07-25 (DB reconciliation completed; debug/demo re-synced from prod)

With the DB tool definitions now reconciled to match the skill docs (see above), this
pass closes out the remaining internal inconsistencies and propagates the fixed content
to `plugin-debug` and `plugin-demo`.

- `cmo-router` — was still stale against `plugin.json`'s own `skills[]`: routed to the
  retired `cmo-measure-customer-ltv` and was missing routes for `cmo-set-marketing-budget`,
  `cmo-explain-marketing-advertising`, `cmo-analyze-ltv`, `cmo-analyze-customer-profitability`,
  `cmo-inspect-customer`, `cmo-list-recent-orders`, `cmo-terminology`. Filled in the full
  routing table and "what CMOgpt can do" summary
- `cmo-primary` — added the "every tool call must carry a real `skill_name`" governance
  section, the `list_ltv_customers` no-placeholder-values guard, and the `date_range`
  (1–90 day) validation block — these existed only in the debug copy and had not been
  ported back
- Fixed broken cross-skill references: `/cmo-adjust-marketing-budget` →
  `/cmo-optimize-marketing-budget` (`cmo-set-marketing-budget`); `cmo-analyze-cohorts` →
  `cmo-analyze-cohort` and `cmo-analyze-segments` → `cmo-analyze-customer-profitability`
  (`cmo-analyze-ltv`, neither of which was ever a real skill name)
- `cmo-diagnose-metrics`, `cmo-build-profit` — `focus_metric`/`focus_domain` → `metrics_code`
  (matches the real param name; these two skills had not picked up the 1.4 rename)
- `cmo-inspect-customer` — removed a stray leading `--` line that broke the file's YAML
  frontmatter
- `cmo-analyze-customer-profitability` — front-matter `description:` block was not
  indented under the YAML `>` fold and had prose bleeding past the closing `---`,
  corrupting the frontmatter; rewritten with the corrected structure already used in the
  debug copy
- `cmo-list-recent-orders` — corrected stale `# cmo-recent-orders` heading to
  `# cmo-list-recent-orders`
- `cmo-analyze-cohort` — added `cohort_date` ISO-8601 normalization guidance (accept and
  silently convert common date formats instead of re-prompting)
- `cmo-terminology` — kept prod's `CUSTOMERS-DORMANT`/`CUSTOMERS-CHURNED` definitions
  (non-overlapping 180–275 / 275–365 day bands with an explicit "past 365 days = treated
  as lost" note) over the debug copy's vaguer version
- `plugin.json` — `tools[]` still declared `get_ltv_segment` (singular) and described
  `upsert_marketing_spend`/`upsert_target` params as `marketing_amt`/`metrics_name`/
  `metrics_value`, disagreeing with the skill docs. Renamed to `get_ltv_segments` and
  corrected the write-tool descriptions to `marketing_spend`/`target_name`/`target_value`,
  per the "skill docs are source of truth" decision above

### Downstream — plugin-debug and plugin-demo brought current

- `plugin-debug` fully regenerated from this reconciled prod content (mechanically
  stripped of `job_id` — debug hard-codes it server-side instead of resolving it from
  the logged-in user). Diff between prod and debug skills is now exactly the `job_id`
  handling difference, nothing else. `plugin.json` bumped to `1.4` to match
- `plugin-demo` had not been merged from prod in a long time and was missing 5 skills
  entirely (`cmo-analyze-cohort`, `cmo-analyze-customer-profitability`, `cmo-analyze-ltv`,
  `cmo-inspect-customer`, `cmo-update-marketing-spend-business-target`) — added, adapted
  to demo conventions (`_demo` tool suffixes, `log_demo_question` logging, sample-store
  framing, standard CTA). Also fixed several real bugs where demo skills called bare
  tool names missing the `_demo` suffix (`get_diagnosis`, `get_metric_history`,
  `get_marketing_budget` used without suffix in 4 skills) and reconciled param naming
  (`marketing_amt`/`metrics_name`/`metrics_value` → `marketing_spend`/`target_name`/
  `target_value`) now that the demo SP is a duplicate of prod's. `plugin.json` bumped to
  `1.4` to match. See `plugin-demo/CHANGELOG.md` for the full list

### Skills — new

- `cmo-version` — reports the installed plugin version, build, and release date, so it's
  possible to tell which build is active when multiple plugin versions may be installed.
  Previously existed on disk but was never usable: broken YAML frontmatter (UTF-8 BOM,
  `name:cmo-version` missing the required space) and absent from `plugin.json`'s
  `skills[]`. Rewritten with valid frontmatter, trimmed to just version/build/release-date
  (dropped the stale "new features"/"retired features" sections), and registered in
  `skills[]`

---

## [1.3] — 2026-07-17

### Manifest

- Bumped `version` to `1.3`
- `plugin.json` — `skills[]` was stale against the actual `skills/` folder: it still
  listed the retired `cmo-diagnose-growth` and `cmo-measure-customer-ltv`, and was
  missing four skills that already existed on disk (`cmo-build-profit`,
  `cmo-analyze-customer-profitability`, `cmo-analyze-ltv`, `cmo-inspect-customer`).
  Replaced the list with the 16 skills that actually exist; verified `skills[]` and
  the `skills/` folder now match exactly (no missing, no dangling entries)
- `capabilities.useCases` — updated the growth-diagnosis entry to reference
  `cmo-build-profit`; replaced the single LTV entry with three: customer-profitability
  concepts, LTV/payback playbook, and single-customer inspection

### Skills — new

- `cmo-analyze-ltv` — contribution LTV:CAC and payback playbook (P1-P5 journey,
  break-even reality, whale identification); built on the concepts defined in
  `cmo-analyze-customer-profitability`
- `cmo-analyze-customer-profitability` — concept/terminology anchor for customer
  profitability: RFM recency segmentation (Active/Lapsed/Dormant/Churned), LTV
  deciles/quartiles, and timing/history rules. Rewritten from `cmo-measure-customer-ltv`
- `cmo-inspect-customer` — single-customer drill-down: full order history rolling up
  to margin, CAC, and break-even, so the founder can trust the portfolio-level numbers

### Skills — resolved (byte-identical duplicate from 1.2)

- 1.2 shipped `cmo-build-profit` and `cmo-diagnose-growth` as byte-for-byte identical
  content and left the choice between them as an open "known issue" for Andrew.
  Andrew's ruling: **`cmo-build-profit` is the correct name** — it describes the
  skill's outcome, which takes priority over matching the `cmo-diagnose-*` sibling
  naming pattern. Deleted `cmo-diagnose-growth`; kept and renamed the surviving copy's
  `name:` field to `cmo-build-profit`; repointed every reference (`cmo-router` routing
  table + "do not use this skill if" list, `plugin.json`) accordingly

### Skills — retired

- `cmo-measure-customer-ltv` — split into `cmo-analyze-customer-profitability`
  (concepts) and `cmo-analyze-ltv` (playbook); see "Skills — new" above

### Bug fixes

- `cmo-inspect-customer` — file began with a stray `--` line before the real `---`
  frontmatter delimiter, so frontmatter wasn't at byte 0 and any strict skill loader
  would fail to parse `name`/`description`. Removed the stray line
- `cmo-analyze-customer-profitability` — frontmatter `description: >` block scalar's
  continuation lines were not indented, which swallowed the skill's body content into
  the YAML frontmatter and broke parsing entirely (confirmed via js-yaml: *"can not
  read a block mapping entry; a multiline key may not be an implicit key"*). Rewrote
  the frontmatter with a properly indented description and moved the body content
  below the closing `---`
- `cmo-analyze-ltv` and `cmo-analyze-customer-profitability` both referenced
  `cmo-analyze-cohorts` (plural — the real skill is singular, `cmo-analyze-cohort`).
  Fixed in both
- `cmo-analyze-ltv` referenced `cmo-analyze-segments`, a skill that never existed.
  Repointed to `cmo-analyze-customer-profitability` and reworded the boundary note to
  frame it as "definitions this skill assumes," matching the concepts → playbook
  relationship between the two skills
- `cmo-set-marketing-budget` referenced `/cmo-adjust-marketing-budget`, which doesn't
  exist (the real skill is `cmo-optimize-marketing-budget`). Fixed
- `cmo-list-recent-orders` had an internal `# cmo-recent-orders` heading that didn't
  match its own skill name. Fixed to `# cmo-list-recent-orders` (cosmetic)
- `cmo-router` — routing table and Step 5 "what can you do" summary had no entry for
  6 of the 16 real skills (`cmo-analyze-ltv`, `cmo-inspect-customer`,
  `cmo-list-recent-orders`, `cmo-set-marketing-budget`,
  `cmo-explain-marketing-advertising`, `cmo-terminology`,
  `cmo-analyze-customer-profitability`); a founder naming any of these at session
  start would fall through to the generic clarifying-question path instead of
  routing directly. Added rows for all of them
- `cmo-router` routed customer-LTV questions to `cmo-measure-customer-ltv`, a name
  that no longer exists. Repointed "what's my LTV to CAC" to `cmo-analyze-ltv` and
  "what does Active/Lapsed/Dormant/Churned mean" to `cmo-analyze-customer-profitability`

### Validation

- Added a validation pass (frontmatter parses as YAML; `name` present and matches the
  folder name; every `cmo-*` string referenced in a skill body resolves to a real
  skill folder) run across all 16 skills — all pass clean, and `plugin.json`'s
  `skills[]` was cross-checked against the actual `skills/` folder contents

---

## [1.2] — 2026-07-14

### Manifest

- Bumped `version` to `1.2`
- `plugin.json` — `skills[]` referenced `./skills/cmo-adjust-marketing-spend`, a path that
  no longer existed on disk (superseded by `cmo-optimize-marketing-budget`); this would
  have broken plugin load. Replaced it, and added the three genuinely new skills
  (`cmo-analyze-cohort`, `cmo-measure-customer-ltv`,
  `cmo-update-marketing-spend-business-target`) to `skills[]`
- `capabilities.useCases` — updated the spend-review entry to reference
  `cmo-optimize-marketing-budget`; added entries for cohort analysis, customer LTV, and
  marketing spend/target updates
- `tools[]` — added 7 tools that skills already referenced but the manifest never
  declared: `get_business_domain`, `get_cohort_analysis`, `get_customer_profitability`,
  `get_ltv_distribution`, `get_ltv_segment`, `get_trading_week`, `list_ltv_customers`
- `tools[]` — renamed three write-adjacent tools to match the skill content, per
  Andrew's confirmation that the skill naming is authoritative:
  `update_marketing_spend` → `upsert_marketing_spend`, `update_target` →
  `upsert_target`, `list_channel` → `get_marketing_channels` (this reverses a same-day
  correction that had gone the other way — see "Bug fixes" below)

### Skills — new

- `cmo-analyze-cohort` — tracks weekly acquisition cohorts over time: break-even speed,
  repeat-purchase rate, CAC trend per cohort, and cohort-to-cohort comparison
- `cmo-measure-customer-ltv` — segments the customer portfolio by recency
  (Active/Lapsed/Dormant/Churned) and profitability (LTV deciles/quartiles, break-even
  status) to prioritise loyalty and retention targeting
- `cmo-update-marketing-spend-business-target` — explains the three sources of marketing
  spend (budget, platform extract, manual override) and their cascading precedence, and
  how business targets are set

### Skills — renamed

- `cmo-adjust-marketing-spend` → `cmo-optimize-marketing-budget` (same weekly
  increase/hold/reduce/cut spend logic; folder and internal references updated to match)

### Bug fixes

- `cmo-measure-customer-ltv` — frontmatter was missing its closing `---`, which
  swallowed the entire skill body into the YAML `description` field; the skill's
  instructions were unreadable by any loader. Fixed
- `cmo-analyze-cohort` — frontmatter `description` block was not indented, which is
  invalid YAML for a block scalar (confirmed parse error via js-yaml: *"can not read a
  block mapping entry; a multiline key may not be an implicit key"*). Fixed
- `cmo-diagnose-contribution-margin`, `cmo-diagnose-growth` — opening `---` had a
  trailing space (`"--- "`), which a strict frontmatter parser would fail to recognise
  as the delimiter, silently dropping `name`/`description`. Fixed
- `cmo-update-marketing-spend-business-target` — frontmatter `name:` did not match its
  folder name. Fixed
- Fixed dead references to two retired skill names, found across 10 files
  (`cmo-build-profit`, `cmo-optimize-marketing-budget`, `cmo-diagnose-contribution-margin`,
  `cmo-diagnose-growth`, `cmo-explain-marketing-advertising`, `cmo-health-check`,
  `cmo-list-recent-orders`, `cmo-set-marketing-budget`, `cmo-router`, `cmo-primary`):
  `cmo-get-diagnosis` → `cmo-diagnose-metrics`, `cmo-adjust-marketing-spend` →
  `cmo-optimize-marketing-budget`, and one instance of bare `cmo-contribution-margin` →
  `cmo-diagnose-contribution-margin`
- `cmo-router` — routing table and Step 5 "what can you do" summary had no entry for
  `cmo-analyze-cohort`, `cmo-measure-customer-ltv`, or
  `cmo-update-marketing-spend-business-target`, so a founder asking a vague question
  about cohorts, LTV, or spend/target updates would never be handed off to them. Added
- Tool naming in `cmo-primary` and `cmo-update-marketing-spend-business-target` was
  initially cross-checked against the live `dw_1pd_app.ref_claude_connector_definition`
  table, which at the time only had `update_marketing_spend` / `update_target` /
  `list_channel` active — so those skills were first edited to match the database.
  Andrew subsequently confirmed the skill naming (`upsert_marketing_spend`,
  `upsert_target`, `get_marketing_channels`) is authoritative; that edit was reverted,
  and `plugin.json`'s `tools[]` was renamed to match the skills instead (see "Manifest"
  above). `cmo-primary` also had two other spots using the database names
  (`update_marketing_spend`/`update_target` in its "Write tools" bullet, `list_channel`
  in its parameter-gathering rules) — normalised to the same
  `upsert_*`/`get_marketing_channels` naming for internal consistency. A later re-query
  of the database confirmed it had since been migrated to the new names — the old
  `update_marketing_spend`/`update_target`/`list_channel`/`get_ltv_segments` no longer
  exist in the table at all (active or inactive), so the skill naming and the database
  now agree
- `cmo-update-marketing-spend-business-target` also referenced `sp_claude_upsert_target()`,
  which never existed in the database under any name. Corrected to `upsert_target()`,
  the confirmed active tool
- `cmo-primary` used `get_ltv_segment` (singular) while `cmo-measure-customer-ltv` used
  `get_ltv_segments` (plural) for what is the same tool. Andrew confirmed the singular
  form is correct; `cmo-measure-customer-ltv` updated to match `get_ltv_segment`, so
  `plugin.json` and every skill now agree

### Known issue — not fixed this release

- `cmo-build-profit/SKILL.md` is a byte-for-byte duplicate of `cmo-diagnose-growth/SKILL.md`
  (identical content, only the `name:` field differs). `cmo-router` and `plugin.json`
  both point to `cmo-diagnose-growth` only; `cmo-build-profit` is kept in the skills
  folder but intentionally left out of `plugin.json`'s `skills[]` to avoid two identical
  skills competing for the same trigger. Needs a decision from Andrew: delete
  `cmo-build-profit`, or clarify why both should exist.

---

## [1.1] — 2026-07-07

### Tools — new (write access)

- `update_marketing_spend` — writes a new daily marketing spend for a channel over a date range
- `update_target` — writes a new founder-set target for a metric
- `list_channel` — returns the store's active marketing channels and their `platform_code` values (resolves the channel for the two write tools above)

### Manifest

- `plugin.json` — added the three new tools to `tools[]`; changed `capabilities.dataAccess` from `read-only` to `read-write`; updated `oauth` scopes from `["read"]` to `["read", "write"]`; updated `useCases` and `dataHandling` to describe founder-confirmed write actions
- Bumped `version` to `1.1`

### Skills — updated

- `cmo-adjust-marketing-spend` — added Step 7: confirm-and-apply flow for `update_marketing_spend`. Never calls the write tool without explicit founder confirmation and all fields (`platform_code`, `from_date`, `to_date`, `marketing_amt`) gathered out loud first
- `cmo-set-marketing-budget` — added Step 9: confirm-and-apply flow for `update_marketing_spend` and `update_target`; fixed a stale skill reference (`/cmo-adjust-marketing-budget` → `/cmo-adjust-marketing-spend`)
- `cmo-primary` — added `list_channel`, `list_recent_orders`, `update_marketing_spend`, `update_target` to the tool list; added a "Gathering required tool parameters" section establishing that write-tool fields are always asked and confirmed, never inferred or defaulted
- `cmo-router` — added a routing row and Step 5 summary line for `cmo-set-marketing-budget`, which previously had no route into it from the front door

---

## [1.0] — 2026-07-03

### First public release — Plugin Directory submission

- `plugin.json` fully declared: tools, skills, capabilities, branding, URLs
- All 12 skills production-ready and fully documented

### Manifest

- Filled out `.claude-plugin/plugin.json` with the full listing schema: `tagline`, `server` (auth/transport), `tools` (with titles, descriptions, `readOnlyHint`), `capabilities` (`dataAccess`, `useCases`, `dataHandling`), `branding`, `privacyPolicyUrl`, `documentationUrl`, `supportUrl`, `category`, `tags`
- Renamed `list-recent-orders` tool to `list_recent_orders` for naming consistency with the rest of the tool set
- Rewrote `displayName`, `tagline`, `description`, `category` (`Analytics` → `ecommerce`), and `tags` to align with production positioning; refreshed `documentationUrl`/`supportUrl`
- `.mcp.json` — removed the redundant `type: "http"` field
- Added `README.md`

### Skills — restructured

- Split the former `cmo-get-diagnosis` catch-all into three focused skills:
  - `cmo-diagnose-metrics` — targeted drill-down for a named metric or domain
  - `cmo-diagnose-contribution-margin` — domain sub-skill isolating discount/COGS/shipping/marketing drivers of margin movement
  - `cmo-diagnose-growth` — growth-quality assessment answering "should I spend more on ads" (replaces `cmo-build-profit`)
- Retired `cmo-get-diagnosis`, `cmo-contribution-margin`, and `cmo-build-profit`; superseded copies kept under `skills/bk/` during the transition and removed once the new skills stabilized

### Skills — new

- `cmo-adjust-marketing-spend` — weekly increase/hold/reduce/cut recommendation with a specific dollar amount, based on MER trend and contribution margin
- `cmo-set-marketing-budget` — five-step MER method for calculating a data-grounded monthly/daily marketing budget
- `cmo-explain-marketing-advertising` — marketing/advertising concept explainer
- `cmo-list-recent-orders` — surfaces recent Shopify orders enriched with contribution margin data for data-connection verification

### Skills — updated

- `cmo-primary`, `cmo-router`, `cmo-terminology`, `cmo-health-check`, `cmo-list-recent-orders` — wording and formatting fixes; removed references to session state in favor of Claude's session expiry

---

## [0.2] — 2026-06-11

### Skills — new

- `cmo-router` — Front door skill; triggers on session start or vague requests ("hi", "what can you do", "help me"); checks connector status, greets by store name, and routes to the correct skill based on intent without performing analysis itself
- `cmo-terminology` — CMOgpt business term glossary (NB, RP, MER, SOB, VAC, LTV, etc.) plus data-lag handling rules for Shopify vs GA4 metrics and `low_confidence` flag guidance

### Skills — updated

- `cmo-health-check` — Rewrote procedure to use `get_diagnosis(job_id, 'SALES-AMT', 7, 3)` + `get_metric_history` for the six health indicators; removed deprecated `get_business_7d_snapshot` call; added explicit session-context step (job_id, plan, store_name, shopify_last_order_date); tightened verdict format and added "What this skill is not" boundary note directing deeper analysis to `cmo-get-diagnosis`
- `cmo-get-diagnosis` — Reframed as "Strategic Diagnosis" for named-metric drill-downs; added clear trigger rules (requires a named metric or domain; open weekly questions → `cmo-health-check`); added 6-step procedure including `available_metrics` coverage check and domain-fetch logic; expanded priority rules with `score_basis`, `low_confidence`, and segment-split guidance
- `cmo-primary` — Removed deprecated `get_business_7d_snapshot` from tool list; fixed incorrect tool reference `about_my_plan` → `about_my_account`

### MCP server

- Server now correctly identifies as `CMOgpt Connector` (previously reported as `CMOgpt-connector`)

---

## [0.1] — 2026-06-10

### Initial release

**MCP Server**
- Server name: `CMOgpt Connector`
- Endpoint: `https://connector.cmogpt.io/mcp`
- Authentication: OAuth 2.0 / Bearer JWT required

**Tools**
- `get_metrics` — list available metrics across all five domains (SALES, PROFITABILITY, REPEAT-BUSINESS, MARKETING, LTV)
- `get_metrics_manifest` — full metric catalog for a given domain
- `get_metric_detail` — detailed definition and context for a single metric
- `get_metric_history` — historical data points for a metric over a configurable look-back window
- `get_diagnosis` — profitability and performance diagnosis, optionally focused on a nominated metric code
- `get_marketing_budget` — daily marketing budget data for the connected store
- `get_benchmarks` — industry benchmark values for each metric
- `get_my_targets` — user-defined targets for the store's KPIs
- `about_my_store` — store context: category, target customers, age, size, growth stage, `shopify_last_order_date`
- `about_my_account` — CMOgpt account summary: contact, signup date, current plan, connected platforms
- `debug_sql` — ad-hoc SQL execution for advanced diagnostics (restricted by plan)

**Skills**
- `cmo-primary` — establishes CMOgpt's default reasoning framework, the seven-metric hierarchy, and the What / Why / What-next output format; loaded at session start
- `cmo-health-check` — weekly business health review across all five domains
- `cmo-get-diagnosis` — structured diagnosis flow starting from contribution margin
- `cmo-contribution-margin` — deep-dive into contribution margin drivers and threats
- `cmo-build-profit` — prescriptive playbook for improving contribution margin from current baseline
