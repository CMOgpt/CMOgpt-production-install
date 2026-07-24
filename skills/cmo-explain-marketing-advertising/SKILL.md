---
name: cmo-explain-marketing-advertising
description: >
  Explains marketing vs advertising concepts, MER vs full-loaded MER, and
  acquisition vs retention budget structure to ecommerce founders. Triggers
  when the founder asks "what is MER", "what's the difference between
  marketing and advertising", "how should I split my budget", "what counts
  as marketing spend", "what is full-loaded MER", or any question about
  understanding or structuring their marketing budget. Grounds explanations
  in the founder's own store data where available. Routes to
  cmo-set-marketing-budget when the founder is ready to act.
---

# CMOgpt — Explain Marketing & Advertising

## Purpose

Many ecommerce founders use "marketing" and "advertising" interchangeably.
They don't mean the same thing, and conflating them leads to bad budget
decisions and misleading performance metrics. This skill corrects that
framing, explains how MER is calculated and what it really measures, and
introduces the acquisition/retention split that separates growing brands
from ones chasing revenue at any cost.

This is an educational skill. Its job is to give the founder a clear mental
model they can act on — not to generate a report. Where the founder's own
data is available, ground every explanation in their actual numbers.

---

## Triggers

Use this skill when the founder asks:
- "What is MER?" / "What does MER mean?" / "How is MER calculated?"
- "What's the difference between marketing and advertising?"
- "What counts as my marketing spend?"
- "What is full-loaded MER?"
- "How should I split my budget between acquisition and retention?"
- "Do I need a retention budget?" / "Is email part of my marketing budget?"
- "What should I be spending on influencers / creatives / email?"
- Any question about how to categorise, understand, or frame marketing spend

---

## Procedure

### Step 1 — Get store context

Call `about_my_account` and `about_my_store`. Extract:
- `job_id` — required for any tool calls
- `growth_stage` — determines the default acquisition/retention allocation
  to reference when explaining budget splits
- `category` — informs which cost lines are typically significant for this
  type of brand (e.g. creatives are a major line for fashion/cosmetics)
- `shopify_last_order_date` — for data freshness if pulling live metrics

If either tool fails or the connector is not installed, proceed with the
conceptual explanation only. Do not block the explanation on live data.

---

### Step 2 — Establish the core distinction

#### Marketing vs Advertising

**Advertising** = paid media spend only. What you pay platforms (Meta, Google,
TikTok) to show your ads. This is what most founders mean when they say
"marketing cost."

**Marketing** = everything you spend to acquire, activate, and retain
customers. It is a much wider category:

| Category              | Examples                                                      |
|-----------------------|---------------------------------------------------------------|
| Performance ads       | Meta Ads, Google Ads, TikTok Ads                              |
| Influencer & affiliate| Influencer campaigns, affiliate commissions, paid partnerships|
| Creative services     | Ad creative production, product photography, video shoots     |
| Email & SMS           | Platform costs (Klaviyo, Attentive), campaign management      |
| Loyalty & retention   | Loyalty program costs, winback campaigns, post-purchase flows |
| Agency & consulting   | Ecommerce agencies, media buying, growth advisors             |
| Events & ATL          | Trade shows, above-the-line advertising, promotions           |
| Tools & platforms     | Analytics platforms, marketing software subscriptions         |

> **Important for fashion and cosmetics brands:** creative production costs —
> product shoots, UGC, video — can be 15–30% of total marketing spend. These
> are marketing costs, not operational costs. If you are not including them
> in your marketing budget, your MER and contribution margin are both
> overstated.

Frame the explanation relative to the founder's category if known from
`about_my_store`.

---

### Step 3 — Explain MER and full-loaded MER

#### MER (Marketing Efficiency Ratio)

MER is the ratio of total revenue to advertising spend:

```
MER = Total Revenue ÷ Advertising Spend
```

A MER of 4.0x means for every $1 spent on ads, $4 of revenue came back.

Most founders calculate MER using advertising spend only (Meta + Google +
TikTok). This is the standard definition in CMOgpt.

#### Full-Loaded MER

Full-Loaded MER uses total marketing spend — not just advertising:

```
Full-Loaded MER = Total Revenue ÷ Total Marketing Spend
```

Total marketing spend includes advertising plus creative, agency, email
platform, influencer, loyalty program costs — everything in the marketing
budget table above.

Full-Loaded MER is almost always lower than standard MER, because the
denominator is larger. It is a more honest measure of marketing efficiency.

| Measure         | Denominator                          | Typical use               |
|-----------------|--------------------------------------|---------------------------|
| MER             | Paid media spend only                | Daily/weekly media review |
| Full-Loaded MER | All marketing costs                  | Budget planning, P&L review|

> If a founder is proud of a 5.0x MER but has not counted their agency
> retainer, creative fees, or email platform, their real Full-Loaded MER
> may be 3.2x. The contribution margin picture looks very different.

If live data is available, pull `get_metric_history(job_id, 'MER', 30)` to
show the founder their actual MER trend alongside this explanation.

---

### Step 4 — Introduce the acquisition/retention split

#### Why the split matters

DTC ecommerce brands should divide their marketing budget into two distinct
buckets with separate owners, targets, and accountability:

**Acquisition budget** — buying new customers.
**Retention budget** — keeping and growing existing customers.

Repeat purchases should never be treated as "free revenue." They require
their own budget, targets, and attention. Brands that do not have a
retention budget are implicitly spending their entire marketing budget
acquiring customers and hoping retention happens on its own. It does not.

#### What each budget covers

**Acquisition budget:**
- Meta Ads (prospecting campaigns)
- Google Ads (prospecting, branded excluded)
- TikTok Ads
- Influencer campaigns
- Affiliate commissions
- Paid partnerships
- Creative testing
- New-to-brand prospecting

**Measured by:**
- Conversion rate
- CAC (Customer Acquisition Cost)
- First order value
- New business contribution margin
- MER of new business
- Payback period
- Contribution margin : CAC ratio

**Retention budget:**
- Email platform + campaign management (Klaviyo, etc.)
- SMS platform + campaigns
- Loyalty program costs
- Post-purchase campaign flows
- Winback programs
- Customer segmentation and repeat purchase campaigns
- Re-engagement advertising (existing customers)

**Measured by:**
- Repeat revenue
- Repeat ratio
- Days between first and second purchase
- Active vs lapsed customer ratio
- Purchase frequency
- Customer contribution-LTV
- Revenue per customer

---

### Step 5 — Explain allocation by growth stage

The right split between acquisition and retention is not fixed. It depends
on where the brand is in its growth:

| Growth stage       | Acquisition | Retention |
|--------------------|-------------|-----------|
| New brand          | 80–90%      | 10–20%    |
| Growing brand      | 70–80%      | 20–30%    |
| Mature brand       | 60–70%      | 30–40%    |
| High-repeat category | 50–70%    | 30–50%    |

If `growth_stage` is available from `about_my_store`, reference the
founder's specific stage in your explanation. Tell them what the default
allocation looks like for a brand at their stage, and whether their current
spend pattern appears to match it.

If live budget data is available, call `get_marketing_budget(job_id)` and
show the founder where their current spend sits relative to this framework.

---

### Step 6 — Respond with the right level of depth

This skill handles a range of founder questions — from a simple "what is
MER?" to "help me think about how to structure my marketing budget."

Match the depth of the answer to the question asked:

- **Single concept question** (e.g. "what is MER"): Answer in 3–5 sentences
  with a concrete example using their numbers if available. Do not dump the
  full framework unprompted.
- **Framing question** (e.g. "what counts as my marketing spend"): Cover the
  marketing vs advertising distinction and the cost category table. Include
  a note about their specific category if relevant.
- **Budget structure question** (e.g. "how should I split my budget"): Cover
  the acquisition/retention split, the allocation by growth stage, and what
  each budget covers and is measured by.
- **Full framework question** (e.g. "explain how marketing budgets work"):
  Cover all sections. Keep it conversational, not a lecture.

Always ground the explanation in the founder's actual numbers where possible.
The best explanations connect the concept to something they can see in their
own business right now.

---

### Step 7 — Route to action

This skill explains the framework. When the founder is ready to act on it,
route them:

- To set or review their marketing budget → `/cmo-set-marketing-budget`
- To adjust spend based on performance → `/cmo-optimize-marketing-budget`
- To diagnose a specific metric → `/cmo-diagnose-metrics`
- For overall health check → `/cmo-health-check`

Do not wait for the founder to ask. If they understand the framework, close
with one sentence: "Want me to look at your current budget against this
framework?" or "Ready to set your acquisition and retention targets?"

---

## Tone and constraints

- Use plain language. These are founders who run lean teams — not CFOs or
  media agency planners.
- Use the founder's actual numbers whenever live data is available. A concept
  explained with their own MER is worth ten times the same concept explained
  in the abstract.
- Never imply their current approach is wrong without evidence. Frame
  corrections as "here's how this is typically structured" before showing
  where their setup may differ.
- Keep the full framework explanation under 5 minutes of reading time.
- Do not present all sections if the founder asked a narrow question.

---

## What this skill is not

This is not a budget-setting tool. It explains the concepts and framework.
For setting targets, allocations, and daily budget numbers, use
`/cmo-set-marketing-budget`.
