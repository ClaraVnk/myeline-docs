# First admin login

Once the stack is up, here is the recommended sequence of actions
to configure Myeline before opening it to your users.

## 1. Log in

Go to `https://your-domain/auth/login`, enter the admin email +
password created during install. 2FA is not active on first login —
you can enable it later via `/user/account`.

## 2. Check system status

Go to `/admin` (admin dashboard). You should see:

- **Overview**: number of users (1, you), organisations (1, auto-created
  in sovereign / hybrid), conversations, documents
- **Status** of dependencies: DB / Redis / Ollama / (Mistral depending on the mode)
- **Licence banner**: visible if expiry is within 30 days

## 3. Check the audit log

`/admin/audit` — you should see the creation event for your admin
account. Every admin action is traced here (user creation, doc
deletion, quota changes…).

## 4. Configure the default organisation

The installer created an organisation **My Organization** in
sovereign / hybrid. Rename it from `/admin/orgs/my-organization`:

- **Name**: legal name of your entity (e.g. *ACME Corp*)
- **Slug**: URL identifier (e.g. *acme*) — changes the URL to
  `/org/acme/...`
- **Plan**: `enterprise` by default in on-prem (all features unlocked)

## 5. Configure the AI provider (sovereign-hybrid only)

On `/admin/orgs/<slug>` pick the synthesis provider:

- **Local Ollama** — `_resolved_local_model()` reads the list of
  pulled Ollama models and lets you pick one
- **Mistral cloud** — uses the platform key set at install or your
  org-specific key
- **Anthropic Claude / OpenAI / Google Gemini** — org key required
  (BYOK)

The choice is **per organisation**: different orgs can use different
providers depending on their needs.

!!! tip "Switch without restart"
    Provider changes take effect on the next RAG query, no restart
    needed.

## 6. Enterprise SSO

Enterprise SSO is available on request (bespoke on-premise service) —
contact hello@myeline.io.

## 7. Invite users

Three paths depending on your approach:

- **Manual invitation**: `/admin/users → Invite` sends an invitation
  email (requires a working mailer)
- **Bulk via CLI**: `flask create-user user@acme.com --org acme`
  for mass imports

## 8. Enable 2FA for admins

Recommended for every administrator account:

1. `/user/account → Security`
2. Pick **TOTP** (Authy, Google Authenticator, 1Password)
3. Scan the QR code, validate with a code
4. Save the backup codes in your vault

## 9. Verify the cron jobs

`/admin/cron` lists scheduled jobs. You should see:

- `record_status` (every 5 min) — dependency sampling for the
  `/status` page
- `backup_databases` (02:30 UTC) — first backup will run tonight
- `check_quotas` (04:00 UTC) — admin alerts
- In sovereign mode: `report_to_central` is a no-op (no
  ENTERPRISE_ORG_ID)

You can **force** a job from the dashboard to test (e.g. run
`backup_databases` immediately and check that the backup appears
in `data/backups/<date>/`).

## 10. Run a test RAG query

Myeline has no manual upload — the library is fed via cloud
connectors (`/user/cloud`) or RSS / web scrapers
(`/user/scrapers`). For a first test:

1. Connect a drive that contains a few PDFs (internal S3 / WebDAV
   in sovereign; Google Drive via service account, or
   OneDrive / Dropbox with your OAuth apps, in sovereign-hybrid).
2. Trigger a manual sync from `/user/cloud` ("Sync").
3. Wait ~30-90 s for indexing (depends on size).
4. `/user` (search) → ask a question about the content.
5. Check the answer + cited sources.

If embedding hangs (Ollama timeout), check that Ollama is up:
`podman-compose logs ollama`.

## `.env` configuration to complete later

The installer may have left some fields blank (skip). Revisit `.env`
later to enable:

- **Pangolin tunnel** if your server is behind NAT
- **rclone backup remote** if you didn't configure it at install time

Any change in `.env` requires a restart:

```bash
podman-compose restart web cron worker
```

## Next steps

- [Backup and restore](../operations/backup-restore.md)
- [Upgrade](../operations/upgrade.md)
- [End-user RAG search](../concepts/rag-search.md)
- [FAQ](../faq.md)
