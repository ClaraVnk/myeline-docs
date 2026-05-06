# Journal d'audit

Toute action sensible (création / suppression / élévation de
privilèges, ajout d'un connecteur cloud, changement de licence,
configuration OIDC, suppression de compte…) est tracée dans la
table `audit_logs` et consultable via `/admin/audit`.

## Champs enregistrés

| Champ          | Contenu                                                  |
|----------------|-----------------------------------------------------------|
| `id`           | identifiant interne (auto-incrément)                      |
| `created_at`   | timestamp UTC                                             |
| `actor_user_id`| utilisateur ayant exécuté l'action (ou `null` = système)  |
| `actor_ip`     | IP source (depuis `X-Forwarded-For` derrière le proxy)    |
| `action`       | code action — ex. `user.create`, `org.delete`, `oauth.attach`, `license.activate` |
| `target_type`  | type de l'objet ciblé (`user`, `org`, `cloud_connection`…) |
| `target_id`    | identifiant de l'objet ciblé                              |
| `metadata`     | JSON libre (ex. `{"old_role": "member", "new_role": "admin"}`) |

**Pseudonymisation** : aucun email, aucun contenu de requête RAG
n'est stocké dans le journal. Seuls les `user_id` numériques sont
conservés. Voir `app/utils/logger.py`.

## Recherche & filtres

`/admin/audit` propose :

- Filtre par action (autocomplete)
- Filtre par utilisateur (id ou email — résolu en id à l'envoi)
- Plage de dates
- Export CSV (limité à 10 000 lignes par export)

## Archivage off-host

Le cron `archive_audit_log` (dimanche 03:30 UTC) **archive les
entrées > 180 jours** vers un bucket S3 (chiffré côté serveur,
classe Glacier IR ou équivalent). Les lignes archivées sont
ensuite **supprimées** de la table principale pour limiter la
croissance.

Configuration via `.env` :

```bash
AUDIT_S3_BUCKET=myeline-audit-archive
AUDIT_S3_ENDPOINT=https://s3.fr-par.scw.cloud
AUDIT_S3_ACCESS_KEY=…
AUDIT_S3_SECRET_KEY=…
AUDIT_S3_RETENTION_DAYS=180   # défaut : 180
```

Sans `AUDIT_S3_BUCKET`, le cron est no-op (aucune perte de données,
juste pas d'archivage). En souverain pur, pointer vers un MinIO
interne plutôt qu'un S3 cloud.

## Durée légale de conservation

13 mois côté actif + archive jusqu'à 5 ans pour les actions liées
à des données personnelles (RGPD, art. 30 — registre des
traitements). Adapter `AUDIT_S3_RETENTION_DAYS` au-delà selon votre
politique interne.
