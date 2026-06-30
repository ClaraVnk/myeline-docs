# Welcome to the Myeline documentation

**Myeline** is an intelligence platform for your data, deployable
directly on **your infrastructure**. It indexes RSS feeds, web
scrapers, personal documents and cloud storage then makes everything
queryable in natural language via RAG. Depending on the edition,
embedding and synthesis run **locally** (on-prem: Ollama + bge-m3) or
through an **external API**: on the hosted Cloud, embedding goes
through the **Mistral API** (EU / France) — included key on Pro, your
own key on Free — and synthesis uses included Mistral (Pro) or your
BYOK key (Free). In sovereign on-prem, everything stays local.

Myeline comes in two main edition families, plus a bespoke offering:

- **[Community / Self-host (OSS)](community/index.md)** — the
  **open-source (AGPL-3.0)**, single-user edition to self-host for free
  (BYOK + local Ollama).
- **[Cloud / Hosted](cloud/index.md)** — the SaaS version on
  [myeline.io](https://myeline.io): **Free** (your key) or **Pro** at
  €7.90/mo (Mistral included, 15-day trial), nothing to install.
- **[On-premise / Sovereign](editions/index.md)** — deployment on
  **your infrastructure** (air-gap or BYOK), multi-tenant: a
  **bespoke, quote-based** installation.

Feature usage (RAG search, library, connectors, conversations, watch)
is **common to all editions** and described in the
[Concepts (all editions)](concepts/index.md).

!!! tip "Using the hosted version (myeline.io)?"
    Start with the [Cloud / Hosted](cloud/index.md) page,
    [Getting started](cloud/getting-started.md) and the
    [Get a free AI API key in 2 minutes](concepts/get-api-key.md) guide.

---

## Which edition is right for you?

<div class="grid cards" markdown>

-   :material-shield-lock:{ .lg .middle } **Sovereign (air-gap)**

    ---

    100% on-prem, **no external API call ever**. Local synthesis
    via Ollama, mailer in log-only mode, connectors restricted to
    internal S3 / WebDAV. Built for regulated sectors (health,
    defence, finance, public sector).

    [:octicons-arrow-right-24: Learn more](editions/sovereign.md)

-   :material-server:{ .lg .middle } **Sovereign-hybrid (BYOK)**

    ---

    On-prem + external APIs with your own keys. You run on your
    infra with your own Mistral / Anthropic / OpenAI / Gemini keys
    and enable cloud connectors with your own OAuth apps.

    [:octicons-arrow-right-24: Learn more](editions/sovereign-hybrid.md)

-   :material-cloud:{ .lg .middle } **Cloud (SaaS)**

    ---

    Hosted on myeline.io (EU / France), nothing to install. **Free**
    is free with your own API key, **Pro** at €7.90/mo with Mistral
    included (15-day trial). Need on-premise? On quote.

    [:octicons-arrow-right-24: Learn more](editions/cloud.md)

-   :fontawesome-brands-github:{ .lg .middle } **Community / Self-Host**

    ---

    **Open-source (AGPL-3.0)**, **single-user** edition to self-host
    for free. Multi-LLM BYOK + **fully local LLM via Ollama**, no
    licence and no organisation. Code on GitHub.

    [:octicons-arrow-right-24: Learn more](editions/community.md)

</div>

---

## Getting started

1. **[Choose your edition](editions/index.md)** — which mode fits
   your need (regulation, budget, autonomy).
2. **[Server prerequisites](install/prerequisites.md)** — CPU / RAM /
   disk sizing depending on mode and number of users.
3. **[Installation](install/index.md)** — step-by-step walkthrough
   guided by our `install.sh` script.
4. **[First admin login](install/first-login.md)** — set up the
   admin account, the organisation, verify the overall state.

---

## Quick references

- :material-cog: **[Day-2 operations](operations/index.md)**: backup,
  upgrade, monitoring, license renewal.
- :material-account-cog: **[Administration](admin/index.md)**: manage
  organisations, quotas, audit log.
- :material-magnify: **[RAG search for end-users](concepts/rag-search.md)**:
  asking questions, refining results, conversations.
- :material-shield-check: **[GDPR compliance](compliance/gdpr.md)**:
  sub-processor registry, DPA, exercising rights.
- :material-help-circle: **[FAQ](faq.md)**: the 20 most frequent
  questions.

---

## Support

Both on-prem editions include email support. Detailed conditions
(SLA, scope, escalation) are specified in the licence contract
signed with your organisation.

Contact: [hello@myeline.io](mailto:hello@myeline.io)
