# Sovereign installation (air-gap)

Complete walkthrough to deploy Myeline in **Sovereign** mode on your
infrastructure, with no external API calls.

!!! info "Before you start"
    You must have received:

    - A licence key by email after the quote was signed (format
      `MYE-...`)
    - Access to the Myeline Git repository (your SSH key registered
      in the GitHub account we provided)

## 1. Prerequisites

Check your server matches the [server prerequisites](prerequisites.md).
For sovereign mode:

- **No GPU**: 16 vCores, 32 GB RAM, 200 GB NVMe minimum
- **With GPU**: RTX 4090 24 GB or L40S recommended for ≤ 200 users

Verify Podman 4.6+ and podman-compose are installed:

```bash
podman --version       # → 4.6.x or newer
podman-compose --version
```

If missing on Rocky / AlmaLinux 9:

```bash
sudo dnf install -y podman
sudo pip3 install podman-compose
```

Verify the **rootless Podman socket** (used for Enterprise
provisioning — not required in sovereign but useful to enable now):

```bash
systemctl --user enable --now podman.socket
ls /run/user/$(id -u)/podman/podman.sock   # must exist
```

## 2. Clone the repo

```bash
git clone -b synapse git@github.com:ClaraVnk/myeline.git
cd myeline
```

## 3. Run the guided installer

```bash
./scripts/install.sh
```

The script greets you and checks the system:

```
╔═══════════════════════════════════════════════════════════╗
║    m y e l i n e    —  guided installer                ║
║    self-hosted RAG platform, FR / EU-hosted              ║
╚═══════════════════════════════════════════════════════════╝

── System pre-checks
[✓] Podman 4.6.0 detected
[✓] 165 GB free disk space
[✓] 32 GB RAM
[✓] 16 CPU cores
```

### 3.1 Pick the mode

```
── Deployment mode

  2) Sovereign installation         ← your choice
  3) Sovereign-hybrid installation

    Your choice [2/3]: 2
[✓] Mode: sovereign
```

### 3.2 Paste the licence key

```
── Myeline licence key

    Licence key: MYE-eyJjdXN0b21lciI6IkFDTUUgQ29ycCIsImV4cGlyZXNfYXQiOiIyMDI3...
[✓] Licence format valid (cryptographic check at boot)
```

The actual cryptographic validation (Ed25519 signature) happens on
first app startup — see [Licence errors](../troubleshooting/license-errors.md)
if the app refuses to start.

### 3.3 Domain configuration

```
── Public domain

    Domain [myeline.local]: myeline.acme.local
[✓] Public URL: https://myeline.acme.local
```

The domain is used to build absolute URLs in emails and OAuth
redirect URIs (unused in sovereign but kept for consistency). In
strict air-gap, use an internal domain resolved by your enterprise
DNS.

### 3.4 Initial admin account

```
── Initial admin account

    Admin email: admin@acme.local
    Password (≥ 12 chars): ************
    Confirm: ************
[✓] Admin account configured
```

Keep this password somewhere safe. Recovery is possible later via
the `flask update-admin` CLI.

### 3.5 Mailer (forced log-only)

```
── Transactional emails (Brevo / Sendinblue)

  In sovereign mode, the mailer is forced to log-only
  (emails written to logs/mailer/ instead of being sent).

    Configure Brevo now? [y/N]: N
[i] Mailer in dry-run mode (emails written to logs).
```

In sovereign mode, you can later configure an internal MTA / SMTP
(Postfix on your network) by editing `.env` and disabling log-only
(`MAIL_DRY_RUN=false`). Custom work to discuss with us depending on
your infra.

### 3.6 AI configuration (Ollama)

```
── AI synthesis

  In sovereign mode, all synthesis goes through local Ollama.

    Ollama URL [http://ollama:11434]:
    Embedding model [bge-m3]:
    LLM synthesis model [mistral-nemo]: llama3.1:70b
[i] Remember to pull the model after install: ollama pull llama3.1:70b
```

The URL `http://ollama:11434` points to the Ollama service inside
the docker-compose. If you use an external Ollama (on a dedicated
GPU server), point to its real URL.

### 3.7 Cloud connectors

```
── Cloud storage connectors

[i] Sovereign mode: only S3 / WebDAV (API token) will be offered
    to users. No OAuth keys to configure.
```

Users will then configure their connection to your internal MinIO
or internal Nextcloud via `/user/cloud`. No global configuration
needed here.

### 3.8 Off-host backups

```
── Off-site backups (rclone)

    Configure off-site backup now? [y/N]: Y
    rclone remote (name:path): minio-internal:myeline-backups
    rclone.conf path [~/.config/rclone/rclone.conf]:
[i] See docs/BACKUP.md for the full procedure.
```

**Prerequisite**: have run `rclone config` ahead of time and set up
a remote pointing to your **internal MinIO** or **NAS**. No traffic
leaves your network if the remote is internal.

### 3.9 Optional rest (Pangolin, GPU…)

The script then asks about Pangolin (reverse-proxy tunnel —
typically not used in air-gap). Skip anything that doesn't make
sense in your context.

## 4. Start the stack

At the end, the script offers to start the stack:

```
    Start the stack now? [Y/n]: Y

[i] Pulling / building images...
[i] Starting services...
[i] Waiting for MariaDB to be ready...
[✓] MariaDB ready
[i] DB migrations + admin account creation...
On-prem (sovereign) — single-tenant org created: My Organization
[✓] Stack operational
```

Final summary:

```
═══════════════════════════════════════════════════════════
   Installation complete 🎉
═══════════════════════════════════════════════════════════

  Mode               : sovereign
  Domain             : https://myeline.acme.local
  Admin email        : admin@acme.local
  Configuration      : /home/<user>/myeline/.env
```

## 5. Pull the Ollama model

```bash
podman-compose exec ollama ollama pull llama3.1:70b
# or:
podman-compose exec ollama ollama pull mistral-nemo
podman-compose exec ollama ollama pull bge-m3   # usually pulled at boot
```

Verify the model is available:

```bash
podman-compose exec ollama ollama list
```

## 6. Configure the internal reverse proxy

Point your internal Caddy / Nginx / Traefik to `localhost:5000`:

```caddy
# /etc/caddy/Caddyfile
myeline.acme.local {
    reverse_proxy localhost:5000
    # Internal PKI — your enterprise CA
    tls /etc/pki/myeline.crt /etc/pki/myeline.key
}
```

## 7. First login

See [First admin login](first-login.md).

## Post-install checks

```bash
# Healthcheck
curl https://myeline.acme.local/healthz
# → {"status": "alive"}

# Full status
curl https://myeline.acme.local/health
# → JSON with DB / Redis / Ollama / Mistral / web (Mistral = "skipped")

# Verify no outbound traffic
sudo iptables -L OUTPUT -v
# There should be no traffic to 0.0.0.0/0 except your internal infra
```

## Next steps

- [First admin login](first-login.md) — set up the org, users,
  optionally OIDC SSO
- [Backup and restore](../operations/backup-restore.md) — verify
  the `backup_databases` cron has run
- [Licence renewal](../operations/license-renewal.md) — note the
  expiry date
