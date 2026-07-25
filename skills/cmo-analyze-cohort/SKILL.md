---
name: cmo-analyze-cohort
description: >
  Track and analyze sales and profitability performance of groups of customers based on their first day of purchase. This focuses on acquisition cost of the customer, and its subsequent purchase pattern over the following weeks. This is the business results of sales, pricing & discounting strategy, marketing activities, loyalty and retention program.
---
# CMOgpt — business answers
This is a longitudinal study of customer profitability performance.  It answers the following question : 
1.	Are newer cohorts more or less profitable than older ones?
2.	How fast does a cohort reach break-even, and is that speeding up or slowing?
3.	What percent of a cohort makes a second purchase, and by when?
4.	Is CAC per cohort rising while contribution stays flat?
5.	Which acquisition week produced the highest-quality customers?
6.	How much of a cohort's value comes from the first order versus repeats?
7.	Is the discount used to acquire a cohort actually paying back?
8.	What does the 90-day contribution curve of a typical cohort look like?
9.	Are my last-30-day cohorts on track to hit target LTV:CAC?
10.	Given how current cohorts are maturing, should I hold, scale, or cut spend?

# Cohort definitions
Customers are grouped into COHORT, based on the COHORT-WEEK (eg  the first purchase) 
Cohort performance is tracked the following days. This is described as DAYS-SINCE-FIRST-ORDER.
Cohort performance are tracked for 6 months.   Eg the user can analyze cohort from upto 6 months ago.
The tool get_cohort_analysis(cohort_date='2026-03-08') will accept any cohort_date, and will translate this into a matching cohort-week.
`cohort_date` must be sent as ISO 8601 `YYYY-MM-DD`. If the founder gives a date in another format (`08/03/2026`, `March 8 2026`, "8 weeks ago", etc.), normalize it to `YYYY-MM-DD` yourself before calling the tool — do not re-ask just to reformat a date you can already parse. Only ask if the date is ambiguous or not resolvable to a real day.


## Cohort analysis

### DAYS-SINCE-ORDER=0 is the first day of purchase.  This is the base line cohort perfomance.
order_count, order_amount, cohort_first_order_CAC, contribution_margin, cohort_contribution_margin_PCT will provide an immediate reading of the profitability 
Further metrics cohort_contribution_LTV_CAC_ratio,  cohort_LTV_CAC_ratio give you a metric that you can benchmark and track over the following weeks.
Of special interest is customer_break_even_PCT. It explains percentage of customer break_even in the first purchase.
 Two further metric orders_with_discount_PCT, orders_with_free_shipping_PCT, could further pinpoint the cause of poor initial performance.

### DAYS-SINCE-ORDER>0  is the sales performance of this cohort in the number of days after the first day of purchase.  This is the sales performance as of that date. All the counts and dollars are cumulative from DAYS-SINCE-ORDER=0.  The ratios and PCT are recalculated as of that day. 
This is a sparse dataset, and some days-since-order may not have any sales activities for this cohort.
When analyzing cohort history use cumulative metrics where every available. 
For example cumulative_order_count, customer_count, cumulative_order_amount, cumulative_repeat_order_count, cumulative_contribution_margin
cumulative_repeat_order_count measures the combined sales performance of this cohort
cumulative_orders_with_discount, cumulative_orders_with_free_shipping counts the total number of orders with discounts.
Of special interest is customer_break_even_PCT.  This tracks the percentage of customer broken even at this DAYS-SINCE-FIRST-ORDER.
Other ratios such as cohort_contribution_margin_PCT, cohort_contribution_LTV_CAC_ratio and cohort_LTV_CAC_ratio are calculated as a entire cohorts contribution to the business.

### COMPARISON & PROJECTION
To analyze or compare to a different cohort, use get_cohort_analysis(cohort_date=?).  
Compare based on the DAYS-SINCE-FIRST-ORDER between the two cohort can give you progress 'benchmark' against a known point.
