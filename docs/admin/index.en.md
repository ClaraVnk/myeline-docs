# Administration

!!! info "On-premise / Sovereign — quote-based"
    The multi-tenant administration described here applies to
    **on-premise** deployments (**bespoke, quote-based**
    installations). On the [Cloud / Hosted](../cloud/index.md) offer,
    team management happens from your dashboard (per-seat billing), with
    no server admin interface.

The Myeline admin interface is reachable at
`https://<your-domain>/admin` for accounts flagged `is_admin`.

The `/admin` dashboard centralises:

- Platform state (users, organisations, RQ queues, ChromaDB, Ollama,
  mailer)
- Licence banner (edition, expiry, status)
- RAG indicators (query volume, errors)
- Quick links to detailed sections

## Sections covered

- **[Organisations and users](organizations.md)** — create an
  organisation, invite members, manage owner / admin / member roles,
  delete an account (GDPR).
- **Enterprise SSO** — deployed on request for on-premise installs —
  contact hello@myeline.io.
- **[Quotas and plans](quotas.md)** — per-plan limits, 80 % / 100 %
  alerts, admin override.
- **[Audit log](audit-log.md)** — who did what, when, off-host
  archival of entries > 180 days old.
- **[Cloud connectors](../concepts/cloud-connectors.md)** — activation
  per edition, BYOC credentials configuration in sovereign-hybrid.

## Best practices

- Create **at least two admin accounts** (resilience in case of
  password loss).
- Enable **TOTP 2FA** for all admins (`/account/2fa`).
- Check `/admin` at least weekly, or wire `/health` to your
  monitoring (Uptime Kuma, Zabbix, Prometheus…).
- Keep a **regular off-site export** of the audit log (see
  [Operations / backup](../operations/backup-restore.md)).
