# Sync failures

Cloud connectors can fail for very different reasons depending on
the provider. This page catalogues the most common symptoms.

## General diagnostic

```bash
# Connection status
podman exec myeline-web flask shell << 'EOF'
from app.models.cloud_connection import CloudConnection
for c in CloudConnection.query.all():
    print(f"{c.id} {c.provider} user={c.user_id} status={c.status} last_sync={c.last_sync_at} last_error={c.last_error}")
EOF

# Cron logs
podman exec myeline-cron cat /var/log/cron/check_cloud_sync.log | tail -50

# Force a manual sync
podman exec myeline-web flask run-cron check_cloud_sync
```

## OAuth refused / `redirect_uri_mismatch`

**Symptom**: "OAuth connection error", redirect loop.

**Cause**: the redirect URI declared in your OAuth app (Google
Console / Azure / Dropbox / etc.) doesn't match what Myeline
generates.

**Fix**:

1. Verify the URL generated:

   ```bash
   podman exec myeline-web flask shell -c "
   from flask import url_for
   print(url_for('user_cloud.gdrive_callback', _external=True))
   "
   ```

2. Copy-paste this exact value into the "Authorised redirect URIs"
   list of the OAuth app on the provider side.

3. Verify `OAUTH_REDIRECT_BASE_URL` in `.env` — must point at the
   exact public URL (e.g. `https://myeline.acme.local`), not a
   private IP.

## Token expired and not refreshed

**Symptom**: `last_error: "401 Unauthorized"`, `status: error`.

**Cause**: the refresh token was revoked (provider-side password
changed, app deauthorised, inactivity period exceeded).

**Fix**: the user must **reconnect their drive** from `/user/cloud`
("Reconnect" button next to the connection in error). Sync history
is preserved.

## 429 / Rate limit

**Symptom**: `last_error: "429 Too Many Requests"`.

**Cause**: too many requests to the provider's API — typically on
a multi-hundred-GB drive with forced sync.

**Fix**:

- Wait — the sync resumes at the next cron (4 h).
- Reduce frequency on the connection ("Sync interval" parameter on
  `/user/cloud`).
- Make sure no other app is exhausting the same Google / Microsoft
  account's quota.

## Files silently ignored

**Symptom**: a file is in the drive but doesn't appear in the
library.

**Possible causes**:

- **Unsupported format** (XLSX, PPTX, ZIP, video, image…). See the
  list of indexed formats in
  [Library and uploads](../usage/upload-documents.md).
- **Size > 50 MB** (silent reject indexer-side).
- **Provider-side encrypted document** (Drive "Confidentiality
  protected", SharePoint "Information rights management"). The API
  refuses download.
- **Trash / recycle bin**: trashed files are never indexed.
- **Folder filter**: if the user picked a specific subfolder when
  connecting, files outside that folder are ignored.

Verification:

```bash
# Detailed worker logs during a sync
podman exec myeline-worker rq info
podman logs --tail 200 myeline-worker | grep -i "skip\|ignore\|reject"
```

## Embedding failure

**Symptom**: the sync ends with success but the file stays at "In
progress" indefinitely.

**Cause**: Ollama unreachable when the worker processed the file.
The worker does 3 retries then gives up — the file is in the queue
but not embedded.

**Fix**:

```bash
# Re-queue
podman exec myeline-worker rq requeue --queue myeline-pro --all
podman exec myeline-worker rq requeue --queue myeline --all
```

Then check Ollama state (see [Ollama issues](ollama-issues.md)).

## Standard tier — limits

**Standard**-tier accounts / licences have strict limits:

- **One** active cloud connection only.
- **One sync per 24 h**, regardless of cron cadence.

The `/user/cloud` progress banner explicitly shows the next allowed
slot. No way to bypass without a plan / licence upgrade.
