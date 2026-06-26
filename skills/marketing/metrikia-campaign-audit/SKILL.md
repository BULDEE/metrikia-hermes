---
name: metrikia-campaign-audit
description: Deep-dive audit of an ad campaign via Metrikia (performance, creative breakdown, multi-touch attribution journey, recommendations). Triggers on "audit campaign X", "Metrikia campaign deep-dive".
version: 1.0.0
author: Alexandre Mallet / BULDEE
license: Apache-2.0
platforms: [macos, linux]
metadata:
  hermes:
    tags: [Marketing, Analytics]
required_environment_variables:
  - name: METRIKIA_API_KEY
    prompt: "Metrikia API key (mk_live_...) for the Metrikia MCP server"
---

# Metrikia campaign audit

Deep-dive on one campaign: attributed performance, creatives, conversion paths, levers.

## Prerequisite

Metrikia MCP server configured (see the `metrikia-weekly-report` skill). Tools used: `list_campaigns`, `get_campaign_performance`, `get_creative_report`, `get_attribution_journey`, `get_budget_advice`, `ask_diana`.

## Procedure

1. **Identify the campaign**: if the user gives a name, resolve the id via `list_campaigns`. Otherwise ask which one.
2. **Performance**: `get_campaign_performance` over the period. Metrikia-attributed ROAS (MTA then CRM), CPA, spend, conversions.
3. **Creatives**: `get_creative_report` for the campaign. Identify winners, losers, fatigue signals (CTR dropping, high frequency).
4. **Attribution**: `get_attribution_journey` to understand the campaign's role in the paths (first touch, assist, last touch). A campaign can be under-credited in last-click yet essential in MTA.
5. **Budget**: `get_budget_advice` for the allocation lever (scale, cut, redistribute).
6. **Synthesis**: `ask_diana` for 2-3 concrete actions.

## Output format

```
Campaign audit: [name]

Performance (Metrikia attribution)
ROAS X.Xx | CPA X EUR | spend X EUR | conversions X

Creatives
Winners: ...
Fatigue: ...

Role in attribution (MTA)
[first touch / assist / last] - X% of converting paths

Recommendations
1. ...
```

## Pitfalls

- ROAS = Metrikia attribution only. A campaign that looks "bad" in last-click can be key in MTA (assist): always cross-check with `get_attribution_journey` before recommending to cut it.
- Check `get_sync_status` if the numbers look incomplete.

## Verification

The audit cross-checks at least performance + creatives + attribution. No "cut the campaign" recommendation without first verifying its assist role in MTA.
