---
name: metrikia-budget-alert
description: Surveillance autonome budget et anomalies ads via Metrikia. Detecte depense aberrante, ROAS qui decroche, creative fatigue, et alerte uniquement si action requise. Tourne en cron via blueprint, livre au home channel.
version: 1.0.0
author: Alexandre Mallet / BULDEE
license: Proprietary
platforms: [macos, linux]
metadata:
  hermes:
    tags: [Marketing, Monitoring]
    blueprint:
      schedule: "0 */6 * * *"
required_environment_variables:
  - name: METRIKIA_API_KEY
    prompt: "Cle API Metrikia (mk_live_...) pour le serveur MCP Metrikia"
---

# Alerte budget et anomalies Metrikia

Veille autonome : ne parle que si quelque chose merite une action. Pas de bruit.

## Pre-requis

Serveur MCP Metrikia configure. Outils : `get_anomalies`, `get_metrics`, `get_budget_advice`, `get_sync_status`.

## Quand l'utiliser

- En autonomie via blueprint (toutes les 6h).
- A la demande : "verifie mes alertes ads".

## Procedure

1. **Sync** : `get_sync_status`. Si la donnee est en retard de plus de quelques heures, signaler le retard de sync et s'arreter (pas d'alerte sur donnee stale).
2. **Anomalies** : `get_anomalies` sur les 24 dernieres heures. Filtrer sur severite reelle (ROAS qui decroche sous un seuil, depense qui explose, creative fatigue confirmee).
3. **Budget** : si une anomalie touche l'allocation, `get_budget_advice` pour la reco concrete (couper, plafonner, redistribuer).
4. **Decision de notification** :
   - Rien de significatif : NE PAS notifier (silence = bon signe).
   - Action requise : alerte courte + reco actionnable.

## Format d'alerte (seulement si action requise)

```
Alerte ads Metrikia

[campagne/levier]: [probleme] (attribution Metrikia)
Impact: ...
Action reco: ...
```

## Pieges

- Seuil de bruit eleve : mieux vaut rater une micro-variation que spammer. Une alerte = une action.
- Ne jamais alerter sur un ROAS plateforme. Toujours l'attribution Metrikia.
- Pas d'alerte sur donnee stale (verifier le sync d'abord).

## Verification

Si rien d'actionnable, l'execution se termine sans notification. Toute alerte emise contient une action concrete, pas juste un constat.
