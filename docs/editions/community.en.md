# Community / Self-Host edition (open source)

The **Community** edition is the **open-source** version of Myeline,
released under the **AGPL-3.0** licence on GitHub. It runs **entirely on
your own infrastructure**, is **single-user**, and depends on **no
Myeline service**: no licence to buy, no account to create, no calls
back to our servers.

This is the open-core heart of Myeline. If you just want to host your
own RAG knowledge base on your server — no organisation, no billing, no
external dependency — this is the edition for you.

[:fontawesome-brands-github: Code on GitHub](https://github.com/ClaraVnk/myeline){ .md-button .md-button--primary }
[:material-book-open-variant: Self-host guide](https://github.com/ClaraVnk/myeline/blob/main/docs/SELF_HOSTING.md){ .md-button }

!!! info "Community vs hosted / on-prem editions"
    This page covers the **open-source, single-user** edition.
    Multi-tenant features (organisations and teams, billing) are
    **not** part of this project — they
    belong to the [Cloud (SaaS) edition](cloud.md) and the
    [on-prem Sovereign / Sovereign-hybrid editions](sovereign-hybrid.md).

## Who is it for?

- **Developers and tinkerers** who want a self-hosted personal RAG,
  inspectable and modifiable (the code is open, AGPL).
- **Self-hosters** already running their own stack (Nextcloud,
  Paperless, etc.) who want to add an intelligence layer over their
  documents.
- **Personal data sovereignty**: everything stays on your server —
  documents, vector index, conversation history. In local-LLM mode
  (Ollama), **no data ever leaves the machine**.

## Features

- **Cloud storage connectors** — automatic sync and indexing from Google
  Drive, OneDrive, Dropbox, kDrive (Infomaniak), S3 / S3-compatible
  (Scaleway, OVH, R2, MinIO…), WebDAV, Notion and Zotero. Each connector
  uses **your own credentials** (see [BYOC](#byoc-your-own-oauth-apps)).
- **RSS / web sources** — point Myeline at a feed or a site; auto-detects
  RSS vs HTML, scrapes content (`trafilatura` + sitemap walking),
  extracts publication dates.
- **Personal library** — upload documents (PDF / DOCX / ODF), parsed and
  indexed alongside everything else.
- **RAG search** — top-K retrieval with MMR diversification, clickable
  `[N]` citations, keyword highlighting, strict/relaxed toggle, temporal
  filtering, source-freshness indicator, multi-turn conversations and
  single-document chat.
- **Query tooling** — automatic history, saved queries, 👍/👎 feedback on
  answers.
- **Digests & watch alerts** — keyword + scope watches running on a
  schedule, emailing a digest of new matching sources.
- **Two-factor authentication** — TOTP-based 2FA on the account.

## Multi-LLM (BYOK) and fully local LLM

The Community edition is **bring-your-own-key**. You pick the synthesis
provider and paste **your own API key** in *Settings → AI provider*; the
key is stored **encrypted at rest**.

| Provider           | Synthesis | Embedding (indexing) |
|--------------------|-----------|----------------------|
| Mistral            | ✅        | ✅                   |
| OpenAI             | ✅        | ✅                   |
| Anthropic Claude   | ✅        | —                    |
| Google Gemini      | ✅        | —                    |
| Voyage / Cohere    | —         | ✅                   |

Indexing needs an embedding-capable key (Mistral or OpenAI cover both
synthesis and embedding; Claude/Gemini answer but cannot embed).

!!! tip "Self-hosted LLM via Ollama — no third party at all"
    For a **fully self-contained** install, run the model yourself. The
    **[Ollama](https://ollama.com)** service ships in the
    `docker-compose` file: point `OLLAMA_URL` at it and Myeline uses it
    for both embedding (`bge-m3`) and synthesis. **No API key, no data
    leaves the host.** This is the recommended setup when data residency
    or air-gapping matters.

    Any **OpenAI-compatible** endpoint (vLLM, LM Studio, llama.cpp
    server) works the same way by pointing the OpenAI provider at your
    local base URL.

## BYOC — your own OAuth apps { #byoc-your-own-oauth-apps }

Each cloud connector authenticates with **your** credentials, not
Myeline's. For the OAuth connectors (Google Drive, OneDrive, Dropbox,
kDrive) you register **one app per provider**, once; the redirect URI is
`https://<your-domain>/user/cloud/<provider>/callback`. The token-based
connectors (Notion, S3, WebDAV, Zotero) just take a token or access key
entered in the UI.

Full details, scopes and console links in the
[self-host guide (§ Bring-your-own credentials)](https://github.com/ClaraVnk/myeline/blob/main/docs/SELF_HOSTING.md#5-bring-your-own-credentials-connectors).

## How it differs from the other editions

| Criterion                       | Community (Self-Host)        | Cloud (SaaS)                 | Sovereign / Sovereign-hybrid |
|---------------------------------|------------------------------|------------------------------|------------------------------|
| **Licence**                     | Open source **AGPL-3.0**     | Proprietary (SaaS)           | Proprietary (quote)          |
| **Cost**                        | Free                         | Free / Pro €7.90/mo          | Quote                        |
| **Hosting**                     | Your infra                   | Myeline Cloud (EU)           | Your infra                   |
| **Users**                       | **Single-user**              | Multi (per seat / org)       | Multi-tenant                 |
| **AI synthesis**                | BYOK **+ local Ollama**      | Mistral included / BYOK      | Local Ollama and/or BYOK     |
| **Cloud connectors**            | All (your OAuth apps)        | All                          | All (BYOC) / S3+WebDAV (air-gap) |
| **Organisations & teams**       | ❌                           | ✅ (Pro)                     | ✅                           |
| **Billing / Stripe**            | ❌                           | ✅                           | ❌                           |
| **Support**                     | Community (best-effort)      | Per plan                     | Included in contract         |

In short: the Community edition is the **single-user open-source
foundation**. Organisations, billing and multi-tenant are reserved
for the [Cloud](cloud.md) and
[Sovereign / Sovereign-hybrid](sovereign-hybrid.md) editions.

## Get started

Requirements: Podman 4.6+ and `podman-compose` (or Docker + Compose).

```bash
git clone https://github.com/ClaraVnk/myeline.git ~/Projects/myeline
cd ~/Projects/myeline
cp .env.example .env        # then edit — see docs/SELF_HOSTING.md

# Build and start the stack (web, worker, cron, MariaDB, Redis, Ollama)
podman-compose -f docker-compose.yml up -d --build

# Create your account (first boot only)
podman-compose -f docker-compose.yml exec web flask create-admin
```

Then open <http://localhost:5000>.

The full walkthrough (prerequisites, key generation, LLM choice,
per-connector OAuth registration) lives in the repo's
**[self-host guide](https://github.com/ClaraVnk/myeline/blob/main/docs/SELF_HOSTING.md)**.

## Licence

The self-host edition is licensed under the **GNU Affero General Public
License v3.0**. The AGPL's network-use clause means that if you offer a
**modified** Myeline as a network service, you must make your modified
source available to its users. A **commercial licence** (use without the
AGPL's obligations) is available — contact:
[hello@myeline.io](mailto:hello@myeline.io).
