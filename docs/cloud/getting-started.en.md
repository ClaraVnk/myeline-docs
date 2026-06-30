# Getting started (Cloud)

This guide takes you from account creation to your first RAG answer on
the hosted offer [myeline.io](https://myeline.io). Allow **5 to 10
minutes**.

!!! info "Cloud only"
    This page is about the **Cloud (SaaS)** edition. For detailed
    feature usage (shared across all editions), see the
    [Concepts (all editions)](../concepts/index.md).

## 1. Create an account

Go to [myeline.io](https://myeline.io) and create a free account
(email + password). Enable **TOTP 2FA** from *Settings → Security* —
recommended from day one.

You start on **Free**: free, no time limit, with your own API key.

## 2. Plug in an API key (Free) or upgrade to Pro

=== "Free — your key (BYOK)"

    On Free, **your API key** powers both indexing **and** synthesis.
    Prefer **Mistral**: a single key covers both. Follow
    [Get a free AI API key in 2 minutes](../concepts/get-api-key.md),
    then paste the key into *Settings → AI provider*.

    !!! warning "No key, no access"
        A Free account must configure a valid key before reaching the
        dashboard. A **Claude** or **Gemini** key alone **cannot index**
        (no embedding API) — use Mistral or OpenAI as your single key.

=== "Pro — Mistral included"

    On **Pro** (€7.90/month, 15-day trial), **Mistral is included**: no
    key to manage, indexing and synthesis work from sign-up. Start the
    trial from the dashboard (*Upgrade to Pro*). You can still plug in
    your own key (BYOK) to override the synthesis provider.

## 3. Connect a source

In *Sources* (or *Connectors*), add at least one source to index:

- **Cloud connector**: Google Drive, OneDrive, Dropbox, kDrive…
  Authorise access via OAuth; Myeline syncs and indexes the documents.
  Details and scopes: [Cloud connectors](../concepts/cloud-connectors.md).
- **RSS / web source**: paste a feed or site URL; Myeline detects RSS
  vs HTML and scrapes the content.

The first sync may take a few minutes depending on volume. See
[Library](../concepts/library.md) for the indexing pipeline and
supported formats.

## 4. First RAG search

Once indexing is done, open **search** and ask a question in natural
language. Myeline retrieves the relevant passages, drafts an answer and
cites its sources with clickable `[N]` links.

- Pick the **scope** (personal / organisation / public).
- Switch to **strict mode** to allow only sourced answers.
- Continue in a **multi-turn conversation** (Pro) to refine.

Everything is detailed in [RAG search](../concepts/rag-search.md) and
[Multi-turn conversations](../concepts/conversations.md).

## 5. Upgrade to Pro (optional)

To lift all limits (unlimited documents and sources, augmented search,
multi-turn chat, watch alerts, hourly sync), upgrade to **Pro** from
your **dashboard** (*Upgrade to Pro* button, 15-day trial). Mistral is
then included.

To work as a team, create an **organisation**: billed **per seat
(€7.90/member)**, shared library.

## What next?

- [Alerts and watch](../concepts/watch-alerts.md) — track a keyword and
  receive a digest.
- [Cloud FAQ](faq.md) — pricing, trial, teams, data residency.
- [Choose your edition](../editions/index.md) — if your needs move
  toward on-premise.
