# Welcome to the Myeline documentation

**Myeline** is an intelligence platform for your data, deployable
directly on **your infrastructure**. It indexes RSS feeds, web
scrapers, personal documents and cloud storage then makes everything
queryable in natural language via RAG. Embedding is **always local**
(Ollama + bge-m3); synthesis uses a local LLM (Ollama) or an external
API (Mistral / Anthropic / OpenAI / Gemini) depending on your edition.

This documentation covers the **two on-prem editions** we deliver
turnkey to organisations: **Sovereign** (air-gap) and
**Sovereign-hybrid** (BYOK).

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
  organisations, OIDC SSO, quotas, audit log.
- :material-magnify: **[RAG search for end-users](usage/rag-search.md)**:
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
