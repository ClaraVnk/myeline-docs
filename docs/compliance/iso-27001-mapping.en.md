# ISO 27001 / 27701 mapping

This page shows how Myeline's native features contribute to the
controls of **ISO/IEC 27001:2022 Annex A** (and by extension to
27701 on privacy).

⚠️ **Important**: Myeline is a *brick* in your ISMS. ISO 27001
certification applies to your organisation, not to a product. This
mapping helps your audit team identify which controls are already
served by the platform and which remain on your side.

## Annex A — Organisational controls (clause 5)

| Control | Title | Myeline coverage |
|---------|-------|------------------|
| 5.1     | IS policies | Out of scope (policy = you) |
| 5.10    | Acceptable use | T&Cs + EULA provided |
| 5.15    | Access control | Login, RBAC org (owner/admin/member), Email + password + TOTP (SSO on request) |
| 5.16    | Identity management | Creation / deactivation / automatic purge |
| 5.17    | Authentication info | Argon2id password, optional TOTP |
| 5.18    | Access rights | RBAC model + escalation audit |
| 5.34    | PII | Log pseudonymisation, field-level encryption, right to erasure |

## Annex A — People controls (clause 6)

Out of scope — training and awareness are on your side.

## Annex A — Physical controls (clause 7)

Out of scope — you host Myeline.

## Annex A — Technological controls (clause 8)

| Control | Title | Myeline coverage |
|---------|-------|------------------|
| 8.2     | Privileged access | `/admin/audit` traces escalations |
| 8.3     | Information access restriction | ChromaDB scopes per user / org |
| 8.5     | Secure authentication | Password + TOTP (SSO on request), rate limiting |
| 8.6     | Capacity management | Plan-based quotas, 80 % / 100 % alerts |
| 8.7     | Malware protection | MIME check + optional ClamAV on uploads |
| 8.8     | Technical vulnerabilities | Up-to-date OCI image, audited deps (`pip-audit`) |
| 8.9     | Configuration management | 12-factor (env vars), boot-time schema validation |
| 8.10    | Information deletion | Hard purge `flask delete-user`, FK cascade delete |
| 8.11    | Data masking | Log pseudonymisation, application encryption (Fernet) |
| 8.12    | Data leakage prevention | Recommended egress filtering (sovereign-hybrid), native air-gap (sovereign) |
| 8.13    | Backup | Daily `backup_databases` cron + documented restore procedure |
| 8.15    | Logging | `/admin/audit` + structured stdout JSON logs |
| 8.16    | Monitoring | `/healthz`, `/health`, `/metrics`, `/status` |
| 8.17    | Clock synchronisation | NTP required (licence signature validation) |
| 8.18    | Privileged software | `flask` CLI restricted to admin shell |
| 8.20    | Network security | TLS enforced, HSTS, strict CSP |
| 8.21    | Network services security | Reverse proxy in front (Pangolin/Nginx) |
| 8.22    | Network segregation | Sovereign: dedicated VLAN, no Internet |
| 8.23    | Web filtering | Out of scope (you) |
| 8.24    | Cryptography | TLS 1.2+, Argon2id, Fernet (AES-128-GCM), Ed25519 (licence) |
| 8.25    | Secure dev lifecycle | Bandit, isort, flake8, > 80 % test coverage |
| 8.26    | Application security | CSRF, rate limiting, security headers, input validation |
| 8.28    | Secure coding | Code reviews, automated tests, strict lint |
| 8.32    | Change management | OCI versioning, idempotent migrations, documented rollback |
| 8.33    | Test info | Isolated test bases, synthetic data |
| 8.34    | Audit | Immutable logs, off-host archive > 180 days |

## ISO/IEC 27701 — privacy extensions

| Control | Myeline coverage |
|---------|------------------|
| 6.5     | Processing-activities log (`/admin/audit`) |
| 6.6     | Incident management (documented 72 h procedure) |
| 7.2     | Legal bases (see [GDPR](gdpr.md)) |
| 7.3     | Data-subject information (T&Cs + privacy policy) |
| 7.4     | Choice and consent (cookies, OAuth, watch alerts) |
| 7.5     | Data-subject rights (export, rectification, erasure) |
| 8.5     | Sub-processors (registry provided — see [GDPR](gdpr.md)) |

## Audit-plan template

Available on request to [hello@myeline.io](mailto:hello@myeline.io)
as a pre-filled Excel spreadsheet, with:

- List of every Annex A control
- Status: covered / partially covered / out of Myeline scope
- Technical evidence (command / endpoint / config file)
- Free field for your audit comments

Useful for preparing an ISO 27001 audit or a vendor compliance
questionnaire.
