# Licence errors

The Myeline licence system uses **offline Ed25519 signatures** (no
network call to validate). Most common errors and their fixes.

## `LicenseExpiredError`

**Symptom**: the `/license-info` banner shows "Expired", paid
features (external AI synthesis, public cloud connectors) are
blocked.

**Cause**: your licence key is past its expiry date (12 months max
by construction).

**Fix**:

1. Contact [hello@myeline.io](mailto:hello@myeline.io) with your
   customer reference to receive a new key.
2. Paste the new key in `/admin/license` (or via `.env`
   `LICENSE_KEY=…` then restart).
3. Verify the new expiry date on `/license-info`.

See [Licence renewal](../operations/license-renewal.md) for the full
procedure.

## `LicenseRevokedError`

**Symptom**: message "Licence revoked by the publisher".

**Cause**: Myeline has published a revocation for this key (typically
following a leak, a product return, or an issuance error).

**Fix**:

- In pure sovereign (air-gap), the revocation is **embedded in the
  next image release** (file `app/data/licenses_revoked.json`).
  Update the image (see [Upgrade](../operations/upgrade.md)). If
  you can't update, contact us: we issue a replacement key after
  verification.
- In sovereign-hybrid, same — but we can also push an out-of-band
  revocation (dedicated image rebuild).

## `LicenseInvalidSignatureError`

**Symptom**: message "Invalid licence signature".

**Possible causes**:

1. **Corrupted key** during copy-paste (whitespace, stray newlines).
   Re-copy from the original email in raw mode.
2. **Wrong public key** server-side. The verification public key is
   embedded in the image (`app/services/license.py`). If you've
   modified that file, restore it.
3. **Tampering**: someone tried to alter the payload. This is a
   security alert — audit who has manipulated `.env`.

## `LicenseClockSkewError`

**Symptom**: message "Server clock skewed".

**Cause**: the system date diverges by more than 24 h from reality —
which makes `not_before` / `expires_at` window validation
impossible.

**Fix**:

```bash
# Check
date
timedatectl status

# Fix via NTP
sudo systemctl restart chronyd
sudo chronyc tracking

# If no Internet (air-gap), point chronyd at an internal NTP server
```

**Important in pure sovereign**: NTP must work internally (no
Internet needed, but a reliable reachable time server is required).

## `LicenseModeMismatchError`

**Symptom**: "Deployment mode incompatible with the licence" at
boot.

**Cause**: `DEPLOYMENT_MODE=sovereign-hybrid` in `.env` but the
issued key is for pure `sovereign` (or vice-versa).

**Fix**: align both. Either change `DEPLOYMENT_MODE` or request a
new key for the right mode.

## Diagnostic

```bash
# Decode your licence payload (without verifying the signature)
podman exec myeline-web python -c "
import os, json, base64
key = os.environ['LICENSE_KEY']
payload_b64 = key.split('.')[0]
print(json.dumps(json.loads(base64.urlsafe_b64decode(payload_b64 + '==')), indent=2))
"
```

Shows `customer`, `mode`, `not_before`, `expires_at`, `tier`.
Useful to diagnose a mismatch without contacting support.
