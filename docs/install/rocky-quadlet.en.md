# Rocky / RHEL deployment with Podman Quadlet

This page complements the standard procedure ([Sovereign install](sovereign.md)
or [Sovereign-hybrid install](sovereign-hybrid.md)) for operators who
want to **drop `podman-compose`** in favour of **Podman Quadlet**
(systemd `.container`, `.network`, `.volume` units) on a Rocky Linux 10
/ RHEL 9+ / Alma 9+ host with **SELinux Enforcing**.

> **Who is this for?** Sysadmins comfortable with systemd. If you're
> new to Podman, stick with `podman-compose` — it's simpler and that's
> what the official bundle ships by default.

Quadlet vs `podman-compose`:

| | `podman-compose` | Quadlet |
|---|---|---|
| Container startup | manual after reboot | auto via systemd |
| Log handling | `podman logs <ctr>` | `journalctl --user -u myeline-web` |
| Healthchecks | defined in compose | defined in `.container` or via systemd |
| Restart policy | docker-compose-style | native systemd (`Restart=always`, `RestartSec=10`) |
| Multi-app on same host | risk of name / port collisions | systemd-level isolation per unit |
| Auto-update via `auto-update` policy | not supported | native (`AutoUpdate=registry`) |

---

## Specific prerequisites

- **Linger enabled** for the rootless Podman user:
  ```bash
  sudo loginctl enable-linger $USER
  ```
  Without this, your containers drop the moment your SSH session ends.

- **`ip_unprivileged_port_start=80`** to bind 80/443 as rootless:
  ```bash
  echo 'net.ipv4.ip_unprivileged_port_start=80' | sudo tee /etc/sysctl.d/90-unprivileged-ports.conf
  sudo sysctl --system
  ```

- **podman 4.6+** (Rocky 10 ships 5.x — OK)

---

## Generating Quadlet units from the compose

The official Myeline bundle ships `docker-compose.yml` + sovereign/hybrid
variants. Quadlet has no direct import — you write the units manually,
mirroring the compose services.

User unit location: `~/.config/containers/systemd/`

Service-by-service conversion pattern:

```ini
# ~/.config/containers/systemd/myeline-web.container
[Unit]
Description=Myeline web (Flask)
After=network-online.target myeline-mariadb.service myeline-redis.service

[Container]
ContainerName=myeline-web
Image=rg.fr-par.scw.cloud/myeline/web:v1.0.2
Environment=FLASK_ENV=production
EnvironmentFile=%h/myeline/.env
PublishPort=5000:5000
Network=myeline-net.network:alias=web
Volume=%h/myeline/uploads:/app/uploads:Z
Volume=%h/myeline/data:/app/data:Z
Volume=%h/myeline/logs:/app/logs:Z
Volume=%h/myeline/app/myeline_license_pubkey.pem:/app/app/myeline_license_pubkey.pem:ro,Z

HealthCmd=curl -fsS -o /dev/null --max-time 5 http://127.0.0.1:5000/healthz
HealthInterval=30s
HealthTimeout=10s
HealthRetries=3
HealthStartPeriod=20s

[Service]
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

Enable + start:
```bash
systemctl --user daemon-reload
systemctl --user start myeline-web
systemctl --user enable myeline-web
```

---

## SELinux Enforcing — known pitfalls

### 1. The `:Z` suffix is not idempotent in rootless

On RHEL/Rocky, Volume options `:Z` (container-private label) and `:z`
(shared label) **change the MCS category on every container restart**.
Result: a volume formatted yesterday becomes unreadable today because
the inodes carry a category that no longer matches the current process
label. Typical symptom: MariaDB looping at startup with "cannot read
`mysql` table", or Ollama unable to find its models.

**Recommendation for services with a persistent datadir** (`mariadb`,
`ollama`, `redis` if AOF enabled):

1. **Pre-label the host directory once**:
   ```bash
   chcon -R -t container_file_t -l s0 ~/myeline/data/mysql
   chown -R 999:999 ~/myeline/data/mysql   # mariadb internal UID
   ```

2. **Disable relabel in the unit**:
   ```ini
   [Container]
   SecurityLabelDisable=true
   Volume=%h/myeline/data/mysql:/var/lib/mysql      # no :Z, no :z
   ```

3. **For ephemeral or buffer volumes** (uploads, app logs), keeping
   `:Z` is fine — no drift since files get rewritten regularly.

### 2. socket-proxy: AVC denial on `connectto`

The `tecnativa/docker-socket-proxy` proxy runs HAProxy which must
establish a connection to the host's Podman socket. With the default
process type, SELinux blocks this `connectto`. Symptom: every Podman
API call from the `web` container returns 503.

Fix in `myeline-socket-proxy.container`:
```ini
[Container]
SecurityLabelType=container_runtime_t
# (NOT SecurityLabelDisable=true — we keep the type, we just change its class)
```

### 3. Check denials

```bash
sudo ausearch -m AVC -ts recent
sudo journalctl -t setroubleshoot --since "1 hour ago"
```

**Never** flip SELinux to `Permissive` or `Disabled` as a workaround —
fix the context/boolean instead.

---

## Rootless UIDs — calculating the host ↔ container mapping

In rootless Podman, container internal UIDs are remapped to host UIDs
via `/etc/subuid`. To prepare a bind-mount with the correct permissions
(typical case: `chown` of the mariadb datadir before first boot), you
must **compute the host UID** that corresponds.

Formula:
```
host_uid = subuid_start + container_uid - 1
```

Get `subuid_start` from `/etc/subuid`:
```bash
SUBUID_START=$(grep "^$USER:" /etc/subuid | cut -d: -f2)
echo "subuid_start = $SUBUID_START"
```

Concrete examples:

| Service | Container UID | Calculation (subuid_start=589824) | Host UID |
|---|---|---|---|
| mariadb (`mysql` user) | 999 | 589824 + 999 - 1 | **590822** |
| redis (`redis` user) | 999 | 589824 + 999 - 1 | **590822** |
| Myeline web (`appuser`) | 1000 | 589824 + 1000 - 1 | **590823** |

Typical use before first mariadb boot:
```bash
sudo chown -R 590822:590822 ~/myeline/data/mysql
```

---

## ACL for `appuser` writes

The `appuser` user in the container (host UID = `subuid_start + 999`)
must be able to **write** to `~/myeline/data/` for `data/backups/`,
`data/cache/`, etc. If you leave that directory owned by your host
user (typically UID 1001), writes are denied.

Recommended solution: **POSIX ACLs** rather than recursive `chown`
(which would break `data/mysql/` and `data/ollama/` that have their
own ownership):

```bash
sudo dnf install -y acl   # if not already installed

APPUSER_HOST_UID=$((SUBUID_START + 999))

# Current permissions (existing files)
sudo setfacl -m u:${APPUSER_HOST_UID}:rwx ~/myeline/data

# Default permissions (inherited by future files)
sudo setfacl -d -m u:${APPUSER_HOST_UID}:rwx ~/myeline/data
```

Verify the ACLs are set:
```bash
getfacl ~/myeline/data
# Should show:
# user:590823:rwx
# default:user:590823:rwx
```

---

## Healthchecks with `curl` (not `wget`)

The Myeline image bundles `curl` but **not `wget`**. If your Quadlet
unit has `HealthCmd=wget -qO- ...`, the healthcheck fails and the
service is permanently marked `unhealthy`. Use:

```ini
HealthCmd=curl -fsS -o /dev/null --max-time 5 http://127.0.0.1:5000/healthz
```

`/healthz` returns 200 immediately (liveness only) — preferred for
healthchecks. `/health` does DB + Redis + Ollama roundtrips and can
return 503 during temporary dependency slowdowns.

---

## DNS aliases in Quadlet

To prefix your `ContainerName=` (e.g. `myeline-mariadb`) while keeping
short hostnames in the app's `.env` (e.g.
`DATABASE_URL=...@mariadb:3306/...`), declare a DNS alias on the
network:

```ini
# ~/.config/containers/systemd/myeline-mariadb.container
[Container]
ContainerName=myeline-mariadb
Network=myeline-net.network:alias=mariadb
```

Without this alias, the `web` container looks up `mariadb` in DNS and
only finds `myeline-mariadb`. Symptom: DB connection error at `web`
boot.

---

## Caddy as HTTPS reverse-proxy

### Local healthcheck

If your `Caddyfile` applies a global HTTPS redirect, an HTTP
healthcheck against `http://localhost/` gets a 308. The container
healthcheck flips to `unhealthy` permanently.

Solution: expose a non-redirected HTTP endpoint locally in the
Caddyfile:

```caddy
http://localhost {
    respond /healthz "OK" 200
}

myeline.acme.local {
    reverse_proxy 127.0.0.1:5000
}
```

Then in the Caddy `.container`:
```ini
HealthCmd=curl -fsS -o /dev/null --max-time 5 http://localhost/healthz
```

### Optional `www.`

If you declare:

```caddy
myeline.acme.local, www.myeline.acme.local {
    reverse_proxy 127.0.0.1:5000
}
```

…but the DNS for `www.myeline.acme.local` doesn't exist, Let's
Encrypt **fails** the HTTP-01 challenge for `www` on every renewal,
logs an error in a loop, and still obtains the main cert (Caddy
handles the degradation). To avoid the noise, split into two blocks:

```caddy
myeline.acme.local {
    reverse_proxy 127.0.0.1:5000
}

# Uncomment if you've created the www A/CNAME DNS record
# www.myeline.acme.local {
#     redir https://myeline.acme.local{uri}
# }
```

---

## Auto-update via Podman

Quadlet supports native auto-update (Watchtower equivalent):

```ini
[Container]
Image=rg.fr-par.scw.cloud/myeline/web:v1.0.2
AutoUpdate=registry
```

Then enable the system timer:

```bash
systemctl --user enable --now podman-auto-update.timer
```

By default the timer fires once a day. It pulls the latest version
of the specified tag, and if the digest changed, recreates the
container. **Caveat**: if you pin a specific tag (e.g. `:v1.0.2`),
auto-update only happens if Myeline re-publishes under that tag —
which doesn't happen for immutable semver tags. To track releases
automatically, use `:latest` or a moving tag (`:v1`), at the cost of
losing control over upgrade timing.

---

## Logs and debugging

```bash
# Follow logs live
journalctl --user -u myeline-web -f

# Logs from last 10 minutes, all Myeline services
journalctl --user --since "10 minutes ago" -u "myeline-*"

# Detailed service state
systemctl --user status myeline-mariadb

# Inspect an SELinux AVC
sudo ausearch -m AVC -ts recent | sudo audit2why
```

---

## Migrating from existing `podman-compose`

If you're already running compose and want to switch to Quadlet
without losing data:

1. Identify your host bind-mount volumes (`./data`, `./uploads`,
   `./logs` typically, from `docker-compose.yml`).
2. Stop the compose stack: `podman-compose down` (without `-v`!).
3. Pre-label the datadirs (see SELinux section).
4. Write your Quadlet units pointing to the **same paths** on the
   host.
5. `systemctl --user daemon-reload && systemctl --user start myeline-mariadb`
6. Check healthchecks + logs before starting the others.
