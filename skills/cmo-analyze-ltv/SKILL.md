---
name: cmo-analyze-ltv
description: >
  Playbook for increasing customer profitability for Shopify brands, built on the
  concepts and segments defined in cmo-analyze-customer-profitability. Use when the
  founder asks "are my customers profitable", "what's my LTV to CAC", "how long
  until a customer pays back", "who are my best customers", or "is my first order
  losing money". Reasons on contribution LTV, not revenue LTV, anchored on the
  portfolio LTV root and the P1-P5 journey. Routes single-customer drill-downs to
  cmo-inspect-customer and acquisition-day quality to cmo-analyze-cohort.
---

# CMOgpt — Customer LTV & Payback

## When this runs
The founder is asking whether the customers they acquire are worth more than they
cost, over a lifetime rather than a single order.

## Inputs (reuse existing metrics — do not expect new CAC/payback codes)
- Root: `LTV-PORTFOLIO-CONTRIB-AMT` (contribution LTV, all customers) and
  `LTV-PORTFOLIO-SALES-AMT` (revenue LTV). Start get_diagnosis here.
- Journey: `P1-LTV`…`P5-LTV`, `Px-CONTRIBUTION-PCT`, `P2-30/60/90/180-DAYS`,
  `Px-DAYS-GAP`, `FREQUENCY`, `FREQUENCY-RP` (LTV domain).
- Efficiency: `CONTRIBUTION-CAC-RATIO` (PROFITABILITY), `PAYBACK-DAYS`,
  `PAYBACK-CYCLE` (MARKETING).
- Break-even reality: `CUSTOMER-BREAK-EVEN-PCT`.
- `get_benchmarks`, `get_my_targets` for the same codes.
- `get_ltv_distribution('LTV-CONTRIB-AMT')` for whale / top-decile questions.

## How to reason
1. **Lead with contribution, not revenue.** `LTV-PORTFOLIO-CONTRIB-AMT` vs CAC is
   the question; `LTV-PORTFOLIO-SALES-AMT` flatters and is context only.
2. **Anchor on 3:1.** State `CONTRIBUTION-CAC-RATIO` and its gap to benchmark.
   Below ~3 is a warning; below 1 the business pays to lose customers.
3. **Split first-order vs repeat economics.** A healthy blended ratio can hide a
   first order that loses money badly, rescued only by P2+. Read `P1-CONTRIBUTION-PCT`
   against the later sequences; if value is back-loaded, say the business is one
   retention wobble from unprofitability.
4. **Payback is cash-flow risk.** Long `PAYBACK-DAYS` plus low
   `CUSTOMER-BREAK-EVEN-PCT` means most customers never pay back — that caps how
   hard the founder can scale spend.
5. **Second-order velocity is the leading indicator.** The `P2-30/60/90/180-DAYS`
   buckets show how fast repeats arrive; a slowing P2 predicts LTV erosion before
   the LTV number moves.
6. **Whales:** use the distribution, report what the top decile shares.

## Output
what / why / what-next (cmo-primary format). One prioritised action, e.g.
"Blended contribution LTV:CAC is 2.1 and only 38% of customers ever break even;
first orders recover 40% of CAC and P2 is slowing past 90 days. Fix repeat rate
before scaling spend — buying more customers today buys unprofitable ones."

## Boundaries
Single customer → cmo-inspect-customer. Cohort quality over time →
cmo-analyze-cohort. Definitions of RFM segments, deciles, or LTV timing/history
rules → cmo-analyze-customer-profitability (this skill assumes those concepts,
it doesn't re-explain them).