# Cloud FAQ

Frequently asked questions about the **hosted** offer (Free / Pro) on
[myeline.io](https://myeline.io). For on-premise questions, see the
[On-premise FAQ](../faq.md).

## Pricing

### How much does the hosted offer cost?

- **Free**: **free**, no time limit — you plug in your **own AI key**
  (BYOK: Mistral / OpenAI / Claude / Gemini).
- **Pro**: **€7.90/month**, **monthly billing**, with a **15-day free
  trial**. Mistral included (no key to manage) and all limits lifted.
- **Teams / organisations**: billed **per seat**, i.e.
  **€7.90/member**.

Full comparison: [Cloud edition (SaaS)](../editions/cloud.md).

### What does Pro add over Free?

Pro includes **Mistral** (indexing + synthesis, no key to manage), adds
**augmented search** (HyDE, multi-query, reranking, contextual
retrieval), and lifts all limits: **unlimited** documents and sources,
**multi-turn chat**, **watch alerts**, **all digest frequencies**,
**sync up to hourly**.

### Is there an annual plan?

No. **Pro is monthly only** (€7.90/month). For an on-premise need,
pricing is **quote-based**: [hello@myeline.io](mailto:hello@myeline.io).

## Trial

### Is the Pro trial really free?

Yes: **15 days**, **no charge before day 15**, **cancel any time**
(card required to start the trial). At the end of the trial you move to
monthly Pro at €7.90/month, or you go back to Free.

### What if I don't upgrade to Pro?

You stay on **Free**, for free and with no time limit, using your own
API key. Free remains fully usable day to day.

## Teams

### How does per-seat billing work?

You create an **organisation** and invite members. Each active member
is billed **€7.90/month**. All members get the full **Pro** feature set
and share an **organisation library**.

### Are libraries isolated between organisations?

Yes. Each organisation has its own vector collection, users and
sources. Organisations are strictly isolated from one another.

## Data and residency

### Where is my data hosted?

The Cloud platform is **hosted in Europe (France)**. Embedding and
synthesis go through the **Mistral AI API** (hosted in the EU / France,
**no training on API data**). On **Free**, synthesis and indexing use
**your** key: data then flows to **your** AI provider, under its terms.

### Is it GDPR-compliant?

Yes, by design: EU hosting, encryption, configurable retention,
exercise of rights (access, erasure). Details:
[GDPR compliance](../compliance/gdpr.md) and
[Data residency](../compliance/data-residency.md).

### What if I can't send my data to a SaaS?

If regulation requires air-gap or strict sovereignty (HDS,
SecNumCloud), the **On-premise / Sovereign** offer deploys Myeline on
**your own infrastructure**. It is a **bespoke, quote-based**
installation: [Choose your edition](../editions/index.md) or
[hello@myeline.io](mailto:hello@myeline.io).

## Getting started

### How do I start?

See [Getting started (Cloud)](getting-started.md): create an account,
plug in a key (or upgrade to Pro), connect a source, run a first RAG
search.
