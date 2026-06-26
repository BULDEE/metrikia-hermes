# metrikia-hermes

Tap de skills Hermes Agent pour piloter **Metrikia** (vrai ROAS attribue MTA, ads, attribution, Diana AI) en autonomie depuis Hermes.

C'est un **tap** : un repo GitHub de fichiers SKILL.md, sans serveur ni registry. Les skills s'appuient sur le **serveur MCP Metrikia** branche dans Hermes.

## Pre-requis

1. Hermes Agent ([Nous Research](https://github.com/nousresearch/hermes-agent)).
2. Une cle API Metrikia (`mk_live_...`) depuis app.metrikia.io/settings/api.
3. Le serveur MCP Metrikia dans `~/.hermes/config.yaml` :

```yaml
mcp_servers:
  metrikia:
    url: https://mcp.metrikia.io/api/v1/mcp
    headers:
      Authorization: "Bearer ${METRIKIA_API_KEY}"
```

Et `METRIKIA_API_KEY` dans l'environnement Hermes.

## Installer

```bash
hermes skills tap add BULDEE/metrikia-hermes
hermes skills browse                       # voir les skills dispo
hermes skills install BULDEE/metrikia-hermes/metrikia-weekly-report
```

## Skills

| Skill | Role | Autonomie |
|-------|------|-----------|
| `metrikia-weekly-report` | Rapport hebdo (MER, ROAS attribue, top campagnes, anomalies, reco Diana) | blueprint lundi 8h |
| `metrikia-campaign-audit` | Audit deep-dive d'une campagne (perf, creatives, attribution MTA, leviers) | a la demande |
| `metrikia-budget-alert` | Veille budget/anomalies, alerte seulement si action requise | blueprint /6h |

## Principe

Tous les chiffres ROAS viennent de l'**attribution Metrikia (MTA puis CRM)**, jamais des chiffres plateforme auto-declares. Les skills sont concus pour la **delegation** : ils tournent en cron (blueprint), livrent au home channel, et ne notifient que si une action est requise.

## Licence

Proprietary. (c) Alexandre Mallet / BULDEE.
