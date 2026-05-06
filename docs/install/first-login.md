# Première connexion admin

Une fois la stack démarrée, voici la séquence d'actions pour
configurer Myeline avant de l'ouvrir à vos utilisateurs.

## 1. Se connecter

Aller sur `https://votre-domaine/auth/login`, saisir l'email +
mot de passe admin créés pendant l'install. La 2FA n'est pas active
au premier login — vous pouvez l'activer ensuite via `/user/account`.

## 2. Vérifier le statut système

Aller sur `/admin` (dashboard administrateur). Vous devriez voir :

- **Vue d'ensemble** : nombre d'utilisateurs (1, vous), organisations
  (1, créée automatiquement en sovereign / hybride), conversations,
  documents
- **Status** des dépendances : DB / Redis / Ollama / (Mistral selon le mode)
- **Bannière de licence** : visible si l'expiration est dans moins
  de 30 jours

## 3. Vérifier le journal d'audit

`/admin/audit` — vous devriez voir l'événement de création de votre
compte admin. Tous les actes admin sont tracés ici (création de
user, suppression de docs, modification de quotas…).

## 4. Configurer l'organisation par défaut

L'installeur a créé une organisation **My Organization** en
sovereign / hybride. Renommez-la depuis `/admin/orgs/my-organization` :

- **Name** : nom légal de votre entité (ex. *ACME Corp*)
- **Slug** : identifiant URL (ex. *acme*) — change l'URL en
  `/org/acme/...`
- **Plan** : `enterprise` par défaut en on-prem (toutes fonctions
  débloquées)

## 5. Configurer le provider IA (souverain-hybride uniquement)

Sur `/admin/orgs/<slug>` choisir le provider de synthèse :

- **Local Ollama** — `_resolved_local_model()` lit la liste des
  modèles Ollama pull et vous laisse choisir
- **Mistral cloud** — utilise la clé platform fixée à l'install ou
  votre clé d'org
- **Anthropic Claude / OpenAI / Google Gemini** — clé d'org
  obligatoire (BYOK)

Le choix est **par organisation** : différentes orgs peuvent utiliser
différents providers selon leurs besoins.

!!! tip "Bascule sans redémarrage"
    Les changements de provider sont effectifs à la prochaine
    requête RAG, sans redémarrage.

## 6. Configurer OIDC SSO (recommandé)

Pour des utilisateurs entreprise, configurez le SSO OIDC plutôt que
l'email + mot de passe :

`/org/<slug>/oidc` :

- **Issuer URL** : URL OIDC discovery de votre IdP
    - Azure AD : `https://login.microsoftonline.com/<tenant-id>/v2.0`
    - Keycloak : `https://keycloak.acme.local/realms/myeline`
    - Okta : `https://acme.okta.com/oauth2/default`
- **Client ID** + **Client Secret** : générés par votre IdP
- **Redirect URI** à autoriser dans votre IdP :
  `https://myeline.acme.local/org/<slug>/oidc/callback`
- **Scopes** : `openid email profile` au minimum

Voir [SSO entreprise (OIDC)](../admin/oidc-sso.md) pour le
walkthrough complet par IdP.

## 7. Inviter les utilisateurs

Trois voies selon votre approche :

- **Self-service** : si OIDC est configuré, les utilisateurs se
  connectent simplement via SSO et leur compte est auto-provisionné
- **Invitation manuelle** : `/admin/users → Inviter` envoie un email
  d'invitation (nécessite un mailer fonctionnel)
- **Bulk via CLI** : `flask create-user user@acme.com --org acme`
  pour des imports de masse

## 8. Activer la 2FA pour les admins

Recommandé pour tous les comptes administrateur :

1. `/user/account → Sécurité`
2. Choisir **TOTP** (Authy, Google Authenticator, 1Password)
3. Scanner le QR code, valider avec un code
4. Récupérer les codes de secours et les stocker dans votre vault

## 9. Vérifier les crons

`/admin/cron` liste les jobs planifiés. Vous devriez voir tourner :

- `record_status` (toutes les 5 min) — sample des dépendances pour
  la page `/status`
- `backup_databases` (02:30 UTC) — la première sauvegarde tournera
  cette nuit
- `check_quotas` (04:00 UTC) — alertes admin
- En sovereign : `report_to_central` est no-op (pas d'ENTERPRISE_ORG_ID)

Vous pouvez **forcer** un job depuis le dashboard pour tester
(ex. lancer `backup_databases` immédiatement et vérifier que la
sauvegarde apparaît dans `data/backups/<date>/`).

## 10. Configurer un test RAG

1. Dans votre interface utilisateur (`/user/library`), uploader un
   PDF test (ex. la documentation Myeline elle-même !)
2. Attendre ~30 s pour l'indexation (dépend de la taille)
3. `/recherche` → poser une question sur le contenu
4. Vérifier la réponse + les sources citées

Si l'embedding bloque (timeout Ollama), vérifier que le service
Ollama est UP : `podman-compose logs ollama`.

## Configuration `.env` à compléter plus tard

L'installeur peut avoir laissé certains champs vides (skip). Repassez
sur `.env` plus tard pour activer :

- **reCAPTCHA** (classic uniquement) si vous voulez la protection
  anti-bot sur l'inscription publique
- **Pangolin tunnel** si votre serveur est derrière NAT
- **Backup remote rclone** si vous ne l'avez pas configuré au démarrage

Tout changement dans `.env` nécessite un redémarrage :

```bash
podman-compose restart web cron worker
```

## Étapes suivantes

- [Sauvegarde et restauration](../operations/backup-restore.md)
- [Mise à jour](../operations/upgrade.md)
- [Recherche RAG côté utilisateur](../usage/rag-search.md)
- [FAQ](../faq.md)
