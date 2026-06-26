---
name: metrikia-ops-health
description: Autonomous operational-health watch on ad accounts via Metrikia (campaign status changes, inactive creatives, sync lag, anomalies across Meta/Google/TikTok). Metrikia already ingests all provider data, so no separate provider MCP is needed. Alerts only when action is required. Never writes to ad budgets.
version: 1.1.0
author: Alexandre Mallet / BULDEE
license: Apache-2.0
platforms: [macos, linux]
metadata:
  hermes:
    tags: [Marketing, Monitoring]
    blueprint:
      schedule: "0 */4 * * *"
required_environment_variables:
  - name: METRIKIA_API_KEY
    prompt: "Metrikia API key (mk_live_...) for the Metrikia MCP server"
---

# Metrikia ops-health watch

Autonomous watch for the operational state of ad accounts. This is the lifecycle/operational side (a campaign silently went paused or archived, a creative went inactive, a sync stopped, an anomaly), distinct from the performance side (`metrikia-weekly-report`, `metrikia-budget-alert`).

Metrikia already ingests all provider data (Meta, Google, TikTok) and serves it back, so this skill reads **Metrikia only**. No separate provider MCP is required or wanted.

## Boundary (non-negotiable)

Autonomous agent: it **OBSERVES and PROPOSES, never acts on the budget**. No pause, no scale, no budget change. Metrikia does not even expose an ad-budget write tool, so the agent is structurally incapable of it. If an action is warranted, recommend it and point the user to the human tool (the Metrikia Claude plugin `/metrikia:ads-ops`, or the platform UI).

## Prerequisite

Only the Metrikia MCP (already wired). Tools used: `list_campaigns`, `get_creative_report`, `get_anomalies`, `get_sync_status`, `get_campaign_performance`.

## When to use

- Autonomously via blueprint (every 4h).
- On demand: "check my ad accounts health".

## Procedure

1. **Sync health**: `get_sync_status`. If a provider/account sync is lagging, that itself is the alert (the data is stale, downstream numbers cannot be trusted). Report and stop for that account.
2. **Campaign status sweep**: `list_campaigns` across all accounts. Flag unexpected transitions: a campaign that is `paused` or `archived` but was active and high-value, or an account with everything paused.
3. **Creative state**: `get_creative_report`. Flag `isActive=false` on creatives that should be running (a delivery gap), and confirmed fatigue.
4. **Anomalies**: `get_anomalies`. Cross-reference with `get_campaign_performance` to rank urgency (an issue on a high attributed-ROAS campaign is more urgent).
5. **Notify decision**:
   - Everything nominal: do NOT notify (silence is good).
   - Real operational issue: short alert + recommended fix + WHERE to do it (human tool / platform UI). No automatic action.

## Alert format (only if action is required)

```
Metrikia ops-health alert

[account / campaign]: [issue, e.g. active high-ROAS campaign now paused, or sync lagging]
Impact: ...
Recommended fix: ...
Execute from: /metrikia:ads-ops (Metrikia Claude plugin) or the platform UI -- this agent does not write
```

## Coverage note

Metrikia exposes campaign/ad lifecycle status (active/paused/archived, isActive), sync state, and anomalies. Deeper platform-policy signals (ad disapprovals, account flags, billing/spend-limit holds) are surfaced only if the Metrikia MCP exposes them; if not, that is a Metrikia product enhancement (expose the providers' effective_status / review status), after which this skill picks them up with no change to the boundary.

## Pitfalls

- Never take a write action. Recommend, do not execute.
- High noise threshold: one alert = one real operational problem.
- Operational state, not performance. For ROAS/budget, use the Metrikia performance skills.

## Verification

If accounts are nominal, the run ends with no notification. Any alert is a real operational issue with a recommended fix and a pointer to the human tool. The agent performed zero ad-budget writes.
