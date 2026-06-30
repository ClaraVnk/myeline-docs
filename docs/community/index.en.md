# Community / Self-host (open source)

The **Community** edition is the **open source** core of Myeline,
released under the **AGPL-3.0** licence on GitHub. You host it **on
your own server**, for free: no licence to buy, no account to create,
no calls back to our servers. It is the **single-user** edition for
anyone who wants their own sovereign, inspectable RAG knowledge base.

[:fontawesome-brands-github: The code on GitHub](https://github.com/ClaraVnk/myeline){ .md-button .md-button--primary }
[:material-book-open-variant: Self-host guide](https://github.com/ClaraVnk/myeline/blob/main/docs/SELF_HOSTING.md){ .md-button }

!!! info "This page, the wiki, and the product"
    The Community product is **English-only** (UI and repo docs in
    English). This wiki stays **bilingual FR / EN**. For **detailed
    usage** of the features (RAG search, library, connectors,
    conversations, watch), see the
    [Concepts (all editions)](../concepts/index.md): they are shared
    across every edition.

## Who is it for?

- **Developers and self-hosters** who want a self-hosted, hackable
  personal RAG with no SaaS dependency.
- **Personal data sovereignty**: documents, vector index and history
  stay on your machine. In local LLM mode (Ollama), **no data leaves
  the host**.
- **Technical evaluation** ahead of a larger on-premise rollout.

## What you get

- **Full RAG**: top-K retrieval with MMR diversification, clickable
  citations, strict/loose mode, time filtering, multi-turn
  conversations, single-document chat.
- **Cloud storage connectors** (Google Drive, OneDrive, Dropbox,
  kDrive, S3, WebDAV, Notion, Zotero) using **your own credentials**
  (BYOC).
- **RSS / web sources** scraped and indexed.
- **Multi-LLM BYOK + fully local LLM via Ollama**: Mistral, OpenAI,
  Claude, Gemini with your key, or Ollama (`bge-m3` + synthesis)
  completely offline.
- **Digests and watch alerts**, **TOTP 2FA**.

The full feature breakdown lives on the edition page:
[Community / Self-Host edition](../editions/community.md).

## Get started in 3 commands

Prerequisites: Podman 4.6+ and `podman-compose` (or Docker + Compose).

```bash
git clone https://github.com/ClaraVnk/myeline.git ~/Projects/myeline
cd ~/Projects/myeline
cp .env.example .env        # then edit — see docs/SELF_HOSTING.md

podman-compose -f docker-compose.yml up -d --build
podman-compose -f docker-compose.yml exec web flask create-admin
```

Then open <http://localhost:5000>. The complete walkthrough
(prerequisites, key generation, LLM choice, per-connector OAuth apps)
is in the
**[self-host guide](https://github.com/ClaraVnk/myeline/blob/main/docs/SELF_HOSTING.md)**.

## What next?

- **Understand usage**: [Concepts (all editions)](../concepts/index.md).
- **Compare editions**: [Choose your edition](../editions/index.md).
- **Need multi-tenant, teams or contractual support?** See the
  [Cloud (SaaS)](../cloud/index.md) or the
  [On-premise / Sovereign](../editions/sovereign.md) offer (quote-based).

## Licence

Self-host edition under the **GNU Affero General Public License v3.0**.
The AGPL network clause requires that, if you expose a **modified**
Myeline as a network service, you make the modified source available to
its users. A **commercial licence** (use without the AGPL obligations)
is available: [hello@myeline.io](mailto:hello@myeline.io).
