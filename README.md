# docker-sglang

Run [SGLang](https://github.com/sgl-project/sglang) in Docker with HuggingFace Hub model loading and a host-mounted model cache.

## Layout

- `docker-compose.yml` — SGLang service with NVIDIA GPU support
- `.env` — runtime configuration (model id, token, ports, etc.)
- `.env.example` — template you can copy
- `./models/` — host directory mounted as the HF cache (created on first run)

## Prerequisites

- Docker + Docker Compose v2
- NVIDIA GPU + recent driver + [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

## Quick start

1. Copy and edit the env file:

   ```bash
   cp .env.example .env
   # edit MODEL_ID and (if gated) HF_TOKEN
   ```

2. Launch:

   ```bash
   docker compose up -d
   docker compose logs -f sglang
   ```

3. Test the OpenAI-compatible endpoint:

   ```bash
   curl http://localhost:30000/v1/models
   curl http://localhost:30000/v1/chat/completions \
     -H "Content-Type: application/json" \
     -d '{
       "model": "default",
       "messages": [{"role": "user", "content": "Hello"}]
     }'
   ```

## Switching models

Edit `MODEL_ID` in `.env` and restart:

```bash
docker compose up -d --force-recreate
```

Models are cached in `MODELS_DIR` (default `./models`) so subsequent launches skip the download.

## Common knobs (`.env`)

| Variable | Purpose |
|---|---|
| `MODEL_ID` | HuggingFace repo id (e.g. `Qwen/Qwen2.5-7B-Instruct`) |
| `HF_TOKEN` | Token for gated/private models |
| `MODELS_DIR` | Host path mounted into `/root/.cache/huggingface` |
| `HOST_PORT` | Host port mapped to container port 30000 |
| `TP_SIZE` | Tensor-parallel size (number of GPUs) |
| `DTYPE` | `auto`, `bfloat16`, `float16`, etc. |
| `MEM_FRACTION_STATIC` | Fraction of GPU mem reserved for KV cache |
| `CONTEXT_LENGTH` | Max context length |
| `SGLANG_IMAGE_TAG` | Override `lmsysorg/sglang` tag |
