# Cartographie ISO 27001 / 27701

Cette page indique comment les fonctionnalités natives de Myeline
contribuent aux contrôles de l'**Annexe A de l'ISO/IEC 27001:2022**
(et par extension à la 27701 sur la vie privée).

⚠️ **Important** : Myeline est une *brique* dans votre SMSI. La
certification ISO 27001 s'attribue à votre organisation, pas à un
produit. Cette cartographie aide vos équipes audit à identifier
quels contrôles sont déjà servis par la plateforme et lesquels
restent à votre charge.

## Annexe A — Contrôles organisationnels (clause 5)

| Contrôle | Libellé | Couverture Myeline |
|----------|---------|---------------------|
| 5.1      | Politiques de SI | Hors scope (politique = vous) |
| 5.10     | Acceptable use | CGU + EULA fournis |
| 5.15     | Contrôle d'accès | Login, RBAC org (owner/admin/member), Email + mot de passe + TOTP (SSO sur demande) |
| 5.16     | Gestion des identités | Création / désactivation / purge automatique |
| 5.17     | Information d'authentification | Argon2id mot de passe, TOTP optionnel |
| 5.18     | Droits d'accès | Modèle RBAC + audit des élévations |
| 5.34     | PII | Pseudonymisation logs, chiffrement champ-niveau, droit à l'effacement |

## Annexe A — Contrôles humains (clause 6)

Hors scope — la formation et la sensibilisation sont à votre charge.

## Annexe A — Contrôles physiques (clause 7)

Hors scope — vous hébergez Myeline.

## Annexe A — Contrôles technologiques (clause 8)

| Contrôle | Libellé | Couverture Myeline |
|----------|---------|---------------------|
| 8.2      | Privileges access | `/admin/audit` trace les élévations |
| 8.3      | Restriction d'accès info | Scopes ChromaDB par utilisateur / org |
| 8.5      | Authentification sécurisée | Mot de passe + TOTP (SSO sur demande), rate limiting |
| 8.6      | Capacity management | Quotas par plan, alertes 80 % / 100 % |
| 8.7      | Protection malware | MIME check + ClamAV optionnel sur uploads |
| 8.8      | Vulnérabilités techniques | Image OCI à jour, deps audités (`pip-audit`) |
| 8.9      | Configuration management | 12-factor (env vars), validation schéma au démarrage |
| 8.10     | Suppression d'info | Hard purge `flask delete-user`, suppression cascade FK |
| 8.11     | Masquage de données | Pseudonymisation logs, chiffrement applicatif (Fernet) |
| 8.12     | Prévention fuite données | Egress filtering recommandé (souverain-hybride), air-gap natif (souverain) |
| 8.13     | Backup | Cron quotidien `backup_databases` + procédure restauration documentée |
| 8.15     | Logging | `/admin/audit` + logs JSON structurés stdout |
| 8.16     | Monitoring | `/healthz`, `/health`, `/metrics`, `/status` |
| 8.17     | Synchronisation horloge | NTP requis (validation signatures licence) |
| 8.18     | Logiciels privilégiés | `flask` CLI restreint au shell admin |
| 8.20     | Sécurité réseau | TLS obligatoire, HSTS, CSP stricte |
| 8.21     | Sécurité services réseau | Reverse proxy en front (Pangolin/Nginx) |
| 8.22     | Ségrégation réseaux | Souverain : VLAN dédié, sans Internet |
| 8.23     | Filtrage Web | Hors scope (à vous) |
| 8.24     | Cryptographie | TLS 1.2+, Argon2id, Fernet (AES-128-GCM), Ed25519 (licence) |
| 8.25     | Cycle de vie dev sécurisé | Bandit, isort, flake8, tests > 80 % couverture |
| 8.26     | Sécurité applicative | CSRF, rate limiting, headers sécurité, validation input |
| 8.28     | Codage sécurisé | Revues de code, tests automatisés, lint strict |
| 8.32     | Change management | Versioning OCI, migrations idempotentes, rollback documenté |
| 8.33     | Test info | Bases de test isolées, données synthétiques |
| 8.34     | Audit | Logs immuables, archivage off-host > 180 j |

## ISO/IEC 27701 — extensions vie privée

| Contrôle | Couverture Myeline |
|----------|---------------------|
| 6.5      | Journal des activités de traitement (`/admin/audit`) |
| 6.6      | Gestion des incidents (procédure 72 h documentée) |
| 7.2      | Bases légales (voir [RGPD](gdpr.md)) |
| 7.3      | Information personnes concernées (CGU + politique de confidentialité) |
| 7.4      | Choix et consentement (cookies, OAuth, watch alerts) |
| 7.5      | Droits personnes concernées (export, rectification, effacement) |
| 8.5      | Sous-traitants (registre fourni — voir [RGPD](gdpr.md)) |

## Modèle de plan d'audit

Disponible sur demande à [hello@myeline.io](mailto:hello@myeline.io)
sous forme de tableur Excel pré-rempli, avec :

- Liste de chaque contrôle Annexe A
- Statut : couvert / partiellement couvert / hors scope Myeline
- Preuve technique (commande / endpoint / fichier de config)
- Champ libre pour vos commentaires d'audit

Utile pour préparer un audit ISO 27001 ou un questionnaire de
conformité fournisseur.
