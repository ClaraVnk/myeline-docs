# Choose your edition

Two on-prem Myeline editions, each calibrated for a specific need.
Pick yours based on **three criteria**: the regulation that applies
to your data, your need for autonomy, and the infrastructure you
have available.

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

**Implications**: to enable cloud connectors (GDrive, OneDrive,
Dropbox), you must register **your own OAuth apps** with each
provider — see the [BYOC](sovereign-hybrid.md#byoc) walkthrough.

[Learn more →](sovereign-hybrid.md){ .md-button .md-button--primary }

## Migrating between editions

Switching between Sovereign and Sovereign-hybrid is possible
**without data loss**:

- **Sovereign → Sovereign-hybrid**: change `DEPLOYMENT_MODE` in
  `.env`, provide a new licence key for the hybrid tier, restart.
  No data migration.
- **Sovereign-hybrid → Sovereign**: symmetric.

Edition changes happen at licence renewal time (max 12 months).
Contact us to plan.
