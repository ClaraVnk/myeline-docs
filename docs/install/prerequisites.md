# Pré-requis serveur

Sizing recommandé pour faire tourner Myeline avec une expérience
interactive (réponse RAG sous 5 s perçus).

!!! warning "À retenir"
    Une seule requête RAG sature **2-4 cœurs** pendant 0,5-2 s
    pour l'embedding + 4-8 cœurs pendant 5-30 s pour la synthèse
    locale. **8 cœurs est le plancher pratique** ; 4 cœurs ne tient
    pas sous usage interactif.

## TL;DR par profil

| Profil | vCPU | RAM | Disque | GPU | Notes |
|---|---|---|---|---|---|
| **Démo / 1 utilisateur** | 8 | 16 GB | 50 GB SSD | – | Embedding CPU local |
| **Souverain ≤ 20 utilisateurs** | 16 | 32 GB | 200 GB NVMe | – | Mistral-Nemo CPU = 15-40 s/requête |
| **Souverain ≤ 200 utilisateurs** | 24 | 64 GB | 500 GB NVMe | RTX 4090 24 GB ou L40S | GPU **fortement recommandé** |
| **Souverain large / Llama 70B** | 32 | 128 GB | 1 TB NVMe | 2× L40S 48 GB | Llama 3.1 70B Q4 ou Mixtral 8×7B |
| **Souverain-hybride ≤ 100 utilisateurs** | 8 | 16 GB | 100 GB SSD | – | Synthèse déportée API BYOK, embedding local |

## Détail des consommations

### CPU

| Charge | Demande |
|---|---|
| Web + worker (idle) | 1-2 cœurs |
| **Embedding (bge-m3 CPU)** | **2-4 cœurs à 100 % pendant 0,5-2 s par requête** |
| ChromaDB HNSW search | 1-2 cœurs pendant ~100 ms |
| Synthèse API externe (souverain-hybride) | 0 (réseau-bound) |
| **Synthèse LLM Ollama CPU** | **4-8 cœurs à 100 % pendant 5-30 s** |
| MariaDB | 1 cœur (2+ en pic) |
| Cron (la plupart < 30 s) | bursts only |

**Single-user reality** : même un serveur idle sature 4 cœurs
pendant la durée de la requête (embedding + HNSW + synthèse en
chaîne).

**Multi-user reality** : avec 8 cœurs vous gérez confortablement
1-2 requêtes simultanées actives, au-delà ça queue. Pour 4+
utilisateurs simultanés actifs, prévoyez 16+ cœurs.

### Mémoire (idle, sans LLM local)

| Composant | RAM |
|---|---|
| `web` (gunicorn 9 workers) | ~1,5 GB |
| `worker` (RQ) | ~200 MB |
| `cron` (supercronic) | ~50 MB |
| `mariadb` (InnoDB buffer) | ~1-1,5 GB |
| `redis` | ~100 MB |
| `ollama` avec bge-m3 chargé | ~700 MB |
| ChromaDB hot index (HNSW) | 200-800 MB |
| **Total idle** | **~4 GB** |
| Headroom peak / pulls / backups | +50 % |
| **Recommandé minimum** | **8 GB** |

### Mémoire (souverain — LLM local)

Ajouter le modèle Ollama choisi :

| Modèle local | Quantif. | RAM résident | Notes |
|---|---|---|---|
| `mistral-nemo` (12 B) | Q4_K_M | ~7 GB | Défaut. Décent, CPU lent |
| `mistral-nemo` | Q8 | ~13 GB | Meilleure qualité |
| `mixtral-8x7b` | Q4 | ~26 GB | CPU ≥ 30 s/réponse, GPU recommandé |
| `llama3.1:70b` | Q4 | ~40 GB | Top-tier local, GPU obligatoire |

**Avec GPU** les modèles vivent en VRAM, pas en RAM système ; les
chiffres ci-dessus restent grossièrement valables pour le budget
mémoire global, mais la latence s'améliore de 5-20× selon la carte.

### Disque

| Usage | Taille | Notes |
|---|---|---|
| OS + images containers | 5-10 GB | Image Python slim ~600 MB ; les modèles Ollama dominent |
| Modèles Ollama | bge-m3 600 MB · mistral-nemo Q4 7 GB | Stockés dans `data/ollama/` |
| ChromaDB | ~10 KB / chunk indexé | 100 k chunks ≈ 1 GB ; croît linéairement |
| MariaDB | 100 MB → 5 GB | Audit log + conversations dominent |
| Uploads | non borné | Plafonné par les plans (max_file_size) |
| Backups (rétention 30 j) | 2-5× la donnée live | `backup_databases` cron |
| Logs | ~10 MB / jour | Rotation par Docker / journald |

NVMe vs SSD SATA : meilleur p99 pour ChromaDB (HNSW seeks) et MariaDB.

### Réseau

- **Outbound** (souverain-hybride uniquement) — provider IA + Brevo
  + GHCR : ~10 Mbps soutenu, plus pendant les image pulls.
- **Inbound** : 100 Mbps confortable pour quelques dizaines
  d'utilisateurs simultanés.
- Latence vers le provider IA (Mistral Paris/Frankfurt, Anthropic /
  OpenAI / Gemini US) : viser < 100 ms.
- En souverain pur : **aucun trafic sortant** (air-gap).

## Référence hardware

| Profil | Hardware type | Spec | Ordre de prix |
|---|---|---|---|
| Démo / Souverain ≤ 20 users (CPU only) | VPS ou bare-metal | 8-16 vCores / 32 GB / 200 GB NVMe | 20-50 €/mois |
| Souverain ≤ 200 users + GPU | Bare-metal | Xeon + RTX 4090 24 GB / 64 GB / 1 TB NVMe | 250-400 €/mois |
| Souverain large / Llama 70B | Bare-metal | 2× L40S 48 GB / 128 GB / 1 TB NVMe | 800-1500 €/mois |
| Souverain-hybride ≤ 100 users | VPS | 8 vCores / 16 GB / 100 GB NVMe | 20-40 €/mois |

!!! tip "Pratique"
    En souverain pur, le coût hardware est dominé par le GPU (si
    requis). En souverain-hybride, l'embedding tient sur un petit
    VPS — la facture LLM passe directement chez votre provider IA.

## OS et runtime

- **Linux** : Rocky / AlmaLinux 9, Debian 12, Ubuntu 22.04 LTS+
- **Containers** : Podman 4.6+ rootless **recommandé**, Docker
  fonctionne aussi (`docker-compose` v2)
- **systemd** : requis pour Podman pods unit-managed (sovereign)
- **Python 3.11+** si vous tournez hors containers (path officiel : containers)
- **SELinux** Enforcing : OK — le compose file labellise les volumes
  avec `:z`
- **GPU** (souverain avec GPU) : NVIDIA drivers 535+, NVIDIA Container
  Toolkit installé ; Ollama auto-détecte
