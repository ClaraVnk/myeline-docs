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
| **Google Drive** | ❌ | ✅ (service account GCP) |
| **OneDrive / SharePoint** | ❌ | ✅ (BYOC OAuth) |
| **Dropbox** | ❌ | ✅ (BYOC OAuth) |
| **kDrive Infomaniak** | ❌ | ✅ (API token) |
| **Zotero** | ❌ | ✅ (API key) |
| **Notion** | ❌ | ✅ (BYOC OAuth) |

En souverain pur, seuls **S3 et WebDAV vers infra interne** sont
exposés — les boutons des autres connecteurs sont masqués et les
routes correspondantes renvoient 404.

## Google Drive — compte de service

Google Drive utilise un **service account GCP** (server-to-server).
Modèle de partage :

1. L'opérateur crée un service account dans son projet GCP (procédure
   complète : [Installation souverain-hybride § Google Drive](../install/sovereign-hybrid.md#google-drive-compte-de-service)).
2. Le JSON est collé dans `GOOGLE_SERVICE_ACCOUNT_CREDENTIALS` du `.env`.
3. **Chaque utilisateur** qui veut indexer un dossier le partage en
   *Lecteur* avec l'email du service account, puis colle l'URL du
   dossier dans `/user/cloud/gdrive/connect`.
4. Le service account voit **uniquement** les dossiers partagés
   explicitement par les utilisateurs — pas le reste de leur Drive.

Propriétés de ce modèle :

- ✅ Indexation continue en arrière-plan sans renouvellement périodique
  de tokens (pas d'access token à refresh, pas de scope OAuth à
  renouveler côté utilisateur)
- ✅ Une seule app GCP pour toute la tenant (vs. une app par utilisateur
  ou par déploiement)
- ✅ Révocation granulaire : l'utilisateur retire le partage du dossier
  côté Drive UI, l'accès tombe instantanément côté Myeline
- ✅ Auditable côté GCP : tous les accès du service account apparaissent
  dans les Cloud Audit Logs du projet, traçables au fichier près

## BYOC — Bring Your Own Credentials (OneDrive, Dropbox, Notion)

Pour les connecteurs OAuth restants, vous **devez enregistrer vos
propres apps OAuth** chez Microsoft / Dropbox / Notion. Voir le
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

| Tier          | Intervalle minimum entre 2 syncs |
|---------------|----------------------------------|
| Free          | 24 h (1 fois par jour)           |
| Pro           | 1 h (configurable, jusqu'à toutes les heures) |
| Enterprise    | Sur mesure (jusqu'à 15 min)      |

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
