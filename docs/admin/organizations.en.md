# Organisations and users

An **organisation** is a shared workspace inside Myeline: dedicated
ChromaDB collection, common library, members with roles, default
LLM provider configurable per org (in sovereign-hybrid).

## Role model

| Role    | Can...                                                                  |
|---------|--------------------------------------------------------------------------|
| Owner   | everything (including deleting the org, transferring ownership)          |
| Admin   | manage members, OIDC, quotas, re-index the collection                    |
| Member  | upload documents, run searches, create conversations                     |

A user can belong to **several organisations**. The org selector at
the top right changes the RAG search context (target ChromaDB
collection).

## Create an organisation

`/admin/orgs/new` — enter the name, slug (URL segment), owner email.
If the email matches an existing account, that user is added as
owner; otherwise an account is created and an invitation email is
sent (in pure sovereign, log-only — fetch the link from
`logs/mailer/`).

## Invite members

From `/org/<slug>/members`, the owner or an admin can:

- Invite by email (sends an enrolment link)
- Promote a member to admin
- Remove a member
- Transfer the owner role (action protected by confirmation)

## Account deletion (GDPR)

- **Soft delete**: `/account/delete` user-side — anonymises email,
  first name, last name and frees personal resources. The audit
  trail is kept 13 months (GDPR registry, legal duration).
- **Hard delete**: `flask delete-user --email=…` admin CLI — full
  purge, including the personal ChromaDB collection.

See [GDPR compliance](../compliance/gdpr.md) for details.

## Per-org AI providers (Enterprise / Sovereign-hybrid)

From `/org/<slug>/admin`, the owner and admins of an Enterprise org
can configure two independent providers:

- **LLM** (answer synthesis): "Fournisseur IA" card —
  Mistral / OpenAI / Claude / Gemini with your key, or stay on the
  platform default.
- **Embedding** (vectorisation of documents and queries for the
  `org_{id}_shared` collection): "Fournisseur d'embedding (BYOK)"
  card.

### "Fournisseur d'embedding (BYOK)" card

| Provider    | Default model                  | Dim   | Notes                                |
|-------------|-------------------------------|-------|--------------------------------------|
| Mistral     | `mistral-embed`               | 1024  | 🇫🇷 EU-hosted, GDPR                 |
| OpenAI      | `text-embedding-3-small`      | 1536  | `-3-large` 3072-dim also supported   |
| Voyage      | `voyage-3`                    | 1024  | Top quality                          |
| Cohere      | `embed-multilingual-v3.0`     | 1024  | Multilingual (100+ languages)        |

To activate or switch providers:

1. Click the target provider tile, paste the API key (`sk-...`,
   `voyage-...`, etc.), and leave the "model" field empty to use the
   default — or specify a variant.
2. If the `org_{id}_shared` collection already exists under a different
   provider/model, the form **refuses the switch** and prints the
   server-side command to run.
3. Wipe + reindex:
   ```bash
   flask wipe-org-embed-collection --org <id> --confirm
   # Re-activate the provider via /org/<slug>/admin
   flask reindex-embeddings
   ```
4. During reindexing, searches stay functional but may be degraded
   (results converge back on the new vector space as chunks are
   re-embedded).

**Security.** Keys are encrypted (`org.embed_api_key_enc`, Fernet) and
never echoed back by the UI — only a `••••••••` mask confirms a key is
stored. The embedding key is independent from the LLM key; they can be
changed separately.

**Scope.** Embed BYOK routing only applies to the org's shared
collection. Each member's personal library (`user_{id}_personal`) and
the public collection (`shared`) stay on the deployment-wide default
provider, because a user may have an individual subscription outside
the org.

**Disable.** The "Désactiver" button clears the three columns
(`embed_provider`, `embed_api_key_enc`, `embed_model`) and reverts the
org to the default. Previously indexed vectors stay in place — re-index
afterwards to realign the vector space with the new default provider.
