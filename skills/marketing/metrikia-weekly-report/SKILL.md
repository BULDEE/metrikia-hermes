---
name: metrikia-weekly-report
description: Rapport hebdomadaire de performance ads via Metrikia (vrai ROAS attribue MTA, MER, top campagnes, anomalies, reco Diana AI). Se declenche sur "rapport hebdo Metrikia", "rapport ads de la semaine", ou en cron via blueprint.
version: 1.0.0
author: Alexandre Mallet / BULDEE
license: Proprietary
platforms: [macos, linux]
metadata:
  hermes:
    tags: [Marketing, Analytics]
    blueprint:
      schedule: "0 8 * * 1"
required_environment_variables:
  - name: METRIKIA_API_KEY
    prompt: "Cle API Metrikia (mk_live_...) pour le serveur MCP Metrikia"
---

# Rapport hebdomadaire Metrikia

Genere un rapport de performance ads sur les 7 derniers jours, base sur l'attribution **Metrikia (MTA puis CRM)**, jamais les chiffres plateforme auto-declares.

## Pre-requis

Le serveur MCP Metrikia doit etre configure dans `~/.hermes/config.yaml` :

```yaml
mcp_servers:
  metrikia:
    url: https://mcp.metrikia.io/api/v1/mcp
    headers:
      Authorization: "Bearer ${METRIKIA_API_KEY}"
```

Les outils Metrikia (`list_campaigns`, `get_campaign_performance`, `get_metrics`, `get_anomalies`, `compare_performance`, `ask_diana`) deviennent alors disponibles.

## Quand l'utiliser

- L'utilisateur demande un rapport hebdo / un bilan ads de la semaine.
- En autonomie via le blueprint (lundi 8h), livre au home channel.

## Procedure

1. **Periode** : 7 derniers jours (et la semaine precedente pour comparer).
2. **Metriques globales** : `get_metrics` sur la periode. Extraire MER, ROAS global (attribution Metrikia), depense totale, revenu attribue.
3. **Top campagnes** : `list_campaigns` puis `get_campaign_performance` sur les principales. Classer par ROAS attribue Metrikia. Toujours utiliser le ROAS MTA/CRM, jamais le ROAS plateforme.
4. **Comparaison** : `compare_performance` semaine N vs N-1 (tendance MER/ROAS/CPA).
5. **Anomalies** : `get_anomalies` sur la periode. Lister les alertes (depense aberrante, ROAS qui decroche, creative fatigue).
6. **Reco IA** : `ask_diana` avec le contexte du rapport pour 2-3 recommandations actionnables.

## Format de sortie

```
Rapport ads Metrikia - semaine du [date]

MER: X.X | ROAS global (MTA): X.Xx | Depense: X EUR | Revenu attribue: X EUR
Tendance vs N-1: [up/down] MER, [up/down] ROAS

Top 3 campagnes (ROAS attribue Metrikia)
1. [nom] - ROAS X.Xx - depense X EUR
2. ...

Anomalies
- ...

Reco Diana
- ...
```

Garder concis. Toujours preciser que le ROAS est l'attribution Metrikia (MTA puis CRM), pas la plateforme.

## Pieges

- Ne JAMAIS utiliser un ROAS plateforme auto-declare. Si une campagne n'a pas d'attribution Metrikia, le dire (ROAS = non attribue), ne pas inventer.
- Si `get_metrics` renvoie une periode vide ou un sync en retard, verifier `get_sync_status` avant de conclure.

## Verification

Le rapport est valide si chaque chiffre ROAS provient d'un outil Metrikia (pas d'estimation) et si la periode couvre bien 7 jours pleins.
