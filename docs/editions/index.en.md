# Choose your edition

Myeline comes in two families: the hosted **Cloud** offering
([myeline.io](https://myeline.io), nothing to install) and the
**on-prem editions** deployed on your own infrastructure. The rest of
this page compares the two on-prem editions.

!!! info "Looking for the hosted version?"
    The [Cloud (SaaS) edition](cloud.md) offers **Free / Pro /
    Enterprise** plans on myeline.io, no installation. Free is free
    (with your own API key), Pro includes Mistral at €19.90/mo. See
    [Cloud edition](cloud.md).

!!! tip "Just want to self-host it for free?"
    The [**Community / Self-Host** edition](community.md) is the
    **open-source (AGPL-3.0)**, **single-user** version to install on
    your own server — multi-LLM BYOK **+ fully local LLM via Ollama**,
    no licence and no organisation. The code is on
    [GitHub](https://github.com/ClaraVnk/myeline).

The two **on-prem editions** below are each calibrated for a specific
need. Pick yours based on **three criteria**: the regulation that
applies to your data, your need for autonomy, and the infrastructure
you have available.

## Comparison table

| Criterion                      | Sovereign           | Sovereign-hybrid           |
|--------------------------------|---------------------|----------------------------|
| **Hosting**                    | Your infra          | Your infra                 |
| **Internet connection**        | None (air-gap)      | Outbound allowed (BYOK)    |
| **AI synthesis**               | Local Ollama only   | Per-org choice: Ollama OR Mistral / Claude / OpenAI / Gemini |
| **Embedding**                  | Local Ollama        | Local Ollama               |
| **Cloud connectors**           | S3 + WebDAV only (toward internal infra) | All (with **your** OAuth apps — see BYOC) |
| **Social login (Google/MS/Apple)** | ❌              | ❌ (use enterprise OIDC instead) |
| **Enterprise OIDC SSO**        | Included            | Included                   |
| **Stripe billing**             | ❌                  | ❌                         |
| **Multi-tenant**               | Single-tenant by default | Single-tenant by default |
| **Updates**                    | Manual (`podman pull`) | Manual (`podman pull`)  |
| **Pricing**                    | Annual licence (quote) | Annual licence (quote) |

## When to choose which?

### Sovereign (air-gap)

**For**: public agencies, vital-importance operators, defence sector,
HDS-strict health, regulated banks, any organisation subject to a
strict data-localisation audit.

**Pros**: not a single bit leaves your network. All components (AI
synthesis, mailer, monitoring) run locally. SecNumCloud-compliant
by construction.

**Trade-offs**: no external API means you must use local LLMs
(Mistral-Nemo, Llama 3.1, Mixtral…), which need GPU-equipped
servers for acceptable performance. See
[server prerequisites](../install/prerequisites.md).

[Learn more →](sovereign.md){ .md-button .md-button--primary }

### Sovereign-hybrid (BYOK)

**For**: companies that want sovereign-like infrastructure isolation
but with frontier LLM quality (Mistral Large, Claude Sonnet 4.6,
GPT-5, Gemini 2.5).

**Pros**: infrastructure and data stay with you, but synthesis can
be routed per-organisation toward the LLM of your choice using
**your own API key** (BYOK). You pay the AI provider directly, no
intermediary.

**Implications**: to enable cloud connectors, you set up credentials
with each provider from your tenant:
- **Google Drive**: create a GCP service account in your project
- **OneDrive / Dropbox / Notion**: register your own OAuth apps (BYOC)

Full walkthrough:
[Sovereign-hybrid installation § Cloud connectors](../install/sovereign-hybrid.md#cloud-storage-connectors).

[Learn more →](sovereign-hybrid.md){ .md-button .md-button--primary }

## What about the Cloud edition?

If you have no self-hosting constraint, the [Cloud (SaaS)
edition](cloud.md) is the fastest path: free account, cloud
connectors, RAG search, no server to manage. The Cloud Enterprise
plan overlaps on-prem needs (isolation, OIDC SSO, audit) — we point
you to the right edition depending on your sovereignty requirements.

## Migrating between editions

Switching between Sovereign and Sovereign-hybrid is possible
**without data loss**:

- **Sovereign → Sovereign-hybrid**: change `DEPLOYMENT_MODE` in
  `.env`, provide a new licence key for the hybrid tier, restart.
  No data migration.
- **Sovereign-hybrid → Sovereign**: symmetric.

Edition changes happen at licence renewal time (max 12 months).
Contact us to plan.
