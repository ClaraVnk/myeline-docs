# Déploiement Rocky / RHEL avec Podman Quadlet

Cette page complète la procédure standard ([Installation depuis l'image](sovereign.md)
ou [Installation souverain-hybride](sovereign-hybrid.md)) pour les
opérateurs qui veulent **abandonner `podman-compose`** au profit de
**Podman Quadlet** (unités systemd `.container`, `.network`, `.volume`)
sur un host Rocky Linux 10 / RHEL 9+ / Alma 9+ avec **SELinux Enforcing**.

> **Pour qui ?** Profil sysadmin habitué à systemd. Si vous découvrez
> Podman, restez sur `podman-compose` — c'est plus simple et c'est
> ce que le bundle officiel ship par défaut.

Quadlet vs `podman-compose` :

| | `podman-compose` | Quadlet |
|---|---|---|
| Démarrage des conteneurs | manuel après reboot | auto via systemd |
| Gestion logs | `podman logs <ctr>` | `journalctl --user -u myeline-web` |
| Healthchecks | définis dans le compose | définis dans le `.container` ou via systemd |
| Restart policy | docker-compose-style | systemd natif (`Restart=always`, `RestartSec=10`) |
| Multi-app sur le même host | risque de collisions de noms / ports | isolement systemd par unité |
| Auto-update via `auto-update` policy | non supporté | natif (`AutoUpdate=registry`) |

---

## Pré-requis spécifiques

- **Linger activé** pour l'utilisateur Podman rootless :
  ```bash
  sudo loginctl enable-linger $USER
  ```
  Sans ça, vos containers tombent dès que la session SSH se ferme.

- **`ip_unprivileged_port_start=80`** pour binder 80/443 en rootless :
  ```bash
  echo 'net.ipv4.ip_unprivileged_port_start=80' | sudo tee /etc/sysctl.d/90-unprivileged-ports.conf
  sudo sysctl --system
  ```

- **podman 4.6+** (Rocky 10 ship 5.x — OK)

---

## Génération des fichiers Quadlet à partir du compose

Le bundle officiel Myeline embarque `docker-compose.yml` + variantes
sovereign/hybrid. Quadlet n'a pas d'import direct — vous devez écrire
les unités manuellement, en miroir des services du compose.

Localisation des unités utilisateur : `~/.config/containers/systemd/`

Pattern de conversion service par service :

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

Activer + démarrer :
```bash
systemctl --user daemon-reload
systemctl --user start myeline-web
systemctl --user enable myeline-web
```

---

## SELinux Enforcing — pièges connus

### 1. Le suffixe `:Z` n'est pas idempotent en rootless

Sur RHEL/Rocky, les options de remontage Volume `:Z` (label privé
conteneur) et `:z` (label partagé) **changent la catégorie MCS à
chaque restart du conteneur**. Conséquence : un volume formaté la
veille devient illisible aujourd'hui parce que les inodes ont une
catégorie qui ne matche plus le label courant du process. Symptôme
typique : MariaDB qui boucle au démarrage avec « cannot read
`mysql` table », ou Ollama qui ne retrouve plus ses modèles.

**Recommandation pour les services à datadir persistant**
(`mariadb`, `ollama`, `redis` si AOF activé) :

1. **Pré-labéliser le dossier hôte une seule fois** :
   ```bash
   chcon -R -t container_file_t -l s0 ~/myeline/data/mysql
   chown -R 999:999 ~/myeline/data/mysql   # UID interne mariadb
   ```

2. **Désactiver le relabel dans l'unité** :
   ```ini
   [Container]
   SecurityLabelDisable=true
   Volume=%h/myeline/data/mysql:/var/lib/mysql      # sans :Z, sans :z
   ```

3. **Pour les volumes éphémères ou tampons** (uploads,
   logs applicatifs), garder `:Z` est OK — pas de drift puisque
   les fichiers sont régulièrement réécrits.

### 2. socket-proxy : AVC denial sur `connectto`

Le proxy `tecnativa/docker-socket-proxy` tourne en HAProxy qui doit
établir une connexion vers le socket Podman du host. Avec le type de
process par défaut, SELinux bloque ce `connectto`. Symptôme : tous
les appels API Podman du conteneur `web` retournent 503.

Fix dans `myeline-socket-proxy.container` :
```ini
[Container]
SecurityLabelType=container_runtime_t
# (PAS SecurityLabelDisable=true — on garde le type, on change juste sa classe)
```

### 3. Vérifier les denials

```bash
sudo ausearch -m AVC -ts recent
sudo journalctl -t setroubleshoot --since "1 hour ago"
```

**Ne jamais** passer SELinux en `Permissive` ou `Disabled` comme
contournement — corrigez le contexte/booléen à la place.

---

## Healthchecks avec `curl` (pas `wget`)

L'image Myeline embarque `curl` mais **pas `wget`**. Si votre unité
Quadlet a un `HealthCmd=wget -qO- ...`, le healthcheck plante et le
service est marqué `unhealthy` en boucle. Utilisez :

```ini
HealthCmd=curl -fsS -o /dev/null --max-time 5 http://127.0.0.1:5000/healthz
```

`/healthz` répond 200 immédiatement (liveness uniquement) — c'est
celui à privilégier pour les healthchecks. `/health` fait des
roundtrips DB + Redis + Ollama et peut renvoyer 503 en cas de
ralentissement temporaire d'une dépendance.

---

## DNS aliases dans Quadlet

Pour pouvoir préfixer vos `ContainerName=` (ex. `myeline-mariadb`)
tout en gardant les hostnames courts dans le `.env` de l'app (ex.
`DATABASE_URL=...@mariadb:3306/...`), déclarez un alias DNS sur le
network :

```ini
# ~/.config/containers/systemd/myeline-mariadb.container
[Container]
ContainerName=myeline-mariadb
Network=myeline-net.network:alias=mariadb
```

Sans cet alias, le conteneur `web` cherche `mariadb` en DNS et ne
trouve que `myeline-mariadb`. Symptôme : erreur de connexion DB au
boot du `web`.

---

## Caddy comme reverse-proxy HTTPS

### Healthcheck local

Si votre `Caddyfile` applique un redirect HTTPS global, un
healthcheck en HTTP sur `http://localhost/` se prend un 308. Le
healthcheck conteneur tombe en `unhealthy` permanent.

Solution : exposer un endpoint HTTP local non-redirigé dans le
Caddyfile :

```caddy
http://localhost {
    respond /healthz "OK" 200
}

myeline.acme.local {
    reverse_proxy 127.0.0.1:5000
}
```

Puis dans le `.container` Caddy :
```ini
HealthCmd=curl -fsS -o /dev/null --max-time 5 http://localhost/healthz
```

### `www.` optionnel

Si vous déclarez :

```caddy
myeline.acme.local, www.myeline.acme.local {
    reverse_proxy 127.0.0.1:5000
}
```

…mais que le DNS de `www.myeline.acme.local` n'existe pas, Let's
Encrypt **échoue** sur le défi HTTP-01 pour `www` à chaque
renouvellement, log une erreur en boucle, et obtient quand même le
cert principal (Caddy gère la dégradation). Pour éviter le bruit,
séparez en deux blocs :

```caddy
myeline.acme.local {
    reverse_proxy 127.0.0.1:5000
}

# Décommentez si vous avez créé le A/CNAME DNS de www
# www.myeline.acme.local {
#     redir https://myeline.acme.local{uri}
# }
```

---

## Auto-update via Podman

Quadlet supporte l'auto-update natif (équivalent Watchtower) :

```ini
[Container]
Image=rg.fr-par.scw.cloud/myeline/web:v1.0.2
AutoUpdate=registry
```

Puis activez le timer système :

```bash
systemctl --user enable --now podman-auto-update.timer
```

Par défaut, le timer fire une fois par jour. Il pull la dernière
version du tag spécifié, et si le digest a changé, recrée le
conteneur. **Attention** : si vous pinnez un tag spécifique
(ex. `:v1.0.2`), il n'y a auto-update que si Myeline re-publie sous
ce tag — ce qui n'arrive pas pour les tags semver immuables. Pour
suivre les releases automatiquement, utilisez `:latest` ou un tag
moving (`:v1`), au prix de perdre le contrôle du moment d'upgrade.

---

## Logs et debug

```bash
# Suivre les logs en direct
journalctl --user -u myeline-web -f

# Logs des 10 dernières minutes, tous services Myeline
journalctl --user --since "10 minutes ago" -u "myeline-*"

# État détaillé d'un service
systemctl --user status myeline-mariadb

# Inspecter un AVC SELinux
sudo ausearch -m AVC -ts recent | sudo audit2why
```

---

## Migration depuis `podman-compose` existant

Si vous tournez déjà sur compose et voulez basculer en Quadlet sans
perdre vos données :

1. Identifiez vos volumes bind-mount sur l'host (`./data`, `./uploads`,
   `./logs` typiquement, depuis `docker-compose.yml`).
2. Arrêtez la stack compose : `podman-compose down` (sans `-v` !).
3. Pré-labélisez les datadirs (cf. section SELinux).
4. Écrivez vos unités Quadlet pointant vers les **mêmes paths** sur
   l'host.
5. `systemctl --user daemon-reload && systemctl --user start myeline-mariadb`
6. Vérifiez les healthchecks + logs avant de démarrer les autres.
