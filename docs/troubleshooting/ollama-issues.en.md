# Ollama issues

Ollama always handles **embedding** (model `bge-m3`) and, in pure
sovereign, **also synthesis**. Ollama failures block the entire RAG
search — it's the most sensitive component.

## Quick diagnostic

```bash
# Loaded models
podman exec ollama ollama list

# Test embedding
curl -s http://localhost:11434/api/embed -d '{"model":"bge-m3","input":"test"}' | jq

# Test synthesis (pure sovereign)
curl -s http://localhost:11434/api/generate -d '{"model":"mistral-nemo","prompt":"Hello","stream":false}' | jq

# GPU state (if applicable)
nvidia-smi
```

`/health` on the Myeline side must show `"ollama": "ok"`. If
`degraded` or `down`, see below.

## `Model not found`

**Symptom**: `404 from Ollama: model 'bge-m3' not found`.

**Cause**: the model wasn't pulled or was deleted.

**Fix**:

```bash
podman exec ollama ollama pull bge-m3
podman exec ollama ollama pull mistral-nemo:latest    # sovereign synthesis
```

Configure pull at startup via `entrypoint` or in
`docker-compose.yml`:

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

**Symptom**: Ollama kills the process, container restarts in a loop,
logs show `OOMKilled`.

**Causes**:

- Model too big for available RAM/VRAM (Mixtral 8×7B = 47 GB,
  Llama 3.1 70B = 40 GB in Q4).
- Several models loaded in parallel.

**Fix**:

1. Check available VRAM (`nvidia-smi`) or RAM (`free -h`).
2. Pick a quantised model (`Q4_K_M` instead of `Q8_0`).
3. Set `OLLAMA_NUM_PARALLEL=1` to limit to 1 model at a time.
4. Set `OLLAMA_MAX_LOADED_MODELS=2` to auto-unload.

See [Server prerequisites](../install/prerequisites.md) for sizing.

## Excess latency

**Symptom**: `/health` `degraded`, RAG queries > 30 s.

**Causes**:

- CPU-only on a synthesis model (Mistral-Nemo on CPU = 20-40 s).
- Underpowered GPU (RTX 3060 12 GB for Llama 70B = inevitable CPU
  swap).
- Saturated disk (models reloaded from slow SSD on every request
  because they don't fit in RAM).

**Fix**:

```bash
# Ollama startup GPU profile
podman exec ollama env | grep OLLAMA
# OLLAMA_HOST=0.0.0.0:11434
# OLLAMA_NUM_GPU=1     # number of GPUs to use
# OLLAMA_KEEP_ALIVE=24h # keep models loaded
```

Setting `OLLAMA_KEEP_ALIVE=24h` avoids repeated reloads. For
CPU-only, accept the latency or add a GPU.

## GPU not detected

**Symptom**: despite a card being present, Ollama runs on CPU.

**Verification**:

```bash
# Host side
nvidia-smi
podman info | grep -i nvidia

# Test the runtime
podman run --rm --device nvidia.com/gpu=all nvidia/cuda:12.5.0-base-ubuntu22.04 nvidia-smi
```

**Common causes**:

- `nvidia-container-toolkit` not installed / misconfigured.
- SELinux denials on device access. Check
  `sudo ausearch -m AVC -ts recent` and adjust the context with
  `chcon` or a dedicated policy module — do **not** disable SELinux.
- Host driver too old for the CUDA version of the Ollama image.

## Verbose logs

To debug a session:

```bash
podman exec ollama env OLLAMA_DEBUG=1 ollama serve
podman logs -f ollama
```

Restore `OLLAMA_DEBUG=0` afterwards to avoid filling the disk.
