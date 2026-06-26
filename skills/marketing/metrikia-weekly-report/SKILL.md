---
name: metrikia-weekly-report
description: Weekly ad performance report via Metrikia (true MTA-attributed ROAS, MER, top campaigns, anomalies, Diana AI recommendations). Triggers on "Metrikia weekly report", "weekly ads report", or on a schedule via blueprint.
version: 1.0.0
author: Alexandre Mallet / BULDEE
license: Apache-2.0
platforms: [macos, linux]
metadata:
  hermes:
    tags: [Marketing, Analytics]
    blueprint:
      schedule: "0 8 * * 1"
required_environment_variables:
  - name: METRIKIA_API_KEY
    prompt: "Metrikia API key (mk_live_...) for the Metrikia MCP server"
---

# Metrikia weekly report

Generate an ad performance report over the last 7 days, based on **Metrikia attribution (MTA then CRM)**, never the platform self-reported numbers.

## Prerequisite

The Metrikia MCP server must be configured in `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  metrikia:
    url: https://mcp.metrikia.io/api/v1/mcp
    headers:
      Authorization: "Bearer ${METRIKIA_API_KEY}"
```

The Metrikia tools (`list_campaigns`, `get_campaign_performance`, `get_metrics`, `get_anomalies`, `compare_performance`, `ask_diana`) then become available.

## When to use

- The user asks for a weekly report / a weekly ads recap.
- Autonomously via the blueprint (Monday 8am), delivered to the home channel.

## Procedure

1. **Period**: last 7 days (and the previous week for comparison).
2. **Global metrics**: `get_metrics` over the period. Extract MER, global ROAS (Metrikia attribution), total spend, attributed revenue.
3. **Top campaigns**: `list_campaigns` then `get_campaign_performance` on the main ones. Rank by Metrikia-attributed ROAS. Always use MTA/CRM ROAS, never platform ROAS.
4. **Comparison**: `compare_performance` week N vs N-1 (MER/ROAS/CPA trend).
5. **Anomalies**: `get_anomalies` over the period. List alerts (aberrant spend, ROAS dropping, creative fatigue).
6. **AI recs**: `ask_diana` with the report context for 2-3 actionable recommendations.

## Output format

```
Metrikia ads report - week of [date]

MER: X.X | Global ROAS (MTA): X.Xx | Spend: X EUR | Attributed revenue: X EUR
Trend vs N-1: [up/down] MER, [up/down] ROAS

Top 3 campaigns (Metrikia-attributed ROAS)
1. [name] - ROAS X.Xx - spend X EUR
2. ...

Anomalies
- ...

Diana recommendations
- ...
```

Keep it concise. Always state that the ROAS is Metrikia attribution (MTA then CRM), not the platform.

## Pitfalls

- NEVER use a platform self-reported ROAS. If a campaign has no Metrikia attribution, say so (ROAS = unattributed), do not make it up.
- If `get_metrics` returns an empty period or a lagging sync, check `get_sync_status` before concluding.

## Verification

The report is valid if every ROAS figure comes from a Metrikia tool (no estimate) and the period covers a full 7 days.
