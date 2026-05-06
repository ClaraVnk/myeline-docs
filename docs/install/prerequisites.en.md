# Server prerequisites

Recommended sizing to run Myeline with an interactive experience
(perceived RAG response under 5 s).

!!! warning "Key takeaway"
    A single RAG query saturates **2-4 cores** for 0.5-2 s for
    embedding + 4-8 cores for 5-30 s for local synthesis. **8 cores
    is the practical floor**; 4 cores cannot sustain interactive use.

## TL;DR by profile

| Profile | vCPU | RAM | Disk | GPU | Notes |
|---|---|---|---|---|---|
| **Demo / 1 user** | 8 | 16 GB | 50 GB SSD | – | CPU embedding + Mistral cloud |
| **Sovereign ≤ 20 users** | 16 | 32 GB | 200 GB NVMe | – | Mistral-Nemo CPU = 15-40 s/query |
| **Sovereign ≤ 200 users** | 24 | 64 GB | 500 GB NVMe | RTX 4090 24 GB or L40S | GPU **strongly recommended** |
| **Sovereign large / Llama 70B** | 32 | 128 GB | 1 TB NVMe | 2× L40S 48 GB | Llama 3.1 70B Q4 or Mixtral 8×7B |

## Detailed consumption

### CPU

| Load | Demand |
|---|---|
| Web + worker (idle) | 1-2 cores |
| **Embedding (bge-m3 CPU)** | **2-4 cores at 100 % for 0.5-2 s per query** |
| ChromaDB HNSW search | 1-2 cores for ~100 ms |
| Mistral cloud synthesis | 0 (network-bound) |
| **Ollama CPU LLM synthesis** | **4-8 cores at 100 % for 5-30 s** |
| MariaDB | 1 core (2+ at peak) |
| Cron (most jobs < 30 s) | bursts only |

**Single-user reality**: even an idle server saturates 4 cores during
the request (embedding + HNSW + synthesis chained).

**Multi-user reality**: with 8 cores you handle 1-2 active concurrent
requests comfortably; beyond that they queue. For 4+ active concurrent
users plan 16+ cores.

### Memory (sovereign — local LLM)

Add the chosen Ollama model:

| Local model | Quant. | RAM resident | Notes |
|---|---|---|---|
| `mistral-nemo` (12 B) | Q4_K_M | ~7 GB | Default. Decent, CPU slow |
| `mistral-nemo` | Q8 | ~13 GB | Better quality |
| `mixtral-8x7b` | Q4 | ~26 GB | CPU ≥ 30 s/answer, GPU recommended |
| `llama3.1:70b` | Q4 | ~40 GB | Top-tier local, GPU mandatory |

**With GPU** the models live in VRAM, not system RAM; the figures
above remain roughly valid for the global memory budget, but latency
improves 5-20× depending on the card.

### Disk

| Usage | Size | Notes |
|---|---|---|
| OS + container images | 5-10 GB | Python slim image ~600 MB; Ollama models dominate |
| Ollama models | bge-m3 600 MB · mistral-nemo Q4 7 GB | Stored in `data/ollama/` |
| ChromaDB | ~10 KB / indexed chunk | 100k chunks ≈ 1 GB; grows linearly |
| MariaDB | 100 MB → 5 GB | Audit log + conversations dominate |
| Uploads | unbounded | Capped per plan (`max_file_size`) |
| Backups (30 d retention) | 2-5× live data | `backup_databases` cron |
| Logs | ~10 MB / day | Rotation via Podman / journald |

NVMe vs SATA SSD: better p99 for ChromaDB (HNSW seeks) and MariaDB.
SATA SSD suffices for small/medium deployments.

### Network

- **Outbound** to Mistral / Brevo / GHCR: ~10 Mbps sustained, more
  during image pulls
- **Inbound**: 100 Mbps comfortable up to medium tier
- Latency to Mistral (Paris/Frankfurt): aim for < 50 ms

## OS and runtime

- **Linux**: Rocky / AlmaLinux 9, Debian 12, Ubuntu 22.04 LTS+
- **Containers**: Podman 4.6+ rootless **recommended**, Docker
  works too (`docker-compose` v2)
- **systemd**: required for Podman pods unit-managed (sovereign)
- **Python 3.11+** if you run outside containers (official path:
  containers)
- **SELinux** Enforcing: OK — the compose file labels volumes with
  `:z`
- **GPU** (sovereign with GPU): NVIDIA drivers 535+, NVIDIA Container
  Toolkit installed; Ollama auto-detects
