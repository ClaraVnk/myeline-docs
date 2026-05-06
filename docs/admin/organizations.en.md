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
