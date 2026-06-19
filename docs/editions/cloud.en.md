# Cloud edition (SaaS)

The **Cloud** edition is the hosted version of Myeline, available
directly at [myeline.io](https://myeline.io) — **nothing to install**.
You create an account, connect your drives and sources, and query
everything in natural language via RAG.

The platform is **hosted in Europe (France)**, GDPR-compliant.
Embedding runs **locally on the platform** (never delegated to a third
party) and the included EU LLM is **Mistral**.

!!! info "Cloud vs on-prem"
    This page covers the **hosted** offering (Free / Pro / Enterprise).
    If you need everything to run on **your own infrastructure**
    (air-gap or BYOK on-prem), see [the on-prem editions](index.md).

## Free, Pro or Enterprise?

Three plans. **Free** is genuinely usable day-to-day with **your own
free API key**; **Pro** mainly brings the convenience of **Mistral
included — no key to manage** and lifts every limit; **Enterprise** is
the isolated version for organisations.

| Criterion                        | Free                                    | Pro — €19.90/mo                          | Enterprise                         |
|----------------------------------|-----------------------------------------|------------------------------------------|------------------------------------|
| **Hosting**                      | Myeline Cloud (EU / France)             | Myeline Cloud (EU / France)              | Dedicated isolated stack           |
| **AI synthesis**                 | **BYOK required** (your Anthropic / OpenAI / Gemini / Mistral key) | **Mistral included** (BYOK to override) | Mistral included + BYOK            |
| **Embedding**                    | Local on platform                       | Local on platform                        | Local on platform                  |
| **Cloud connectors**             | All (Google Drive, OneDrive, Dropbox, kDrive…) | All                              | All                                |
| **Personal library**             | Up to **500 documents**                 | **Unlimited**                            | Unlimited                          |
| **Custom RSS / web sources**     | **50 sources**, **100 articles/source** | **Unlimited**                            | Unlimited                          |
| **Digest**                       | Monthly only                            | **All frequencies**                      | All frequencies                    |
| **Multi-turn chat**              | ❌ (plain RAG search)                   | ✅                                       | ✅                                 |
| **Watch alerts**                 | ❌                                      | ✅                                       | ✅                                 |
| **Cloud sync**                   | Every **24 h** (fixed)                  | Configurable, **down to every hour**     | Custom (down to 15 min)            |
| **Team / organisation**          | ❌                                      | **Per seat — €19.90/seat, up to 10 members** | Unlimited members             |
| **OIDC SSO**                     | ❌                                      | ❌                                       | ✅                                 |
| **Audit log**                    | ❌                                      | ❌                                       | ✅                                 |
| **SLA**                          | Best-effort                             | Best-effort                              | Custom SLA                         |
| **Pricing**                      | €0 (you pay your AI provider)           | €19.90/mo                  | On quote                           |

!!! tip "🎁 Free Pro trial — 15 days"
    The **Pro** plan includes a **15-day free trial** (card required, no
    charge until day 15, cancel anytime). And the **Free** edition stays
    free forever, with your own API key.

## Free — free, with your own key (BYOK)

The Free edition is **free with no time limit**. The only requirement:
you bring **your own AI API key** (BYOK) among Anthropic Claude,
OpenAI, Google Gemini or Mistral. Without a key, RAG search can't
produce an AI synthesis — indexing and raw search still work, but the
synthesised answer needs a valid key.

Several providers offer a **free tier** (Gemini, Mistral), which makes
Free genuinely usable at no cost. See the guide
[Get a free AI API key in 2 minutes](../usage/get-api-key.md).

Included in Free:

- **All cloud connectors** (Google Drive, OneDrive, Dropbox, kDrive…)
- **Personal library up to 500 documents**
- **Custom RSS / web sources**: up to **50 sources**, capped at
  **100 articles per source**
- **Monthly digest**
- **Cloud sync every 24 h** (fixed cadence)
- **RAG search** running on your key

## Pro — €19.90/mo, Mistral included

The Pro edition (**€19.90/mo**, or **−20% on yearly billing**)
includes **Mistral AI**: no key to manage, AI synthesis works from
sign-up. You can still **bring your own key** (BYOK) to override the
provider if you prefer Claude, GPT or Gemini.

Pro takes everything in Free **with no limits**:

- **Unlimited documents, sources and articles**
- **All digest frequencies** (daily, weekly, monthly…)
- **Multi-turn chat** and **watch alerts**
- **Cloud sync configurable, down to every hour**
- **Organisation / team** billed **per seat**: **€19.90/seat, up to 10
  members**. Beyond 10 members, move to Enterprise.

## Enterprise — on quote

The Enterprise edition is for organisations that want a **dedicated,
isolated stack** on Myeline Cloud, with:

- **OIDC SSO** (Azure AD, Okta, Keycloak…)
- **Unlimited members**
- **Custom SLA**
- Full **audit log**

!!! tip "Enterprise and on-prem overlap"
    Enterprise needs (isolation, SSO, audit, unlimited members) overlap
    with the [on-prem Sovereign / Sovereign-hybrid editions](index.md).
    If sovereignty or air-gap are requirements, let's talk:
    [hello@myeline.io](mailto:hello@myeline.io).

## How to sign up

1. **Create a free account** at [myeline.io](https://myeline.io).
2. **Get an API key** from an AI provider (free with Gemini or Mistral)
   — see [Get a free AI API key in 2 minutes](../usage/get-api-key.md).
3. **Paste your key** in *Settings → AI provider*.
4. **Connect your drives** and add your RSS / web sources.
5. To move to **Pro**, do it from your **dashboard** (*Upgrade to Pro*
   button) — Mistral is then included and all limits are lifted.

## Need self-hosting?

If you can't send your data to a SaaS — strict HDS regulation,
air-gap, SecNumCloud — the [on-prem editions](index.md) deploy Myeline
**on your own infrastructure**, in Sovereign (air-gap) or
Sovereign-hybrid (BYOK) mode.
