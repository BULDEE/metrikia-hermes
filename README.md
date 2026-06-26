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
| `metrikia-budget-alert` | Budget/anomaly watch, alerts only when action is required | blueprint every 6h |

## Principle

Every ROAS figure comes from **Metrikia attribution (MTA then CRM)**, never the platform self-reported numbers. The skills are built for **delegation**: they run on a schedule (blueprint), deliver to the home channel, and only notify when action is required.

## License

Apache-2.0. (c) Alexandre Mallet / BULDEE.
