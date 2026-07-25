---
name: cmo-inspect-customer
description: >
  Line-of-sight into a single customer's profitability for Shopify brands. Use when
  the founder asks "show me customer X", "why is this customer profitable", "break
  down this customer's LTV", or "how was this customer's CAC calculated". Shows the
  customer's orders rolling up into margins, CAC and break-even so the founder can
  trust the number. Not for portfolio or segment analysis.
---

# CMOgpt — Inspect Customer Profitability

## Purpose
Trust. The founder makes decisions on LTV and CAC; this skill shows the working so
they believe it — how a customer's orders roll up gross → delivered → contribution
margin, and how a static Customer-CAC was set at first order.

## Inputs
- `get_customer_profitability(customer_id)` — customer LTV row + supporting orders.

## How to reason
1. **Show the build-up, not just the total.** Walk each order: gross margin →
   delivered margin (less discount, shipping, 3PL, packaging, fees) → contribution
   margin (less allocated marketing). Then sum to contribution LTV.
2. **CAC is static.** Customer-CAC is fixed at the first-order date and does not
   re-allocate as new orders arrive. Say so plainly — it is a common confusion.
3. **Break-even in plain terms.** State whether and when the customer broke even
   (`break_even_days`, `is_break_even`) and what the first order contributed toward
   CAC (`first_order_contribution_cac_PCT`).
4. **Flag discount / free-shipping drag** if it materially moved contribution.

## Output
An ordered, line-by-line walk-through the founder can follow — here, showing the
numbers *is* the value. Close with a one-line verdict: profitable, break-even, or
underwater, and why.

## Boundaries
Aggregates and "which customers" lists → cmo-analyze-ltv / list_customers. This
skill is always about one named customer.