# Installation souverain-hybride

Walkthrough complet pour l'édition **Souverain-hybride** : on-prem +
APIs externes BYOK + connecteurs cloud BYOC.

!!! info "Avant de commencer"
    Vous devez avoir reçu :

    - Une clé de licence par email (format `MYE-...`, 12 mois)
    - Un accès au repo Git Myeline

## 1. Pré-requis

Voir [Pré-requis serveur](prerequisites.md). En souverain-hybride,
les LLMs externes prennent en charge la synthèse, donc :

- **CPU** : 8 vCores, 16 GB RAM, 100 GB NVMe suffisent (l'embedding
  reste local mais le synthèse est déportée sur l'API)
- **GPU optionnel** : utile uniquement si vous voulez aussi un
  fallback Ollama local
- **Sortie HTTPS** : votre firewall doit autoriser au minimum
  `api.mistral.ai` (et tout autre provider IA que vous comptez
  utiliser : `api.anthropic.com`, `api.openai.com`,
  `generativelanguage.googleapis.com`)

## 2. Cloner le repo

```bash
git clone -b synapse git@github.com:ClaraVnk/myeline.git
cd myeline
```

## 3. Lancer l'installeur

```bash
./scripts/install.sh
```

Choisir l'**option 3** :

```
── Choix du mode de déploiement
    Votre choix [1/2/3] : 3
[✓] Mode : hybrid
```

### 3.1 Licence + domaine + admin

Identique à l'installation souveraine — voir
[Installation souveraine § 3.2-3.4](sovereign.md#32-coller-la-cle-de-licence).

### 3.2 Mailer (Brevo)

Contrairement au souverain pur, le mailer **peut** envoyer de vrais
emails via Brevo en souverain-hybride.

```
── Emails transactionnels (Brevo / Sendinblue)

    Configurer Brevo maintenant ? [O/n] : O
    Clé API Brevo (xkeysib-...) : xkeysib-XXXXX...
    Expéditeur par défaut [hello@myeline.acme.local] :
    Nom d'expéditeur [Myeline] : ACME Knowledge
[✓] Mailer configuré
```

Si vous ne configurez pas Brevo, le mailer reste en log-only
(comme en sovereign).

### 3.3 Synthèse IA — BYOK

```
── Synthèse IA

  Mistral est le provider par défaut (français, hébergé UE,
  RGPD-compatible). En hybride, chaque organisation Enterprise
  pourra basculer vers OpenAI / Anthropic / Gemini avec sa propre
  clé via /admin/orgs (BYOK).

    Configurer la clé Mistral plateforme ? [O/n] : O
    Clé Mistral AI : xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    Modèle de synthèse [mistral-medium-latest] :
    Modèle auxiliaire (HyDE/rerank) [mistral-small-latest] :
```

La clé "platform" Mistral fixée ici est utilisée par défaut pour les
nouvelles organisations. Chaque org admin pourra ensuite la **remplacer**
par une autre clé (Mistral autre compte, Anthropic, OpenAI, Gemini)
ou basculer vers Ollama local.

### 3.4 Connecteurs cloud — BYOC {#byoc}

C'est **le moment-clé** du déploiement souverain-hybride. Vous devez
**créer vos propres apps OAuth** chez Google / Microsoft / Dropbox.

```
── Connecteurs de stockage cloud

  IMPORTANT — BYOC (Bring Your Own Credentials)

  Vous tournez sur https://myeline.acme.local, donc les redirect URIs
  pointent vers ce domaine. Vous devez créer vos propres apps OAuth
  chez chaque provider (Google / Microsoft / Dropbox) et autoriser
  vos URIs ; les credentials de myeline.io ne fonctionneront pas ici.
```

#### Google Drive

1. **[console.cloud.google.com](https://console.cloud.google.com)** → créer un projet (ou en sélectionner un)
2. **APIs & Services → Library** : activer **Google Drive API**
3. **APIs & Services → OAuth consent screen** :
    - User type : **Internal** si tout le monde est dans votre Workspace, **External** sinon
    - App name : *"ACME Myeline"*
    - Authorized domains : ajouter `myeline.acme.local`
    - Scopes : ajouter `auth/drive.readonly`
4. **Credentials → Create Credentials → OAuth client ID** :
    - Type : **Web application**
    - Authorized redirect URIs : `https://myeline.acme.local/user/cloud/gdrive/callback`
5. Coller `client_id` et `client_secret` dans le wizard installeur

```
    Activer les connecteurs Google Drive ? [o/N] : o
      Redirect URI à autoriser : https://myeline.acme.local/user/cloud/gdrive/callback
      Google OAuth client_id : xxxxxxxxx.apps.googleusercontent.com
      Google OAuth client_secret : GOCSPX-xxxxxxxxxxxxxxxxxxxxxxxx
```

#### OneDrive (Microsoft Graph)

1. **[portal.azure.com](https://portal.azure.com)** → **Microsoft Entra ID → App registrations → New registration**
2. Name : *"ACME Myeline"*
3. Supported account types : **Single tenant** si interne, **Multitenant** sinon
4. Redirect URI : **Web**, valeur `https://myeline.acme.local/user/cloud/onedrive/callback`
5. Après création : **Certificates & secrets → New client secret** — copier la *Value* (visible une fois seulement)
6. **API permissions → Add permission → Microsoft Graph → Delegated** : ajouter `Files.Read` et `offline_access`
7. Coller `Application (client) ID` + le secret dans le wizard

#### Dropbox

1. **[dropbox.com/developers/apps](https://www.dropbox.com/developers/apps)** → **Create app**
2. API : **Scoped access**
3. Type of access : **Full Dropbox** (ou App folder selon politique)
4. Name : *"acme-myeline"*
5. **Settings → OAuth 2 → Redirect URIs** : `https://myeline.acme.local/user/cloud/dropbox/callback`
6. **Permissions** : activer `files.metadata.read` et `files.content.read`
7. **App key** + **App secret** → wizard

#### kDrive Infomaniak (OAuth)

1. **[developer.infomaniak.com](https://developer.infomaniak.com)** → **Create application**
2. Redirect URI : `https://myeline.acme.local/user/cloud/kdrive/callback`
3. `Client ID` + `Client Secret` → wizard

### 3.5 Authentication

Le wizard saute la section "connexion sociale" en souverain-hybride
(voir [SSO entreprise (OIDC)](../admin/oidc-sso.md)). Pour
l'authentification entreprise, configurez **OIDC SSO** après
l'install via `/org/<slug>/oidc`.

### 3.6 Reste

Backups rclone (vers MinIO ou S3 cloud), Pangolin tunnel optionnel.

## 4. Démarrer la stack

```
    Démarrer la stack maintenant ? [O/n] : O
[✓] Stack opérationnelle
```

## 5. Configurer le reverse-proxy

Pointez votre Caddy / Nginx vers `localhost:5000`. Les redirect URIs
OAuth supposent que votre reverse-proxy gère la terminaison TLS sur
le domaine `myeline.acme.local`.

```caddy
myeline.acme.local {
    reverse_proxy localhost:5000
    # Vous pouvez utiliser Let's Encrypt si votre domaine est
    # accessible Internet, sinon votre AC interne.
}
```

## 6. Première connexion

Voir [Première connexion admin](first-login.md).

## Vérifications post-install

```bash
# Healthcheck
curl https://myeline.acme.local/healthz

# Mistral atteignable ?
curl https://myeline.acme.local/health
# → JSON avec mistral: "healthy"

# Stripe gaté
curl -I https://myeline.acme.local/payment/checkout_success
# → 404 (normal, Stripe désactivé en hybride)

# Page licence
curl https://myeline.acme.local/license-info
# → page HTML avec votre tier, customer, expiry
```

## Étapes suivantes

- [Première connexion admin](first-login.md)
- [Configurer OIDC entreprise](../admin/oidc-sso.md)
- [Activer le bon LLM par organisation](../admin/organizations.md)
- [Sauvegarde off-host](../operations/backup-restore.md)
