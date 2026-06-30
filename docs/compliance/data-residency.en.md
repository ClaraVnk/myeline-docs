# Data residency

A central topic for organisations under sovereignty constraints
(HDS, OIV, public sector, defence).

## Overview per edition

| Component                 | Sovereign (air-gap)            | Sovereign-hybrid (BYOK)                     |
|---------------------------|--------------------------------|---------------------------------------------|
| Web application           | Your infra                     | Your infra                                  |
| Database                  | Your infra                     | Your infra                                  |
| ChromaDB (vectors)        | Your infra                     | Your infra                                  |
| Uploads / files           | Your infra                     | Your infra                                  |
| Embedding (bge-m3)        | Your infra (local Ollama)      | Your infra                                  |
| LLM synthesis             | Your infra (local Ollama)      | Your choice: local Ollama OR external BYOK API |
| Mailer                    | Log-only (your infra)          | Brevo (EU) or internal MTA                  |
| Audit log archive         | Internal MinIO (your infra)    | Internal MinIO or cloud S3 (your choice)    |
| Backups                   | Your infra + off-host of your choice | Same                                  |
| OAuth credentials         | n/a (public connectors disabled) | Your providers (US/EU per provider)       |

## Sovereign (air-gap) — technical lock-in

In sovereign mode:

- **No external API call is technically possible** by construction.
  Routes `/payment/*`, `/auth/social/*` return 404. Public cloud
  connectors (GDrive, OneDrive…) are hidden and their routes
  disabled.
- **Brevo mailer is forced to log-only** — even if an API key is
  present in `.env`.
- **Frontend assets are bundled locally** (no jsdelivr CDN, no
  Google Fonts) — see `app/static/vendor/`.
- **Offline licence validation** via Ed25519 signature (no licence
  server call).
- **No telemetry** (Google Analytics and the like hard-disabled).

Verification: `curl -i http://localhost:5000/auth/social/google` must
return 404; `python -m app.utils.deployment_mode` must show
`mode=sovereign`.

## Sovereign-hybrid — identified outbound calls

In sovereign-hybrid, **every outbound call is explicit and
documented**. Exhaustive list per configuration:

| Destination                   | When                                 | Typical volume         |
|-------------------------------|--------------------------------------|------------------------|
| Mistral / Anthropic / OpenAI / Gemini API | On every RAG query if BYOK active | 1-5 KB per question   |
| Brevo SMTP                    | Transactional email (signup, reset, invitation) | 1 KB per email |
| Google / Microsoft / Dropbox APIs | Cloud connector sync             | Variable (listing + download) |
| OIDC IdP                      | SSO login (SSO on request, no longer self-service) | 2-3 KB per auth        |

**No outbound call** ever occurs:

- To a Myeline licence server (offline validation)
- To a frontend CDN
- To a telemetry service

## Infrastructure recommendations

### Pure sovereign

- Dedicated VLAN, no Internet route (or default DROP on egress
  firewall).
- Internal DNS only.
- NTP synced to an internal server (for licence signature
  validation — clock must remain within ±1 day of reality, otherwise
  the licence will be rejected as expired or not-yet-valid).

### Sovereign-hybrid

- Egress filtering: explicit whitelist of allowed domains
  (api.mistral.ai, api.anthropic.com, api.openai.com,
  generativelanguage.googleapis.com, api.brevo.com, accounts.google.com,
  login.microsoftonline.com, api.dropbox.com…). Everything else DROP.
- Outbound connection logging (for audit).
- Reverse proxy (Pangolin / Traefik / Nginx) in front, TLS ≥ 1.2,
  HSTS, strict CSP.

## Personal data vs technical metadata

Myeline distinguishes three criticality levels:

1. **User data** (documents, queries, conversations) — always stays
   on your infra.
2. **Technical identifiers** (user_id, org_id, action_code in the
   audit log) — stay on your infra.
3. **Anonymous metrics** (uptime, latency, error rate) — exposed
   via `/metrics`, scrapable only by your Prometheus (no external
   push).

No **email**, no **query content**, no **indexed document** ever
leaves your infra — except if you explicitly enable an external
AI provider in sovereign-hybrid (and then only the question text +
relevant chunks transit to the provider).
