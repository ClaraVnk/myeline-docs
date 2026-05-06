# Sovereign-hybrid installation

Complete walkthrough for the **Sovereign-hybrid** edition: on-prem +
external APIs (BYOK) + cloud connectors (BYOC).

!!! info "Before you start"
    You must have received:

    - A licence key by email (format `MYE-...`, 12 months)
    - Access to the Myeline Git repository

## 1. Prerequisites

See [Server prerequisites](prerequisites.md). In sovereign-hybrid,
external LLMs handle synthesis, so:

- **CPU**: 8 vCores, 16 GB RAM, 100 GB NVMe is enough (embedding
  stays local but synthesis is offloaded to the API)
- **Optional GPU**: useful only if you also want a local Ollama
  fallback
- **Outbound HTTPS**: your firewall must allow at least
  `api.mistral.ai` (and any other AI provider you plan to use:
  `api.anthropic.com`, `api.openai.com`,
  `generativelanguage.googleapis.com`)

## 2. Clone the repo

```bash
git clone -b synapse git@github.com:ClaraVnk/myeline.git
cd myeline
```

## 3. Run the installer

```bash
./scripts/install.sh
```

Pick **option 3**:

```
── Deployment mode
    Your choice [1/2/3]: 3
[✓] Mode: hybrid
```

### 3.1 Licence + domain + admin

Same as sovereign installation — see
[Sovereign installation § 3.2-3.4](sovereign.md#32-paste-the-licence-key).

### 3.2 Mailer (Brevo)

Unlike pure sovereign, the mailer **can** send real emails via
Brevo in sovereign-hybrid mode.

```
── Transactional emails (Brevo / Sendinblue)

    Configure Brevo now? [Y/n]: Y
    Brevo API key (xkeysib-...): xkeysib-XXXXX...
    Default sender [hello@myeline.acme.local]:
    Sender name [Myeline]: ACME Knowledge
[✓] Mailer configured
```

If you skip Brevo, the mailer stays in log-only mode (like sovereign).

### 3.3 AI synthesis — BYOK

```
── AI synthesis

  Mistral is the default provider (French, EU-hosted, GDPR-aligned).
  In hybrid mode, each Enterprise organisation can switch to OpenAI
  / Anthropic / Gemini with its own key via /admin/orgs (BYOK).

    Configure Mistral platform key? [Y/n]: Y
    Mistral AI key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    Synthesis model [mistral-medium-latest]:
    Auxiliary model (HyDE/rerank) [mistral-small-latest]:
```

The Mistral "platform" key set here is used by default for new
organisations. Each org admin can then **replace** it with another
key (different Mistral account, Anthropic, OpenAI, Gemini) or
switch to local Ollama.

### 3.4 Cloud connectors — BYOC { #byoc }

This is **the key moment** of the sovereign-hybrid deployment. You
must **register your own OAuth apps** with Google / Microsoft /
Dropbox.

```
── Cloud storage connectors

  IMPORTANT — BYOC (Bring Your Own Credentials)

  You run on https://myeline.acme.local, so the redirect URIs point
  to that domain. You must register your own OAuth apps with each
  provider (Google / Microsoft / Dropbox) and authorise your URIs;
  myeline.io credentials will not work here.
```

#### Google Drive

1. **[console.cloud.google.com](https://console.cloud.google.com)** → create a project (or select one)
2. **APIs & Services → Library**: enable **Google Drive API**
3. **APIs & Services → OAuth consent screen**:
    - User type: **Internal** if everyone is in your Workspace, **External** otherwise
    - App name: *"ACME Myeline"*
    - Authorized domains: add `myeline.acme.local`
    - Scopes: add `auth/drive.readonly`
4. **Credentials → Create Credentials → OAuth client ID**:
    - Type: **Web application**
    - Authorized redirect URIs: `https://myeline.acme.local/user/cloud/gdrive/callback`
5. Paste `client_id` and `client_secret` into the installer wizard

```
    Enable Google Drive connectors? [y/N]: y
      Redirect URI to authorise: https://myeline.acme.local/user/cloud/gdrive/callback
      Google OAuth client_id: xxxxxxxxx.apps.googleusercontent.com
      Google OAuth client_secret: GOCSPX-xxxxxxxxxxxxxxxxxxxxxxxx
```

#### OneDrive (Microsoft Graph)

1. **[portal.azure.com](https://portal.azure.com)** → **Microsoft Entra ID → App registrations → New registration**
2. Name: *"ACME Myeline"*
3. Supported account types: **Single tenant** if internal, **Multitenant** otherwise
4. Redirect URI: **Web**, value `https://myeline.acme.local/user/cloud/onedrive/callback`
5. After creation: **Certificates & secrets → New client secret** — copy the *Value* (visible only once)
6. **API permissions → Add permission → Microsoft Graph → Delegated**: add `Files.Read` and `offline_access`
7. Paste `Application (client) ID` + the secret into the wizard

#### Dropbox

1. **[dropbox.com/developers/apps](https://www.dropbox.com/developers/apps)** → **Create app**
2. API: **Scoped access**
3. Type of access: **Full Dropbox** (or App folder depending on policy)
4. Name: *"acme-myeline"*
5. **Settings → OAuth 2 → Redirect URIs**: `https://myeline.acme.local/user/cloud/dropbox/callback`
6. **Permissions**: enable `files.metadata.read` and `files.content.read`
7. **App key** + **App secret** → wizard

#### kDrive Infomaniak (OAuth)

1. **[developer.infomaniak.com](https://developer.infomaniak.com)** → **Create application**
2. Redirect URI: `https://myeline.acme.local/user/cloud/kdrive/callback`
3. `Client ID` + `Client Secret` → wizard

### 3.5 Authentication

The wizard skips the "social login" section in sovereign-hybrid (see
[why](../admin/oidc-sso.md)). For enterprise authentication,
configure **OIDC SSO** after install via `/org/<slug>/oidc`.

### 3.6 The rest

rclone backups (toward MinIO or cloud S3), optional Pangolin tunnel.

## 4. Start the stack

```
    Start the stack now? [Y/n]: Y
[✓] Stack operational
```

## 5. Configure the reverse proxy

Point your Caddy / Nginx to `localhost:5000`. The OAuth redirect
URIs assume that your reverse proxy handles TLS termination on the
`myeline.acme.local` domain.

```caddy
myeline.acme.local {
    reverse_proxy localhost:5000
    # You can use Let's Encrypt if your domain is internet-reachable,
    # otherwise your internal CA.
}
```

## 6. First login

See [First admin login](first-login.md).

## Post-install checks

```bash
# Healthcheck
curl https://myeline.acme.local/healthz

# Mistral reachable?
curl https://myeline.acme.local/health
# → JSON with mistral: "healthy"

# Stripe gated
curl -I https://myeline.acme.local/payment/checkout_success
# → 404 (normal, Stripe disabled in hybrid)

# Licence page
curl https://myeline.acme.local/license-info
# → HTML page with your tier, customer, expiry
```

## Next steps

- [First admin login](first-login.md)
- [Configure enterprise OIDC](../admin/oidc-sso.md)
- [Pick the right LLM per organisation](../admin/organizations.md)
- [Off-host backups](../operations/backup-restore.md)
