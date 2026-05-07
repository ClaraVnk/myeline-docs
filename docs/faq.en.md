# FAQ

The most frequent questions about Myeline on-prem editions.

## General

### What's the difference between sovereign and sovereign-hybrid?

- **Sovereign**: 100 % air-gap, no outbound calls technically
  possible. AI synthesis runs locally via Ollama.
- **Sovereign-hybrid**: air-gap by default, but each organisation
  can enable an external AI provider (Mistral / Claude / OpenAI /
  Gemini) with **its own key** (BYOK).

Details: [Choose your edition](editions/index.md).

### Can I migrate between the two?

Yes, with no data loss. Change `DEPLOYMENT_MODE` in `.env`, provide
the new licence key for the desired tier, restart.

### How much does the licence cost?

On quote, depending on number of users and scope. Contact
[hello@myeline.io](mailto:hello@myeline.io). Licences are **annual**,
12 months max (no perpetual licence is issuable, by construction).

### Is there a "trial" mode?

Yes — 30-day evaluation licence, request via the same channel.
Covers all hybrid-tier features.

## Data

### Does my data leave my network?

- **Sovereign**: no, never. No outbound call is technically possible.
- **Sovereign-hybrid**: only if you explicitly enable an external AI
  provider. In that case, the question text + relevant chunks
  transit to the provider on every query. Documents and embeddings
  always stay on your infra.

Details: [Data residency](compliance/data-residency.md).

### What database size can I handle?

Order of magnitude on standard sizings:

| Volume                | Recommended setup            |
|-----------------------|------------------------------|
| ≤ 100k documents      | 32 GB RAM, NVMe, optional GPU |
| 100k - 1M documents   | 64 GB RAM, GPU 24 GB, striped NVMe |
| > 1M documents        | ChromaDB sharding + dedicated Ollama cluster |

See [Server prerequisites](install/prerequisites.md).

### Is it GDPR / HDS compliant?

- **GDPR**: yes, by construction (encryption, pseudonymisation,
  audit, configurable retention, right to erasure).
- **HDS**: Myeline itself is not HDS-certified — certification is
  on the host. In sovereign mode, you are the host: if your infra
  is HDS-certified, your Myeline deployment is.

Details: [GDPR compliance](compliance/gdpr.md).

## Features

### Can I use GPT-4 or Claude?

- **Sovereign**: no.
- **Sovereign-hybrid**: yes, via BYOK. Each organisation picks its
  provider and supplies its own API key. No AI cost via Myeline —
  you pay the provider directly.

### Which document formats are supported?

PDF, DOCX, ODT, TXT, MD, HTML, CSV. See
[Library and uploads](usage/upload-documents.md).

OCR on scanned PDFs is disabled by default — enable via
`RAG_OCR_ENABLED=true` (sovereign-hybrid only, Tesseract dependency).

### Is there a manual file upload?

**No, not at all** — neither personal nor organisation side. The
library (personal or shared) is filled exclusively via:

- **Cloud connectors**: Google Drive, OneDrive, Dropbox, kDrive,
  Zotero, S3 / S3-compatible, WebDAV
- **RSS / web scrapers**

This is a design choice (single source of truth on the customer
side, compliance, simple ingestion path) — see
[Library § Why no manual upload?](usage/upload-documents.md#why-no-manual-upload).

### How many concurrent users?

Without hardware bottleneck: **several hundred** on a properly-sized
infra. The typical bottleneck is Ollama (AI synthesis) on a single
GPU — beyond ~20-30 active concurrent users, plan a GPU cluster or
switch to BYOK in sovereign-hybrid.

### Does multi-tenant work on-prem?

Yes. One Myeline instance can host **multiple organisations**, each
with its own ChromaDB collection, users, OIDC, AI provider (in
sovereign-hybrid). Organisations are strictly isolated.

## Operations

### How often should I update?

- **Security patches**: within 7 days of release.
- **Minor releases**: per your maintenance window.
- **Major releases** (rare): 30 days notice, opportunity to track
  release candidates in a pre-prod environment.

See [Upgrade](operations/upgrade.md).

### How do I back up?

Automatic daily cron at 02:30 (DB + ChromaDB + uploads). Off-host
recommended via rclone / MinIO / borg. See
[Backup and restore](operations/backup-restore.md).

### Can I integrate Myeline with my enterprise SSO?

Yes — OIDC (Azure AD, Okta, Keycloak, Authentik, any standard IdP).
See [Enterprise SSO](admin/oidc-sso.md).

### Is there a public API?

No formally-exposed REST API to end-users in current editions.
Admins have access to internal endpoints (`/admin/*`, `/metrics`,
`/healthz`), and Flask blueprints are introspectable if you want to
script internally.

A public API (REST + OpenAPI) is on the roadmap for Q4 2026.

## Support

### What's the SLA?

The SLA and detailed support conditions are specified in the
licence contract signed with your organisation.

### Where do I ask a question?

- Email: [hello@myeline.io](mailto:hello@myeline.io)
- For reproducible bugs: attach a `/health` snapshot + last 50
  `web` log lines.

### Can I access the source code?

Yes in sovereign-hybrid under escrow and on specific request
(commercial clauses). Myeline source is not public open-source but
the publisher can provide a read-only repository for client-side
security audit.
