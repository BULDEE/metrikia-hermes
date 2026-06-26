---
name: metrikia-ops-health
description: Autonomous read-only operational-health watch on ad accounts (ad disapprovals, delivery stopped, account flags, spend-limit hits) via a READ-ONLY provider MCP. Covers what attribution does not: platform-side operational issues. Alerts only when action is required. Never writes.
version: 1.0.0
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

Autonomous watch for the operational health of ad accounts. This is NOT performance: performance and attribution are Metrikia's job (`metrikia-weekly-report`, `metrikia-budget-alert`). This skill catches the platform-side problems attribution does not see: an ad disapproved, delivery stopped, an account flagged, a spend limit hit.

## Boundary (non-negotiable)

This is an autonomous agent. It **OBSERVES and PROPOSES, it never acts on the budget**. No pause, no scale, no budget change here. If an action is warranted, it recommends it and tells the user to execute it from the human-driven tool (the Metrikia Claude Code plugin `ads-ops` command, or the platform UI). The autonomous agent is not a budget actuator.

## Prerequisite: a READ-ONLY provider MCP

Wire a provider MCP in `~/.hermes/config.yaml` with **read-only scopes only**. The point is that the autonomous agent is structurally incapable of writes.

- Pipeboard (`github.com/pipeboard-co/meta-ads-mcp`) with read scopes, or
- Google Ads MCP (read-only by design), or per-platform read endpoints.

Do NOT wire a write-capable provider MCP into Hermes. Write belongs to the human tool.

Tools used (read): account status, ad review/disapproval status, delivery status, spend-limit / billing status.

## When to use

- Autonomously via blueprint (every 4h).
- On demand: "check my ad accounts health".

## Procedure

1. **Health pull** (read-only): for each connected account, read review status, delivery status, account/billing flags, spend-limit state.
2. **Filter to real issues**: a disapproved ad that is live-relevant, delivery stopped on an active campaign, an account flagged or hitting a spend cap. Ignore noise (a disapproved ad in a paused campaign is not urgent).
3. **Cross-reference impact** (optional): if an issue hits a campaign, you may pull its attributed weight from Metrikia (`get_campaign_performance`) to rank urgency (a delivery stop on a high-ROAS campaign is more urgent).
4. **Notify decision**:
   - Nothing wrong: do NOT notify (silence is good).
   - Real ops issue: short alert + the recommended fix + WHERE to do it (human tool / platform UI). No automatic action.

## Alert format (only if action is required)

```
Metrikia ops-health alert

[account / campaign]: [issue] (operational, platform-side)
Impact: [e.g. delivery stopped on a high-ROAS campaign]
Recommended fix: ...
Execute from: Metrikia plugin /metrikia:ads-ops or the platform UI (this agent does not write)
```

## Pitfalls

- Never take a write action. Recommend, do not execute.
- High noise threshold: one alert = one real operational problem needing the user.
- This is operational health, not performance. For ROAS/budget, use the Metrikia performance skills.

## Verification

If accounts are healthy, the run ends with no notification. Any alert is a real operational issue with a recommended fix and an explicit pointer to the human tool. The agent performed zero writes.
