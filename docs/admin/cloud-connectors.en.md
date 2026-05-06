# Cloud connectors

Cloud connectors let a user (or an organisation) **automatically
sync** an external drive into the ChromaDB collection of their
scope — indexed files become RAG-queryable.

## Connectors available per edition

| Connector  | Sovereign (air-gap) | Sovereign-hybrid |
|------------|---------------------|------------------|
| **S3 / S3-compatible** (internal MinIO) | ✅ | ✅ |
| **WebDAV** (internal Nextcloud) | ✅ | ✅ |
| **Google Drive** | ❌ | ✅ (BYOC) |
| **OneDrive / SharePoint** | ❌ | ✅ (BYOC) |
| **Dropbox** | ❌ | ✅ (BYOC) |
| **kDrive Infomaniak** | ❌ | ✅ |
| **Zotero** | ❌ | ✅ |
| **Notion** | ❌ | ✅ (BYOC) |

In pure sovereign, only **S3 and WebDAV toward internal infra** are
exposed — buttons for other connectors are hidden and the
corresponding routes return 404.

## BYOC — Bring Your Own Credentials

In sovereign-hybrid, you **must register your own OAuth apps** with
Google / Microsoft / Dropbox / Notion. See the full walkthrough:
[Sovereign-hybrid installation § BYOC](../install/sovereign-hybrid.md#byoc).

**Why this is mandatory**: the OAuth redirect URI points to
`https://<your-domain>/user/cloud/<provider>/callback`. Providers
check this redirect against the whitelist of the app that initiated
the flow — myeline.io's OAuth app cannot serve your users.

## Sync frequency

`check_cloud_sync` cron (every 4 hours), with a **per-licence floor**
(from `app/cron/check_cloud_sync.py`):

| Licence       | Minimum interval between two syncs |
|---------------|------------------------------------|
| Standard      | 24 h (once per day)                |
| Pro           | 4 h                                |
| Team          | 1 h                                |
| Enterprise    | 15 min                             |

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
