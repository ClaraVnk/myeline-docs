# Backup and restore

Myeline runs with **three sources of truth** to back up:

1. **MariaDB** — accounts, organisations, document metadata, audit
   log, conversations.
2. **ChromaDB** — embedding vectors + metadata (sources of RAG
   answers).
3. **File volumes** — user uploads (`data/uploads/`), logs (`logs/`),
   custom static assets (`static/custom/`).

Without all three, a restore is incomplete.

## Daily automatic backup

The `backup_databases` cron runs every day at **02:30 UTC** (off-peak,
adjustable in `scripts/crontab`). It produces in
`/var/lib/myeline/backups/`:

```
backups/
├── mariadb-2026-05-06.sql.gz       # compressed mysqldump
├── chroma-2026-05-06.tar.zst       # ChromaDB snapshot
└── volumes-2026-05-06.tar.zst      # uploads + custom logs
```

**Local retention**: 14 days (variable `BACKUP_RETENTION_DAYS`).
Beyond that, files are removed from the host.

## Off-host backup (recommended)

For real resilience, **synchronise** `/var/lib/myeline/backups/` to
**separate** storage (different datacenter, different vendor).

### Option A — `rclone` (sovereign-hybrid only)

```bash
# Interactive configuration
rclone config
# Then in /etc/cron.d/myeline-backup-offsite:
30 3 * * *  root  rclone sync /var/lib/myeline/backups remote:myeline-backups --transfers=4 --checksum
```

### Option B — Internal MinIO (pure sovereign)

```bash
mc alias set local-minio https://minio.dmz.local:9000 ACCESS SECRET
30 3 * * *  root  mc mirror /var/lib/myeline/backups local-minio/myeline-backups
```

### Option C — `borgbackup` (client-side encryption)

Recommended if the target is partially trusted — client AES-256
encryption + deduplication.

## Full restore

```bash
# 1. Stop the stack
podman-compose down

# 2. Restore MariaDB
zcat backups/mariadb-2026-05-06.sql.gz | podman exec -i myeline-db mariadb -u root -p$DB_ROOT myeline

# 3. Restore ChromaDB
tar --use-compress-program=unzstd -xf backups/chroma-2026-05-06.tar.zst -C data/

# 4. Restore the volumes
tar --use-compress-program=unzstd -xf backups/volumes-2026-05-06.tar.zst -C data/

# 5. Restart
podman-compose up -d
podman-compose exec web flask db upgrade
```

## Restore drill

**Strong recommendation**: test the restoration procedure on a clean
environment **at least once per quarter**. An untested backup is no
backup.

## Secret rotation after a leak

If a backup leaked, **rotate immediately**:

- `SECRET_KEY` (logs out all sessions)
- `CLOUD_TOKEN_KEY` (invalidates all encrypted OAuth tokens — users
  will need to reconnect their drives)
- DB and Redis passwords
- External API tokens (Mistral, Anthropic, OpenAI, Gemini)
- Licence key (contact Myeline for re-issuance)

See [Licence errors](../troubleshooting/license-errors.md) for the
full licence rotation flow.
