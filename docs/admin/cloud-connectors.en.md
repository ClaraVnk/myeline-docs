# Cloud connectors

Cloud connectors let a user (or an organisation) **automatically
sync** an external drive into the ChromaDB collection of their
scope — indexed files become RAG-queryable.

## Connectors available per edition

| Connector  | Sovereign (air-gap) | Sovereign-hybrid |
|------------|---------------------|------------------|
| **S3 / S3-compatible** (internal MinIO) | ✅ | ✅ |
| **WebDAV** (internal Nextcloud) | ✅ | ✅ |
| **Google Drive** | ❌ | ✅ (GCP service account) |
| **OneDrive / SharePoint** | ❌ | ✅ (BYOC OAuth) |
| **Dropbox** | ❌ | ✅ (BYOC OAuth) |
| **kDrive Infomaniak** | ❌ | ✅ (API token) |
| **Zotero** | ❌ | ✅ (API key) |
| **Notion** | ❌ | ✅ (BYOC OAuth) |

In pure sovereign, only **S3 and WebDAV toward internal infra** are
exposed — buttons for other connectors are hidden and the
corresponding routes return 404.

## Google Drive — service account

Google Drive uses a **GCP service account** (server-to-server).
Sharing model:

1. The operator creates a service account in their GCP project (full
   procedure: [Sovereign-hybrid installation § Google Drive](../install/sovereign-hybrid.md#google-drive-service-account)).
2. The JSON is pasted into `GOOGLE_SERVICE_ACCOUNT_CREDENTIALS` in `.env`.
3. **Each user** who wants to index a folder shares it as *Viewer* with
   the service account email, then pastes the folder URL into
   `/user/cloud/gdrive/connect`.
4. The service account only sees the folders explicitly shared by
   users — not the rest of their Drive.

Properties of this model:

- ✅ Continuous background indexing without periodic token renewal
  (no access token to refresh, no OAuth scope to renew on the user side)
- ✅ One GCP app for the whole tenant (vs. one app per user or per
  deployment)
- ✅ Granular revocation: the user removes the folder share from the
  Drive UI, Myeline's access drops instantly
- ✅ GCP-auditable: all service account accesses appear in the
  project's Cloud Audit Logs, traceable at the file level

## BYOC — Bring Your Own Credentials (OneDrive, Dropbox, Notion)

For the remaining OAuth connectors, you **must register your own OAuth
apps** with Microsoft / Dropbox / Notion. See the full walkthrough:
[Sovereign-hybrid installation § BYOC](../install/sovereign-hybrid.md#byoc).

**Why this is mandatory**: the OAuth redirect URI points to
`https://<your-domain>/user/cloud/<provider>/callback`. Providers
check this redirect against the whitelist of the app that initiated
the flow — myeline.io's OAuth app cannot serve your users.

## Sync frequency

`check_cloud_sync` cron (every 4 hours), with a **per-licence floor**
(from `app/cron/check_cloud_sync.py`):

| Tier          | Minimum interval between two syncs |
|---------------|------------------------------------|
| Free          | 24 h (once per day)                |
| Pro           | 1 h (configurable, up to hourly)   |
| On-premise / Sovereign | Custom (down to 15 min)            |

The owner can trigger an on-demand sync from `/user/cloud` (capped
at 1/h regardless of tier).

## Credential storage

- **OAuth tokens** (access + refresh): `cloud_connections.token_enc`
  encrypted via Fernet (`CLOUD_TOKEN_KEY`).
- **Static API keys** (kDrive, Zotero, S3): same mechanism.
- **No key ever appears in logs** or in the audit log.

See `app/utils/crypto.py` for implementation details.

## Disabling a connector globally

To forbid a connector across the entire deployment (even in
sovereign-hybrid), add to `.env`:

```bash
CLOUD_CONNECTORS_DISABLED=dropbox,notion
```

Comma-separated list. Buttons disappear, routes return 404, existing
syncs are frozen.
