# metrikia-hermes

A Hermes Agent skills tap to drive **Metrikia** (true MTA-attributed ROAS, ads, attribution, Diana AI) autonomously from Hermes.

This is a **tap**: a GitHub repo of SKILL.md files, no server and no registry. The skills rely on the **Metrikia MCP server** wired into Hermes.

## Requirements

1. Hermes Agent ([Nous Research](https://github.com/nousresearch/hermes-agent)).
2. A Metrikia API key (`mk_live_...`) from app.metrikia.io/settings/api.
3. The Metrikia MCP server in `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  metrikia:
    url: https://mcp.metrikia.io/api/v1/mcp
    headers:
      Authorization: "Bearer ${METRIKIA_API_KEY}"
```

And `METRIKIA_API_KEY` in the Hermes environment.

## Install

```bash
hermes skills tap add BULDEE/metrikia-hermes
hermes skills browse                       # list available skills
hermes skills install BULDEE/metrikia-hermes/metrikia-weekly-report
```

## Skills

| Skill | Role | Autonomy |
|-------|------|----------|
| `metrikia-weekly-report` | Weekly report (MER, attributed ROAS, top campaigns, anomalies, Diana recs) | blueprint Monday 8am |
| `metrikia-campaign-audit` | Deep-dive audit of one campaign (perf, creatives, MTA attribution, levers) | on demand |
| `metrikia-budget-alert` | Budget/anomaly watch (performance), alerts only when action is required | blueprint every 6h |
| `metrikia-ops-health` | Operational-health watch via Metrikia (campaign status changes, inactive creatives, sync lag, anomalies across all platforms). Alerts only, never writes | blueprint every 4h |

## Agentic boundary (read this)

This tap runs inside an **autonomous, always-on agent**. That changes what it is allowed to do compared to a human-driven tool.

```
Autonomous agent (Hermes, these skills)  =  OBSERVE + PROPOSE. Never a budget actuator.
Human-driven tool (Metrikia Claude plugin /metrikia:ads-ops)  =  EXECUTE writes.
```

These skills **never** pause, scale, or change a budget. An autonomous agent with write access to ad accounts is a money-risk (a misread message or a cron run could move real spend). They read everything from **Metrikia**, which already ingests all provider data (Meta, Google, TikTok) and serves both attributed performance and operational state. So Hermes needs only the Metrikia MCP for reads: no provider MCP is wired here, and Metrikia exposes no ad-budget write tool, which makes the budget-actuator boundary structural rather than just a convention. When an action is warranted, the skills **recommend it and point to the human tool** (the Metrikia Claude plugin `/metrikia:ads-ops`, or the platform UI) to execute it. Provider MCPs (write) belong to the human-in-the-loop plugin, not here.

## Principle

Every ROAS figure comes from **Metrikia attribution (MTA then CRM)**, never the platform self-reported numbers. The skills are built for **delegation**: they run on a schedule (blueprint), deliver to the home channel, and only notify when action is required.

## License

Apache-2.0. (c) Alexandre Mallet / BULDEE.
