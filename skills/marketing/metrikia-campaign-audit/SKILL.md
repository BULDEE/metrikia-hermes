---
name: metrikia-campaign-audit
description: Audit approfondi d'une campagne ads via Metrikia (performance, breakdown creatives, parcours d'attribution multi-touch, reco). Se declenche sur "audite la campagne X", "deep-dive campagne Metrikia".
version: 1.0.0
author: Alexandre Mallet / BULDEE
license: Proprietary
platforms: [macos, linux]
metadata:
  hermes:
    tags: [Marketing, Analytics]
required_environment_variables:
  - name: METRIKIA_API_KEY
    prompt: "Cle API Metrikia (mk_live_...) pour le serveur MCP Metrikia"
---

# Audit de campagne Metrikia

Deep-dive sur une campagne : performance attribuee, creatives, parcours de conversion, leviers.

## Pre-requis

Serveur MCP Metrikia configure (voir skill `metrikia-weekly-report`). Outils utilises : `list_campaigns`, `get_campaign_performance`, `get_creative_report`, `get_attribution_journey`, `get_budget_advice`, `ask_diana`.

## Procedure

1. **Identifier la campagne** : si l'utilisateur donne un nom, resoudre l'id via `list_campaigns`. Sinon demander laquelle.
2. **Performance** : `get_campaign_performance` sur la periode. ROAS attribue Metrikia (MTA puis CRM), CPA, depense, conversions.
3. **Creatives** : `get_creative_report` pour la campagne. Identifier les winners, les losers, les signaux de fatigue (CTR qui decroche, frequence elevee).
4. **Attribution** : `get_attribution_journey` pour comprendre le role de la campagne dans les parcours (premier touch, assist, dernier touch). Une campagne peut etre sous-creditee en last-click et essentielle en MTA.
5. **Budget** : `get_budget_advice` pour le levier d'allocation (scaler, couper, redistribuer).
6. **Synthese** : `ask_diana` pour 2-3 actions concretes.

## Format de sortie

```
Audit campagne: [nom]

Performance (attribution Metrikia)
ROAS X.Xx | CPA X EUR | depense X EUR | conversions X

Creatives
Winners: ...
Fatigue: ...

Role dans l'attribution (MTA)
[premier touch / assist / last] - X% des parcours convertis

Reco
1. ...
```

## Pieges

- ROAS = attribution Metrikia uniquement. Une campagne "mauvaise" en last-click peut etre cle en MTA (assist) : toujours croiser avec `get_attribution_journey` avant de recommander de la couper.
- Verifier `get_sync_status` si les chiffres semblent incomplets.

## Verification

L'audit croise au moins performance + creatives + attribution. Aucune reco "couper la campagne" sans avoir verifie son role d'assist en MTA.
