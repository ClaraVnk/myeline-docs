# Quotas et plans

En éditions on-prem (souverain et souverain-hybride), la **licence
couvre l'opérateur** — il n'y a pas de notion de plan utilisateur
interne. Toutes les fonctionnalités sont activées : multi-turn,
conversations, alertes, historique, single-document chat.

Les quotas existent quand même, mais comme **garde-fous techniques**
(éviter qu'un utilisateur monopolise le serveur), pas comme levier
commercial.

## Quotas par défaut

| Métrique                   | Limite             | Périmètre        |
|----------------------------|--------------------|------------------|
| Requêtes RAG               | 300 / jour         | par utilisateur  |
| Sources RSS / scrapers     | 50 / org           | par organisation |
| Documents personnels       | 10 GB / utilisateur| par utilisateur  |
| Connexions cloud actives   | 10 / utilisateur   | par utilisateur  |
| Historique conversations   | illimité           | —                |

Les valeurs sont définies dans `app/services/quotas.py` et
peuvent être surchargées via variables d'environnement (voir
[Pré-requis serveur](../install/prerequisites.md)).

## Surveillance

Le cron `check_quotas` s'exécute quotidiennement à 04:00 UTC :

- **80 %** d'usage atteint → email d'avertissement au owner de l'org
- **100 %** atteint → blocage soft (HTTP 429 jusqu'au lendemain)
- En souverain pur, le mailer est en log-only — vérifier
  `logs/mailer/` ou brancher un MTA interne.

## Surcharge admin

Un admin global peut **lever ponctuellement** un quota depuis
`/admin/users/<id>` (champ « override quota », expire après 24 h).
Action tracée dans le journal d'audit.

## Métriques exposées

`/metrics` (Prometheus) expose :

- `myeline_quota_usage{user_id, metric}` (gauge)
- `myeline_rag_queries_total{user_id, status}` (counter)
- `myeline_quota_blocks_total` (counter)

Brancher Grafana ou votre stack obs pour les dashboards.
