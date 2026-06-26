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
| `metrikia-ops-health` | Read-only operational-health watch (ad disapprovals, delivery stopped, account flags) via a read-only provider MCP. Alerts only, never writes | blueprint every 4h |

## Agentic boundary (read this)

This tap runs inside an **autonomous, always-on agent**. That changes what it is allowed to do compared to a human-driven tool.

```
Autonomous agent (Hermes, these skills)  =  OBSERVE + PROPOSE. Never a budget actuator.
Human-driven tool (Metrikia Claude plugin /metrikia:ads-ops)  =  EXECUTE writes.
```

These skills **never** pause, scale, or change a budget. An autonomous agent with write access to ad accounts is a money-risk (a misread message or a cron run could move real spend). So the skills read attributed truth (Metrikia) and operational health (read-only provider MCP), and when an action is warranted they **recommend it and point to the human tool** to execute it. Do not wire a write-capable provider MCP into Hermes for these skills. Write belongs to the human-in-the-loop plugin.

## Principle

Every ROAS figure comes from **Metrikia attribution (MTA then CRM)**, never the platform self-reported numbers. The skills are built for **delegation**: they run on a schedule (blueprint), deliver to the home channel, and only notify when action is required.

## License

Apache-2.0. (c) Alexandre Mallet / BULDEE.
