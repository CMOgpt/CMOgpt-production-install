---
name: cmo-update-marketing-spend-business-target
description: >
  CMOgpt uses marketing spend to calculate profitability. CMOgpt also allows users to set targets for specific metrics.
---
# marketing spend

Marketing spend has the most significant impact on profitability.  This is the core component of Contribution calculation.  Marketing spend is calculated for each day.
The source of Marketing spend can come from 3 sources

## marketing budget
A monthly budget, set by the users. Usually this is set for the whole business.  The user can set up monthly budget using the CMOpgt UI.  A monthly budget is allocated evenly across the days of the month. 

## platform spend
Daily extract of actual account spend for advertising platforms.  This requires users to connect their CMOgpt account to their marketing platforms.   
Typically Meta and Google Ads are two primary sources. 

## user manual input
Users can override both the marketing budget or the platform spend data.  User can override platform spend for a specific date or a range of dates.  
There are two ways that the user can update marketing spend. 
1. use CMOgpt UI to update platform spend for a given day
2. Use Claude tool upsert_marketing_spend(). This tool needs platform_code, from_date and to_date

## cascading hierarchy
All of these sources are optional. And user may progressively set up marketing spend as they wanted more granular measurement of profitability.
CMOgpt will resolve which budget to use by the following cascading method
Use budget if exist
If platform spend is available, sum all platform spend, and replace spend based on budget
If the user entered budget exist, use this to override both platform spend and monthly budget

## refresh metrics
Users can refresh all the metrics after updating the budget or manual override. This is done via a metrics recalc option on the CMOgpt UI.


# business target
Users can set business target. This can be done by 
1. CMOgpt UI 
2. use Claude tool upsert_target()