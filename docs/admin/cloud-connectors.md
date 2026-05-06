# Connecteurs cloud

Les connecteurs cloud permettent à un utilisateur (ou à une
organisation) de **synchroniser automatiquement** un drive externe
vers la collection ChromaDB de son périmètre — les fichiers indexés
deviennent interrogeables en RAG.

## Connecteurs disponibles par édition

| Connecteur | Souverain (air-gap) | Souverain-hybride |
|------------|---------------------|-------------------|
| **S3 / S3-compatible** (MinIO interne) | ✅ | ✅ |
| **WebDAV** (Nextcloud interne) | ✅ | ✅ |
| **Google Drive** | ❌ | ✅ (BYOC) |
| **OneDrive / SharePoint** | ❌ | ✅ (BYOC) |
| **Dropbox** | ❌ | ✅ (BYOC) |
| **kDrive Infomaniak** | ❌ | ✅ |
| **Zotero** | ❌ | ✅ |
| **Notion** | ❌ | ✅ (BYOC) |

En souverain pur, seuls **S3 et WebDAV vers infra interne** sont
exposés — les boutons des autres connecteurs sont masqués et les
routes correspondantes renvoient 404.

## BYOC — Bring Your Own Credentials

En souverain-hybride, vous **devez enregistrer vos propres apps
OAuth** chez Google / Microsoft / Dropbox / Notion. Voir le
walkthrough complet :
[Installation souverain-hybride § BYOC](../install/sovereign-hybrid.md#byoc).

**Pourquoi c'est obligatoire** : le redirect URI OAuth pointe vers
`https://<votre-domaine>/user/cloud/<provider>/callback`. Les
providers vérifient ce redirect contre la whitelist de l'app qui
initie le flow — l'app OAuth de myeline.io ne peut pas servir vos
utilisateurs.

## Fréquence de synchronisation

Cron `check_cloud_sync` (toutes les 4 heures), avec **plancher par
licence** (depuis `app/cron/check_cloud_sync.py`) :

| Licence       | Intervalle minimum entre 2 syncs |
|---------------|----------------------------------|
| Standard      | 24 h (1 fois par jour)           |
| Pro           | 4 h                              |
| Team          | 1 h                              |
| Enterprise    | 15 min                           |

Le owner peut déclencher une sync à la demande depuis
`/user/cloud` (limité à 1 fois/heure quel que soit le tier).

## Stockage des credentials

- **OAuth tokens** (access + refresh) : `cloud_connections.token_enc`
  chiffré via Fernet (clé `CLOUD_TOKEN_KEY`).
- **API keys statiques** (kDrive, Zotero, S3) : même mécanisme.
- **Aucune clé n'apparaît dans les logs** ni dans le journal d'audit.

Voir `app/utils/crypto.py` pour les détails d'implémentation.

## Désactiver un connecteur globalement

Pour interdire un connecteur sur tout le déploiement (même en
souverain-hybride), ajouter à `.env` :

```bash
CLOUD_CONNECTORS_DISABLED=dropbox,notion
```

Liste séparée par virgule. Les boutons disparaissent, les routes
renvoient 404, les syncs déjà en place sont gelées.
