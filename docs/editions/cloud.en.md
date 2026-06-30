# Cloud edition (SaaS)

The **Cloud** edition is the hosted version of Myeline, available
directly at [myeline.io](https://myeline.io) — **nothing to install**.
You create an account, connect your drives and sources, and query
everything in natural language via RAG.

The platform is **hosted in Europe (France)**, GDPR-compliant.
Embedding goes through the **Mistral AI embedding API** (hosted in
EU / France, no training on API data): with the **included platform
key** on Pro, with **your own key** on Free. The included EU LLM for
synthesis is **Mistral**. *(Local Ollama + bge-m3 embedding only
applies to the sovereign on-premise offering.)*

!!! info "Hosted or on-premise?"
    This page covers the **hosted** offering (Free / Pro). If you need
    everything to run on **your own infrastructure** (air-gap,
    SecNumCloud, HDS), the **On-premise / Sovereign** offering is a
    **bespoke, quote-based** installation — see
    [On-premise / Sovereign](sovereign.md) or email us at
    [hello@myeline.io](mailto:hello@myeline.io).

## Free or Pro?

Two hosted plans. **Free** is genuinely usable day-to-day with **your
own free API key**; **Pro** mainly brings the convenience of **Mistral
included — no key to manage**, lifts every limit and adds team work.

| Criterion                        | Free                                    | Pro — €7.90/mo                           |
|----------------------------------|-----------------------------------------|------------------------------------------|
| **Hosting**                      | Myeline Cloud (EU / France)             | Myeline Cloud (EU / France)              |
| **AI synthesis**                 | **BYOK required** (your Mistral / OpenAI / Claude / Gemini key) | **Mistral included** (BYOK to override) |
| **Embedding (indexing)**         | **Your own key** (Mistral or OpenAI — Claude/Gemini can't index) | **Mistral included** |
| **Augmented retrieval** (HyDE, multi-query, reranking) | ❌                | ✅                                       |
| **Cloud connectors**             | All (Google Drive, OneDrive, Dropbox, kDrive…) | All                              |
| **Personal library**             | Up to **500 documents**                 | **Unlimited**                            |
| **Custom RSS / web sources**     | **50 sources**, **100 articles/source** | **Unlimited**                            |
| **Digest**                       | Monthly only                            | **All frequencies**                      |
| **Multi-turn chat**              | ❌ (plain RAG search)                   | ✅                                       |
| **Watch alerts**                 | ❌                                      | ✅                                       |
| **Cloud sync**                   | Every **24 h** (fixed)                  | Configurable, **down to every hour**     |
| **Team / organisation**          | ❌                                      | **Per seat — €7.90/member**              |
| **Pricing**                      | €0 (you pay your AI provider)           | **€7.90/mo** (15-day trial)              |

!!! tip "🎁 Free Pro trial — 15 days"
    The **Pro** plan includes a **15-day free trial** (card required, no
    charge until day 15, cancel anytime). Billing is **monthly**. The
    **Free** edition stays free forever, with your own API key.

## Free — free, with your own key (BYOK)

The Free edition is **free with no time limit**. The only requirement:
you bring **your own AI API key** (BYOK), which powers **both indexing
(embeddings) AND answer synthesis**. Free runs **entirely on your key**
— Myeline spends **€0 in API** on your behalf.

Since everything runs on your key, the provider choice matters:

- a single **Mistral** or **OpenAI** key covers **both** (indexing +
  answers) — these providers expose an embedding API;
- a **Claude** or **Gemini** key can **answer** but **cannot index**
  (no embedding API) — avoid it as your only key on Free.

**No key = no access**: a Free user must configure a valid key before
reaching the dashboard.

!!! info "Augmented retrieval: a Pro perk"
    The **augmented retrieval** passes (HyDE, multi-query, reranking,
    contextual retrieval) call the platform LLM and are therefore
    **off on Free** — they're **included on Pro** as a value-add.

Several providers offer a **free tier** (Mistral, Gemini), which makes
Free genuinely usable at no cost — prefer Mistral to cover indexing
**and** answers with a single key. See the guide
[Get a free AI API key in 2 minutes](../usage/get-api-key.md).

Included in Free:

- **All cloud connectors** (Google Drive, OneDrive, Dropbox, kDrive…)
- **Personal library up to 500 documents**
- **Custom RSS / web sources**: up to **50 sources**, capped at
  **100 articles per source**
- **Monthly digest**
- **Cloud sync every 24 h** (fixed cadence)
- **RAG search**: both indexing **and** synthesis run on your key

## Pro — €7.90/mo, Mistral included

The Pro edition (**€7.90/mo**, billed **monthly**, **15-day free
trial**) includes **Mistral AI** (France, EU-hosted, no training on API
data) for **both indexing (embeddings) AND synthesis**: no key to
manage, everything works from sign-up. You can still **bring your own
key** (BYOK) to override the synthesis provider if you prefer Claude,
GPT or Gemini.

Pro takes everything in Free **with no limits**, and adds **augmented
retrieval** (HyDE, multi-query, reranking, contextual retrieval),
which is off on Free:

- **Unlimited documents, sources and articles**
- **All digest frequencies** (daily, weekly, monthly…)
- **Multi-turn chat** and **watch alerts**
- **Cloud sync configurable, down to every hour**
- **Organisation / team** billed **per seat**: **€7.90/member**. Every
  member gets the full Pro feature set.

## On-premise / Sovereign — on quote

Can't send your data to a SaaS — strict HDS regulation, air-gap,
SecNumCloud, sovereignty requirement? The **On-premise / Sovereign**
offering deploys Myeline **on your own infrastructure**.

It's a **bespoke, quote-based** installation: scope, AI models
(Mistral, local Ollama, BYOK), connectors and integration are tailored
with you. There is **no self-service subscription** — let's talk.

[:octicons-arrow-right-24: Learn more about On-premise / Sovereign](sovereign.md){ .md-button }
[:octicons-mail-24: Contact us](mailto:hello@myeline.io){ .md-button .md-button--primary }

## How to sign up

1. **Create a free account** at [myeline.io](https://myeline.io).
2. **Get an API key** from an AI provider (free with Gemini or Mistral)
   — see [Get a free AI API key in 2 minutes](../usage/get-api-key.md).
3. **Paste your key** in *Settings → AI provider*.
4. **Connect your drives** and add your RSS / web sources.
5. To move to **Pro**, do it from your **dashboard** (*Upgrade to Pro*
   button, 15-day trial) — Mistral is then included and all limits are
   lifted.
