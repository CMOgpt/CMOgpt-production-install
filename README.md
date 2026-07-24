# CMOgpt — AI CMO for Shopify

CMOgpt is a read-only AI marketing analyst for Shopify brands. Ask in plain English to find out what's actually making money, where margin is leaking, whether ad spend is paying back, and which customers are worth keeping — each answer paired with a concrete next move.

## What CMOgpt Does

- **Health check** — weekly snapshot of revenue, margin, MER, discount rate, and repeat ratio read as a system, with a single verdict and priority action
- **Metric diagnosis** — root-cause drill-down into any underperforming metric using a pre-scored causal decision tree
- **Contribution margin diagnosis** — identify whether margin pressure is driven by discounting, COGS, shipping subsidy, or marketing cost
- **Growth quality diagnosis** — evaluate whether revenue growth is compounding value or buying revenue at the expense of margin and retention
- **Marketing spend review** — weekly recommendation to increase, hold, reduce, or cut spend based on MER trend and contribution margin
- **Marketing budget setting** — five-step MER method to calculate a data-grounded monthly and daily marketing budget
- **Order verification** — view recent Shopify orders enriched with contribution margin data to verify live data connection

## Connector Setup

1. Open Claude and search for **CMOgpt** in the integrations directory
2. Click **Connect** — you will be redirected to `cmogpt.io` to log in
3. Select the Shopify store to connect
4. Return to Claude — CMOgpt is ready to use

The connector uses OAuth 2.0 with PKCE. Your Shopify data is accessed read-only and never shared with third parties other than Shopify (source) and Anthropic (Claude processing).

## Skills (Slash Commands)

| Skill | Command | Description |
|---|---|---|
| Primary | `/cmo-primary` | Entry point — routes your question to the right skill |
| Router | `/cmo-router` | Internal routing logic |
| Health Check | `/cmo-health-check` | Full weekly business health snapshot |
| Diagnose Metrics | `/cmo-diagnose-metrics` | Root-cause drill-down on any metric |
| Diagnose Contribution Margin | `/cmo-diagnose-contribution-margin` | Margin pressure breakdown |
| Diagnose Growth | `/cmo-diagnose-growth` | Growth quality vs revenue-buying diagnosis |
| Adjust Marketing Spend | `/cmo-adjust-marketing-spend` | Weekly spend recommendation with dollar amount |
| Set Marketing Budget | `/cmo-set-marketing-budget` | MER-based budget calculation |
| Explain Marketing & Advertising | `/cmo-explain-marketing-advertising` | Definitions for marketing metrics and terms |
| Terminology | `/cmo-terminology` | CMOgpt-specific metric definitions |
| List Recent Orders | `/cmo-list-recent-orders` | Recent orders with contribution margin data |

## MCP Tools

All tools are read-only (`readOnlyHint: true`).

| Tool | Description |
|---|---|
| `about_my_store` | Store profile — category, growth stage, last order date |
| `about_my_account` | Account details including connected platforms |
| `get_diagnosis` | Pre-scored causal decision tree for a metric |
| `get_marketing_budget` | Current daily marketing budget |
| `get_metrics_manifest` | All available metrics by domain |
| `get_metric_detail` | Current value, benchmark, and status for a metric |
| `get_metric_history` | Historical daily data for a metric (up to 90 days) |
| `get_my_targets` | Founder-set metric targets |
| `get_benchmarks` | Industry benchmarks matched to store category |
| `list_recent_orders` | Up to 200 recent Shopify orders with CM data |

## Support

- Website: [cmogpt.io](https://cmogpt.io)
- Support: [cmogpt.io/support](https://cmogpt.io/support)
- Documentation: [cmogpt.io/docs](https://cmogpt.io/docs)
- Privacy Policy: [cmogpt.io/privacy](https://cmogpt.io/privacy)
- Email: support@cmogpt.io
