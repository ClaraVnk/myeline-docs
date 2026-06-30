# Get a free AI API key in 2 minutes

On Myeline's **Free** edition, you bring your own AI API key (BYOK).
Good news: several providers offer a **free tier**, no credit card
needed. This guide shows the fastest route.

!!! tip "Are you on the Pro edition?"
    No key needed: **Mistral is included**. This guide only concerns
    the [Free](../editions/cloud.md) edition, where you bring your own
    key.

## Easiest: Google Gemini (free, no card)

1. Open [aistudio.google.com/apikey](https://aistudio.google.com/apikey).
2. Sign in with a Google account.
3. Click **Create API key**.
4. Copy the key shown (it starts with `AIza…`).

No credit card required; the free tier is plenty for individual use.

## Free alternative: Mistral

1. Open [console.mistral.ai/api-keys](https://console.mistral.ai/api-keys).
2. Create an account (email + password).
3. Click **Create new key**.
4. Copy the key.

Mistral offers a free tier and it's a **European** LLM — handy if data
residency matters to you.

## Paid options: OpenAI and Anthropic

Both providers require an **active billing account** (credit card) —
there is no genuinely free tier.

- **OpenAI**: [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
  → *Create new secret key*.
- **Anthropic Claude**: [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys)
  → *Create Key*.

!!! warning "Keep your key secret"
    An API key grants access to your account (and your billing). Never
    share it or publish it. If it leaks, revoke it from the provider's
    console and generate a new one.

## Final step

Paste your key into **Settings → AI provider** in Myeline. That's it —
RAG search now produces syntheses on your key.
