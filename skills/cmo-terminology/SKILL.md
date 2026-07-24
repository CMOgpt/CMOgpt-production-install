---
name: cmo-terminology
description: >
  CMOgpt terminology and abbreviation reference. Load when the founder asks
  what a CMOgpt term means, or when clarifying metric codes during analysis.
  Covers NB (new business), RP (repeat), AMT, PCT, P1-P5 purchase sequence,
  BUDGET-DAILY, MER, LTV, customer recency segments (active/lapsed/dormant/ churned), and data lag rules for Shopify vs GA4 metrics.
---

# CMOgpt — business terms 
A few abbreviations that is unique to CMOgpt
## CMO terminology
NB : new business, first  time order
RP : repeat purchases
AMT : always dollar amout.  This is also know as revenue
PCT : percentage, it is expressed as fraction of 1.  Eg 0.35 is 35%
P1, P2, P3, P4, P5 :  Purchase sequence 1, 2 and 3.   P1 is therefore new business, P2 to P5 are repeat purchases
MARKETING-BUDGET : A monthly budget, set by the users. Usually this is set for the whole business.  The user can set up monthly budget using the CMOpgt UI.  A monthly budget budget is allocate evenly across the days of the month
PLATFORM-SPEND : Daily extract of actual account spend for advertising platforms.  This requires users to connect their CMOgpt account to their marketing platforms.   
MARKET-SPEND-USER-INPUT :  users can override daily spend by advertising platforms. User can override PLATFORM-SPEND or MARKETING-BUDGET for a specific date or a range of dates.  
MARKETING-SPEND : Is the daily amount spend on all marketing activities. This include as ppc platforms cost, creatives, above the line media and promotion, ecomm, email, analytics and CRM systems, marketing and consulting service.   The source can be sourced from MARKETING-BUDGET, PLATFORM-SPEND or MARKET-SPEND-USER-INPUT.
ORDER-MARKETING-AMT : This is the cost of a sales proportional to the MARKETING-SPEND. This is a measurement of the overall marketing cost of generating  an order.  This included new and return business. It included all marketing activities.   This is a wider and more accurate assessment of the marketing effectiveness.  This is a wider definition than CAC, as CAC measured new business, and only focus on platform costs. 
RECENCY : the number days since a customers last purchase
FREQUENCY : the number orders from a customer 
SOB : source of business. This is the first contact channel as detected by GA4.  Valid values are organic, paid, and earned.
VAC : visitor acquisition cost.
SHIPPING-PCT : the percentage charged to the customer for shipping.  Zero shipping cost means this costs is transferred to the ecomm business. That is this cost affects profitability.
REVENUE-LTV :  total revenue of a customer's orders (12-month rollup).   The industry default definition of LTV s based on total sales of a a customers lifetime.  In CMOgpt, LTV called REVENUE-LTV to distingush this from CONTRIBUTION-LTV
CONTRIBUTION-LTV : total contribution amount from orders of customers in the pas 12 months. This is a profit measure.
CONTRBN-LTV : is an abbreviation of CONTRIBUTION-LTV
IS-BREAK-EVEN : when a customers total sales exceeded the CAC of his first order
MER : expressed as a ratio, and ranges from 1.0+
MER-PCT : expressed as a percentage. It is the same as 1/MER.    Eg MER = 4 is the same as MER-PCT =0.25
PORTFOLIO    : all customers who has purchased in the last 12 months. This is LTV-domain roll-up root.
CUSTOMERS-ACTIVE : is the count of Customers who has purchased last 90 days
CUSTOMERS-LAPSED : count of customers who has purchased between the last 90-180 days
CUSTOMERS-DORMANT : count of customers who has purchased between the last 180-275 days
CUSTOMERS-CHURNED : count of customers who has purchased between  275-365 days. Customers past 365 days falls outside CMOgpt's LTV lookback, effectively treated as lost.   
COHORT : customers who purchase the first time on a specific date (COHORT-DATE)
COHORT-WEEK : the week when this cohort group of customer purchase the first time.  Week starts Sunday to Saturday.
DAYS-SINCE-FIRST-ORDER : is the number of days since the first purchase (COHORT-DATE). This is a qualification of a COHORT.
REACTIVATION-CUSTOMER-COUNT : the number of customers who has migrated from churned, dormant or lapsed segment into active segment.

## How Claude handles data lag
- Shopify metrics: always complete through yesterday. No adjustment needed.
- GA4 metrics: complete through 2 days ago. For day_span ≥ 14, proceed normally — the OLS 
  regression handles the 2 missing points. For day_span ≤ 7, treat GA4 trend signals as 
  supporting evidence only, not a lead finding. Check `low_confidence` flag in the output.
- Blended metrics (MER, CONVERSION-RATE, SALES-VISITOR): flag as low_confidence if either 
  upstream source is incomplete.

When `low_confidence = 1`, do not lead with that metric's trend signal alone. You may still 
use it to corroborate a finding that is confirmed by a Shopify-only metric.