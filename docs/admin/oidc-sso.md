# SSO entreprise (OIDC)

Myeline supporte **OpenID Connect** pour permettre à vos utilisateurs
de se connecter avec leur compte Azure AD / Entra ID, Okta, Keycloak,
Authentik ou tout IdP conforme OIDC.

L'OIDC SSO est **inclus dans toutes les éditions on-prem** (souverain
et souverain-hybride). En revanche, le **social login**
(Google / Microsoft / Apple via boutons publics) est **désactivé**
sur ces deux éditions — voir [Choisir son édition](../editions/index.md).

## Pré-requis côté IdP

Créer une application OIDC (client confidentiel) avec :

- **Redirect URI** : `https://<votre-domaine>/org/<slug>/oidc/callback`
- **Scopes** : `openid`, `profile`, `email`
- **Type de réponse** : `code` (Authorization Code Flow + PKCE)

Récupérer :

- L'**URL de discovery** (`https://idp.example.com/.well-known/openid-configuration`)
- Le **client_id** et le **client_secret**

## Configuration côté Myeline

Dans `/admin/orgs/<slug>/oidc` :

1. Coller l'URL de discovery → Myeline auto-fournit les endpoints
   `authorization`, `token`, `jwks`.
2. Coller `client_id` et `client_secret` (chiffrés au repos via
   Fernet, dérivé de `CLOUD_TOKEN_KEY`).
3. Tester avec un compte → l'utilisateur est provisionné
   automatiquement à la première connexion (rôle `member` par défaut,
   promouvable ensuite par un admin).

## Mappage des claims

| Claim OIDC      | Champ Myeline       |
|-----------------|---------------------|
| `email`         | `User.email`        |
| `given_name`    | `User.first_name`   |
| `family_name`   | `User.last_name`    |
| `sub`           | `User.oidc_sub`     |
| `groups` (opt.) | rôle (mapping admin)|

Le mapping de groupes vers rôles Myeline (member / admin) est
configurable dans la même page — chaîne CSV des groupes IdP qui
doivent être mappés sur `admin`.

## Forcer le SSO

Une fois l'OIDC validé, vous pouvez **désactiver la connexion
par email + mot de passe** pour les membres de l'organisation —
ils seront forcés vers l'IdP. Le owner conserve un fallback mot de
passe pour ne pas perdre l'accès en cas de panne IdP.
