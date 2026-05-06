# Upgrade

On-prem editions receive Myeline updates via an **OCI image**
published to our registry. No automatic updates: you decide when.

## Release cycle

- **Patch** (`x.y.Z`): security fixes + critical bugs — every 1-2
  weeks.
- **Minor** (`x.Y.0`): new features, backward-compatible DB
  migrations — every 1-2 months.
- **Major** (`X.0.0`): breaking changes (rare, announced 30 days
  ahead).

An email lands at the admin of each deployment when a patch / minor
ships (in pure sovereign, log-only — subscribe humans to the
`releases@myeline.io` mailing list instead).

## Standard procedure

```bash
cd /opt/myeline

# 1. Backup first (the cron ran last night, but a fresh backup
#    avoids losing the day's changes).
podman-compose exec web flask run-cron backup_databases

# 2. Pull the new image
podman-compose pull web cron worker

# 3. Recreate the containers
podman-compose up -d

# 4. Apply DB migrations (idempotent — no-op if already up to date)
podman-compose exec web flask db upgrade

# 5. Health check
curl -s https://your-domain/health | jq
```

`/health` must return 200 with all components `ok`. If one is
`degraded` or `down`, see [Troubleshooting](../troubleshooting/index.md).

## Rollback

Every image is tagged with its version (`v2026.05.01`,
`v2026.04.20`, etc.) — `latest` is just a moving alias. To revert
to the previous version:

```bash
# 1. Edit .env and pin the explicit tag
echo 'MYELINE_VERSION=v2026.04.20' >> .env

# 2. Recreate
podman-compose up -d

# 3. If a downward migration is needed:
podman-compose exec web flask db downgrade
```

⚠️ A schema downgrade can be **destructive** if the upward version
added required columns. Prefer restoration from backup (see
[Backup and restore](backup-restore.md)) if in doubt.

## Multi-step migrations

For risky changes (column drops, mass renames) we apply the
**expand → migrate → contract** pattern across two releases:

1. Release N: add the new column / structure (old one stays in place).
2. Silent backfill in application code.
3. Release N+1: remove the old structure.

No upgrade should ever require > 30 seconds downtime (the time of
`podman-compose up -d`). The `flask db upgrade` migrations are
designed to apply in seconds even on multi-GB databases.

## Licence update

Unrelated to image upgrades. See
[Licence renewal](license-renewal.md).
