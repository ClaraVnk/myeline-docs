# Enterprise SSO (OIDC)

Myeline supports **OpenID Connect** so your users can log in with
their Azure AD / Entra ID, Okta, Keycloak, Authentik or any
OIDC-compliant IdP account.

OIDC SSO is **included in every on-prem edition** (sovereign and
sovereign-hybrid). However, **social login** (Google / Microsoft /
Apple via public buttons) is **disabled** in those two editions —
see [Choose your edition](../editions/index.md).

## IdP-side prerequisites

Create an OIDC application (confidential client) with:

- **Redirect URI**: `https://<your-domain>/org/<slug>/oidc/callback`
- **Scopes**: `openid`, `profile`, `email`
- **Response type**: `code` (Authorization Code Flow + PKCE)

Collect:

- The **discovery URL**
  (`https://idp.example.com/.well-known/openid-configuration`)
- The **client_id** and the **client_secret**

## Myeline-side configuration

In `/admin/orgs/<slug>/oidc`:

1. Paste the discovery URL → Myeline auto-fills the
   `authorization`, `token`, `jwks` endpoints.
2. Paste `client_id` and `client_secret` (encrypted at rest via
   Fernet, derived from `CLOUD_TOKEN_KEY`).
3. Test with one account → the user is auto-provisioned on first
   login (default `member` role, promotable later by an admin).

## Claim mapping

| OIDC claim     | Myeline field       |
|----------------|---------------------|
| `email`        | `User.email`        |
| `given_name`   | `User.first_name`   |
| `family_name`  | `User.last_name`    |
| `sub`          | `User.oidc_sub`     |
| `groups` (opt.)| role (admin mapping)|

The IdP-groups → Myeline-roles (member / admin) mapping is
configurable on the same page — a CSV string of IdP groups that
should be mapped to `admin`.

## Force SSO

Once OIDC is validated, you can **disable email + password login**
for org members — they will be forced to the IdP. The owner keeps
a password fallback so access is preserved if the IdP goes down.
