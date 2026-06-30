# Audit log

Every sensitive action (creation / deletion / privilege escalation,
adding a cloud connector, licence change, account
deletion…) is recorded in the `audit_logs` table and viewable via
`/admin/audit`.

## Recorded fields

| Field          | Content                                                  |
|----------------|----------------------------------------------------------|
| `id`           | internal id (auto-increment)                             |
| `created_at`   | UTC timestamp                                            |
| `actor_user_id`| user who performed the action (or `null` = system)       |
| `actor_ip`     | source IP (from `X-Forwarded-For` behind the proxy)      |
| `action`       | action code — e.g. `user.create`, `org.delete`, `oauth.attach`, `license.activate` |
| `target_type`  | type of target object (`user`, `org`, `cloud_connection`…)|
| `target_id`    | id of the target object                                  |
| `metadata`     | free-form JSON (e.g. `{"old_role": "member", "new_role": "admin"}`) |

**Pseudonymisation**: no email, no RAG query content is stored in
the log. Only numeric `user_id`s are kept. See `app/utils/logger.py`.

## Search & filters

`/admin/audit` provides:

- Filter by action (autocomplete)
- Filter by user (id or email — resolved to id at submission)
- Date range
- CSV export (capped at 10,000 rows per export)

## Off-host archival

The `archive_audit_log` cron (Sundays 03:30 UTC) **archives entries
> 180 days** to an S3 bucket (server-side encrypted, Glacier IR
class or equivalent). Archived rows are then **deleted** from the
main table to limit growth.

Configuration via `.env`:

```bash
AUDIT_S3_BUCKET=myeline-audit-archive
AUDIT_S3_ENDPOINT=https://s3.fr-par.scw.cloud
AUDIT_S3_ACCESS_KEY=…
AUDIT_S3_SECRET_KEY=…
AUDIT_S3_RETENTION_DAYS=180   # default: 180
```

Without `AUDIT_S3_BUCKET` the cron is a no-op (no data loss, just
no archiving). In pure sovereign, point at an internal MinIO rather
than a cloud S3.

## Legal retention

13 months active + archive up to 5 years for actions related to
personal data (GDPR, art. 30 — processing registry). Adjust
`AUDIT_S3_RETENTION_DAYS` upward depending on your internal policy.
