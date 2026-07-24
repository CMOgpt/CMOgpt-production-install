---
name: cmo-analyze-customer-profitability
description: >
Evaluate the profitability of customers to the business. CMOgpt calculate the revenue and contributionn of the entire customer base (the PORTFOLIO). 
It further segment customers by their recency of purchase.
This allows the ecomm owners to measure profitability, and design loyalty or retention program.
# business answers
Customer LTV answer these questions from ecomm founders :
1.	Which customers are actually profitable once acquisition cost is subtracted?
2.	What is my portfolio contribution LTV:CAC, and is it above 3:1?
3.	How many days does a new customer take to pay back acquisition cost?
4.	What share of customers ever break even?
5.	Does my first order lose money, and how deeply am I subsidising acquisition?
6.	How much lifetime profit is discounting and free shipping eating?
7.	What is a repeat customer worth versus a one-and-done buyer?
8.	Who are my whales (top-decile LTV) and what do they share?
9.	Does the P1 to P2 gap predict whether a customer becomes valuable?
10.	If I push CAC higher to acquire faster, at what point does LTV:CAC break?
Routes acquisition-   cohort questions to cmo-analyze-cohorts.
---

# LTV segments
Customer LTV is based on a typical consumer retail RFM model.  
## Recency segments
Recency is the days since last purchase.  This is grouped into 4 segments : Active, Lapsed, Dormant and Churned. The cutoff days of these segments are stored in about_my_account. By default these are 90, 180, 270, 365 days.    These can be changed. Tell the founder to contact CMOgpt support to request a one time change
## Frequency
Frequency is the number of orders of  customer in the last 12 months.  This lookback window of 12 months is done deliberately to provide a common base for comparision.
## Monentary
Total order value of a customer is their LTV.   This is a commonly accepted industry definition.  However this revenue based measure does not account for the cost of acquistions, service costs, and revenue forfeited from various discounting.   CMOgpt focus on profit, therefore we use the total contribution margin as a true measuremet of customer profitability.
is_break_even indicate when the total contribution margin of a customers order exceeded the original CAC of that customer's first order.


# Timing & history
## Timing and look back window
CMOgpt calculate LTV weekly.  It looks back 365 days for trading results.  
## History
CMOgpt keeps a history of LTV. However this history starts from the point the user signup with CMOgpt. Therefore there will be no LTV history before this date.  The user signup date can be found from the connector about_my_account() 

# Decile and Quartiles
## customers are ranked by their LTV (sales-LTV, eg sum of total order).  This is grouped 10% bands, with the top 10% revenue customers in decile 10.
Similarily logic applies to Quartile.  It is possible that customer migrate between bands, as their purchase increases.

## How to reason
1. **Active net gain is the heartbeat.** `SEGMENT-NET-GAIN-COUNT` for Active,
   trended, tells you if the base is growing or quietly draining. Lead here.
2. **Contribution concentration.** Report where profit sits. If Churned is 35% of
   customers but negative contribution, it is dead weight, not a win-back target —
   move retention spend to Lapsed, who are recoverable.
3. **Reactivation = loyalty proof.** `SEGMENT-REACTIVATED-*` into Active is the best
   read on whether retention works. Break it by source: winning back Lapsed is
   cheap, Churned is expensive.
4. **Drift cost.** Quantify contribution lost to the Lapsed→Dormant→Churned slide
   so the founder sees the price of inaction.

# How to use LTV in growing the business
## strategy & reasoning
LTV segment bands (Active, Lapsed, etc) gives users a framework for marketing, loyalty and retention strategy.
LTV, contribution-LTV, is_break_even, discounting, free shipping metrics gives ecomm founder priority or sequencing of marketing execution.

## input
use the following tools to advice ecomm founders and build strategy
- `get_ltv_distribution()`  returns an aggregated distribution of equal-count by deciles and quartiles
- `get_ltv_segments()`  returns the customer LTV segments and distribution stats. This returns a time series snapshot of customer LTV segments.
- metrics : `LTV_segment`, `customers_count`,`customers_PCT`, `customers_orders_count`, `customers_repeat_ratio`, 
`list_ltv_customers`, `customers_frequency`, `customer_recency`, `customer_CAC`,  `customers_contribution_LTV`, `customers_contribution_margin_PCT`, 
`customers_marketing_cost`, `is_break_even_PCT`, `ltv_cac_ratio`, `contribution_ltv_cac_ratio`, 
`customers_recency`, `customers_net_gain`, `reactivated_from_lapsed`, `reactivated_from_dormant`, `reactivated_from_churned`
- `list_ltv_customers(segment, ltv_decile, is_break_even, list_order)`  - returns the top a list of customers of a specific segment (Active, Lapsed, Dormant etc), LTV band, and is_break_even status
- `get_customer_profitability(customer_id)` to inspect a customer's profitability, showing a full order history, cost of acquisition, marketing, margins and profitability

