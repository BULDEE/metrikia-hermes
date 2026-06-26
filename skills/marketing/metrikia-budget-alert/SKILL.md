---
name: metrikia-budget-alert
description: Autonomous budget and anomaly monitoring for ads via Metrikia. Detects aberrant spend, dropping ROAS, creative fatigue, and alerts only when action is required. Runs on a schedule via blueprint, delivers to the home channel.
version: 1.0.0
author: Alexandre Mallet / BULDEE
license: Apache-2.0
platforms: [macos, linux]
metadata:
  hermes:
    tags: [Marketing, Monitoring]
    blueprint:
      schedule: "0 */6 * * *"
required_environment_variables:
  - name: METRIKIA_API_KEY
    prompt: "Metrikia API key (mk_live_...) for the Metrikia MCP server"
---

# Metrikia budget and anomaly alert

Autonomous watch: speak only when something needs action. No noise.

## Prerequisite

Metrikia MCP server configured. Tools: `get_anomalies`, `get_metrics`, `get_budget_advice`, `get_sync_status`.

## When to use

- Autonomously via blueprint (every 6h).
- On demand: "check my ad alerts".

## Procedure

1. **Sync**: `get_sync_status`. If data is lagging by more than a few hours, report the sync lag and stop (no alert on stale data).
2. **Anomalies**: `get_anomalies` over the last 24h. Filter on real severity (ROAS dropping below a threshold, exploding spend, confirmed creative fatigue).
3. **Budget**: if an anomaly touches allocation, `get_budget_advice` for the concrete recommendation (cut, cap, redistribute).
4. **Notification decision**:
   - Nothing significant: do NOT notify (silence is a good sign).
   - Action required: short alert + actionable recommendation.

## Alert format (only if action is required)

```
Metrikia ad alert

[campaign/lever]: [problem] (Metrikia attribution)
Impact: ...
Recommended action: ...
```

## Pitfalls

- High noise threshold: better to miss a micro-variation than to spam. One alert = one action.
- Never alert on a platform ROAS. Always Metrikia attribution.
- No alert on stale data (check the sync first).

## Verification

If nothing is actionable, the run ends with no notification. Any alert emitted contains a concrete action, not just an observation.
