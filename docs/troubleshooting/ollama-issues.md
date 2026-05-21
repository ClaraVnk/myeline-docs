# Problèmes Ollama

Ollama assure **toujours l'embedding** (modèle `bge-m3` par défaut —
[voir alternatives supportées](#changer-de-modele-dembedding)) et, en
souverain pur, **aussi la synthèse**. Les pannes Ollama bloquent
toute la recherche RAG — c'est le composant le plus sensible.

## Diagnostic rapide

```bash
# Modèles chargés
podman exec ollama ollama list

# Tester l'embedding
curl -s http://localhost:11434/api/embed -d '{"model":"bge-m3","input":"test"}' | jq

# Tester la synthèse (souverain pur)
curl -s http://localhost:11434/api/generate -d '{"model":"mistral-nemo","prompt":"Hello","stream":false}' | jq

# État GPU (si applicable)
nvidia-smi
```

`/health` côté Myeline doit afficher `"ollama": "ok"`. Si `degraded`
ou `down`, voir ci-dessous.

## `Model not found`

**Symptôme** : `404 from Ollama: model 'bge-m3' not found`.

**Cause** : le modèle n'a pas été pulled ou a été supprimé.

**Résolution** :

```bash
podman exec ollama ollama pull bge-m3
podman exec ollama ollama pull mistral-nemo:latest    # synthèse souverain
```

Configurer le pull au démarrage via `entrypoint` ou via le
`docker-compose.yml` :

```yaml
ollama:
  image: ollama/ollama:latest
  command: |
    sh -c "
      /bin/ollama serve &
      sleep 5
      /bin/ollama pull bge-m3
      /bin/ollama pull mistral-nemo
      wait
    "
```

## OOM (Out of Memory)

**Symptôme** : Ollama tue le process, le conteneur redémarre en
boucle, logs `OOMKilled`.

**Causes** :

- Modèle trop gros pour la RAM/VRAM disponible (Mixtral 8×7B = 47 GB,
  Llama 3.1 70B = 40 GB en Q4).
- Plusieurs modèles chargés en parallèle.

**Résolution** :

1. Vérifier la VRAM dispo (`nvidia-smi`) ou la RAM (`free -h`).
2. Choisir un modèle quantifié (`Q4_K_M` au lieu de `Q8_0`).
3. Variable `OLLAMA_NUM_PARALLEL=1` pour limiter à 1 modèle à la fois.
4. Variable `OLLAMA_MAX_LOADED_MODELS=2` pour décharger
   automatiquement.

Voir [Pré-requis serveur](../install/prerequisites.md) pour le sizing.

## Latence excessive

**Symptôme** : `/health` `degraded`, requêtes RAG > 30 s.

**Causes** :

- CPU only sur un modèle de synthèse (Mistral-Nemo en CPU = 20-40 s).
- GPU sous-dimensionné (RTX 3060 12 GB pour un Llama 70B = swap CPU
  inévitable).
- Disque saturé (modèles chargés depuis SSD lent à chaque requête
  faute de pouvoir tenir en RAM).

**Résolution** :

```bash
# Profil GPU au démarrage Ollama
podman exec ollama env | grep OLLAMA
# OLLAMA_HOST=0.0.0.0:11434
# OLLAMA_NUM_GPU=1     # nombre de GPU à utiliser
# OLLAMA_KEEP_ALIVE=24h # garder les modèles chargés
```

Activer `OLLAMA_KEEP_ALIVE=24h` évite les rechargements répétés.
Pour le CPU pur, accepter les latences ou ajouter un GPU.

## GPU non détecté

**Symptôme** : malgré une carte présente, Ollama tourne en CPU.

**Vérification** :

```bash
# Côté host
nvidia-smi
podman info | grep -i nvidia

# Test du runtime
podman run --rm --device nvidia.com/gpu=all nvidia/cuda:12.5.0-base-ubuntu22.04 nvidia-smi
```

**Causes courantes** :

- `nvidia-container-toolkit` non installé / mal configuré.
- SELinux denials sur l'accès au device. Vérifier
  `sudo ausearch -m AVC -ts recent` puis adapter le contexte avec
  `chcon` ou un module de policy dédié — ne **pas** désactiver
  SELinux.
- Driver host trop ancien pour la version CUDA de l'image Ollama.

## Logs verbeux

Pour debugger une session :

```bash
podman exec ollama env OLLAMA_DEBUG=1 ollama serve
podman logs -f ollama
```

Restaurer ensuite à `OLLAMA_DEBUG=0` pour ne pas saturer le disque.

## Changer de modèle d'embedding

Le défaut `bge-m3` couvre la majorité des cas (multilingue, 1024 dim,
qualité/vitesse optimale). Si vous préférez un autre modèle (corpus
mono-langue, hardware contraint, exigence qualité ↑), voir le tableau
complet et la procédure dans
[Installation souveraine § Choix du modèle d'embedding](../install/sovereign.md#choix-du-modele-dembedding).

**Résumé express** :

```bash
# 1. Pull le nouveau modèle dans le conteneur Ollama
podman exec ollama ollama pull mxbai-embed-large

# 2. Mettre à jour .env
sed -i 's/^OLLAMA_EMBED_MODEL=.*/OLLAMA_EMBED_MODEL=mxbai-embed-large/' .env

# 3. Restart web pour relire .env (down + up, pas restart — restart
# ne relit pas env_file dans podman-compose)
podman-compose down && podman-compose up -d

# 4. Réindexer le corpus existant avec le nouveau modèle
podman-compose exec web flask reindex-embeddings
```

!!! warning "Changement de dimension = reindex obligatoire"
    Passer de `bge-m3` (1024) à `nomic-embed-text` (768) sans réindexer
    provoque un mismatch `embedding dimension mismatch: collection has
    1024, query has 768` sur la première requête. Toujours lancer la
    réindexation **avant** de réouvrir l'application aux utilisateurs.
