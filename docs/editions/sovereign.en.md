# Sovereign edition (air-gap)

The **Sovereign** edition deploys Myeline **100% on your
infrastructure**, with no external API call. Designed for regulated
sectors where data residency and isolation are contractual or
regulatory requirements.

## Air-gap promise

No bit of user data leaves your network perimeter. Concretely, in
this edition:

| Component                | Air-gap behaviour                                   |
|--------------------------|------------------------------------------------------|
| **Embedding**            | Local Ollama (`bge-m3`) — on your infra              |
| **AI synthesis**         | Local Ollama (your choice: Mistral-Nemo, Llama 3.1, Mixtral, etc.) |
| **Mistral / Anthropic / OpenAI / Gemini** | **Blocked** — even if an API key is in `.env` |
| **Mailer (Brevo)**       | Forced log-only — emails written to `logs/mailer/`   |
| **Stripe**               | `/payment/*` returns 404, webhook 404                |
| **Cloud connectors**     | **S3 (internal MinIO) and WebDAV (internal Nextcloud) only**. Public-cloud (GDrive, OneDrive, Dropbox, Notion, Zotero, kDrive) blocked. |
| **Social login**         | `/auth/social/*` blocked. Enterprise OIDC SSO (Azure AD on-prem, Keycloak…) remains active. |
| **License validation**   | Offline Ed25519 signed — no network call required    |
| **Frontend CDN**         | Assets bundled locally (`/static/vendor/`) — no jsdelivr / Google Fonts |
| **Google Analytics**     | Hard-disabled                                         |

## For whom?

- Health sector under strict **HDS** (beyond just "host at an HDS")
- **Defence / DGA** (defence secret zones, French IGI 1300)
- **OIV / OIS** (NIS2, French LPM)
- **Banks** under ECB / ACPR regulation with isolation requirements
- Sensitive research (CEA, ONERA…)
- Law firms with absolute confidentiality contracts
- Public agencies with **SecNumCloud** mandates

## Prerequisites

See [Server prerequisites](../install/prerequisites.md). Summary:

- **CPU only**: 16 vCores, 32 GB RAM, 200 GB NVMe (≤ 20 internal users).
  LLM queries take 15-40 s in CPU-only synthesis.
- **With GPU**: RTX 4090 24 GB or L40S — recommended from 1 user
  for an interactive experience.
- **Bare-metal + 2× GPU** for Llama 3.1 70B or Mixtral 8×7B.

## Pricing model

- **Annual licence** (quoted, renewed yearly)
- No Stripe, no self-serve subscription
- The functional perimeter includes **all Pro+ features
  automatically** (multi-turn, conversations, alerts, query
  history…) — since the licence covers the operator, there's no
  notion of an internal "user plan"
- Support included — detailed terms in the licence contract

## Known limitations to be aware of

- **No external API** = no Mistral cloud / Claude / OpenAI / Gemini
  → you're capped by the local model quality you can host.
  Mistral-Nemo Q4 is decent for 90% of cases; for complex tasks
  you'll want Llama 3.1 70B (GPU mandatory).
- **No Brevo** = no transactional emails by default. To get real
  emails, configure an internal MTA / SMTP and point Myeline at it
  (custom work to discuss).
- **No public cloud connectors** = users can't index their personal
  Google Drive. Only an internal MinIO or Nextcloud makes sense in
  air-gap.
- **Manual updates**: you decide when to pull the new image. No
  automatic updates.
- **Licence revocation**: cannot be done instantly in air-gap. The
  practical defence is short expirations (≤ 12 months). See
  [Licence renewal](../operations/license-renewal.md).

## Getting started

```bash
# 1. Receive your licence key by email after the quote is signed.
# 2. On your server:

git clone -b synapse git@github.com:ClaraVnk/myeline.git
cd myeline
./scripts/install.sh
# → Choose option 2 (Sovereign installation)
# → Paste the received licence key
# → Answer the guided questions (Ollama URL, model, etc.)
```

For the full walkthrough, see
[Sovereign installation](../install/sovereign.md).
