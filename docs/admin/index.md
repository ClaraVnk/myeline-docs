# Administration

L'interface d'administration de Myeline est accessible à
`https://<votre-domaine>/admin` pour les comptes flaggés `is_admin`.

Le dashboard `/admin` regroupe :

- État de la plateforme (utilisateurs, organisations, files RQ,
  ChromaDB, Ollama, mailer)
- Bannière de licence (édition, expiration, statut)
- Indicateurs RAG (volume de requêtes, erreurs)
- Liens rapides vers les sections détaillées

## Sections couvertes

- **[Organisations et utilisateurs](organizations.md)** — créer une
  organisation, inviter des membres, gérer les rôles owner / admin /
  member, supprimer un compte (RGPD).
- **[SSO entreprise (OIDC)](oidc-sso.md)** — brancher Azure AD / Okta /
  Keycloak / Authentik comme IdP pour une organisation.
- **[Quotas et plans](quotas.md)** — limites par plan, alertes 80 % /
  100 %, surcharge admin.
- **[Journal d'audit](audit-log.md)** — qui a fait quoi, quand,
  archivage off-host des entrées > 180 jours.
- **[Connecteurs cloud](cloud-connectors.md)** — activation par
  édition, configuration des credentials BYOC en souverain-hybride.

## Bonnes pratiques

- Créer **au moins deux comptes admin** (résilience en cas de perte
  de mot de passe).
- Activer la **2FA TOTP** pour tous les admins (`/account/2fa`).
- Vérifier `/admin` au moins une fois par semaine, ou brancher
  `/health` à votre supervision (Uptime Kuma, Zabbix, Prometheus…).
- Conserver un **export régulier** du journal d'audit hors site (voir
  [opérations / sauvegarde](../operations/backup-restore.md)).
