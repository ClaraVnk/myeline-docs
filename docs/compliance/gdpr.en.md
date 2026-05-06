# GDPR compliance

This page documents Myeline's GDPR posture in on-prem editions
(sovereign and sovereign-hybrid).

## Roles

In sovereign and sovereign-hybrid, **you are the data controller**
(GDPR art. 4(7)). Myeline is the software publisher, not a
processor — unless you contract us for support that includes
operational access to your instance, in which case a dedicated DPA
applies for those interventions.

## Sub-processor registry

List of potential sub-processors per edition:

### Sovereign (air-gap)

**No sub-processor** — the platform runs 100 % on your premises, no
personal data is transferred to a third party.

### Sovereign-hybrid (BYOK)

Depends on the services each organisation enables:

| Service                 | Data transmitted                            | Location           |
|-------------------------|---------------------------------------------|--------------------|
| Mistral AI              | RAG queries (question text + chunks)        | EU (France)        |
| Anthropic Claude        | RAG queries                                 | US                 |
| OpenAI                  | RAG queries                                 | US                 |
| Google Gemini           | RAG queries                                 | US / EU per plan   |
| Brevo (mailer)          | Transactional emails (address + content)    | EU (Germany)       |
| OAuth provider (Google, MS, Dropbox…) | OAuth identifiers + file listing | US (typically)     |

**Each organisation** must record these sub-processors in **its own**
GDPR registry depending on the services it has activated. Myeline
mandates no choice — you can disable everything and stay fully local.

## Legal bases

| Processing                          | Legal basis                          |
|-------------------------------------|--------------------------------------|
| Account creation                    | Contract (T&Cs)                      |
| Authentication                      | Contract                             |
| RAG search                          | Contract                             |
| Cloud connectors                    | Consent (explicit OAuth)             |
| Transactional email                 | Contract                             |
| Marketing email (newsletter)        | Opt-in consent                       |
| Audit log                           | Legal obligation (GDPR art. 30)      |
| Anonymous metrics (Prometheus)      | Legitimate interest (security, perf) |

## Data subject rights

| Right                  | Implementation                                               |
|------------------------|--------------------------------------------------------------|
| **Access**             | Full export via `/account/export-data` (JSON + uploads)      |
| **Rectification**      | From the profile `/account` (name, first name, verified email) |
| **Erasure**            | `/account/delete` (soft) or admin request (hard purge)       |
| **Restriction**        | Account suspension on request (data not deleted)             |
| **Portability**        | Same export as access, open JSON format                      |
| **Objection**          | Newsletter unsubscribe + watch-alert deactivation            |
| **Automated decision** | No automated individualised decision is made by Myeline      |

Processing time: **30 days** maximum, compliant with art. 12.3.

## Retention periods

| Data                                | Duration                                 |
|-------------------------------------|------------------------------------------|
| Active account                      | Duration of the contractual relationship |
| Inactive account                    | 3 years after last login → purge         |
| Unconfirmed account                 | 30 days → automatic purge                |
| Audit log (active)                  | 13 months                                |
| Audit log (S3 archive)              | 5 years (configurable)                   |
| Application logs                    | 90 days                                  |
| Mailer logs (pure sovereign)        | 30 days (manual purge)                   |
| Backups                             | 14 days local + per your off-host policy |
| RAG conversations                   | Indefinite, manual user deletion         |

## Technical security

- **Encryption at rest**: MariaDB DB encrypted by your LUKS / FS,
  sensitive fields (OAuth tokens, alert keywords, conversations,
  TOTP secrets) encrypted at the application level via Fernet.
- **Encryption in transit**: TLS 1.2+ enforced (HTTPS, HSTS, secure
  cookies).
- **Authentication**: argon2id for passwords, optional TOTP,
  enterprise OIDC SSO.
- **Log pseudonymisation**: no email logged, only numeric `user_id`s.
- **Audit log**: every sensitive action traced.

## Breach notifications

In case of a breach affecting Myeline-side infrastructure (cloud
provider, git registry, etc.):

- Admin notification within **72 h** by signed email.
- Details: nature of the breach, data concerned, mitigations,
  recommendations.

Breaches on your own infra are not notifiable to us — that's your
perimeter.

## DPO contact

Myeline data protection officer:
[dpo@myeline.io](mailto:dpo@myeline.io).
