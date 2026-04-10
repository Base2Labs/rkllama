# rkllama

Ollama-compatible LLM inference server that runs models on the Rockchip NPU (RK3588/RK3576). Acts as a drop-in replacement for Ollama on Orange Pi / Radxa / Turing Pi boards, exposing both the Ollama API and partial OpenAI API compatibility.

## What it does

- Runs `.rkllm` (LLM) and `.rknn` (vision encoder / image gen / TTS / STT) models directly on the NPU
- Exposes `/api/chat`, `/api/generate`, `/api/tags`, `/api/pull`, and more (Ollama-compatible)
- Exposes `/v1/chat/completions`, `/v1/images/generations`, `/v1/audio/speech`, `/v1/audio/transcriptions` (OpenAI-compatible)
- Supports tool/function calling (Qwen, Llama 3.2+)
- Dynamic model loading/unloading with a 30-minute inactivity TTL
- Prompt cache files persist chat session context across model reloads (7-day TTL)
- Pulls models directly from Hugging Face

## Tech stack

- Python 3.9–3.12
- Flask 2.3 + flask-cors
- `rknn-toolkit-lite2` 2.3.2 (bundled as local `.whl` per Python version)
- `rkllm-runtime` v1.2.3, `rknn-runtime` v2.3.2

## Project structure

```
rkllama/
├── src/rkllama/
│   ├── server/server.py    # Flask REST server entry point
│   ├── client/client.py    # CLI client entry point
│   └── lib/                # Bundled .so and .whl native libs
├── models/                 # Place .rkllm / .rknn model files here
├── config/default.ini      # Default configuration
├── converter/              # GGUF → RKLLM conversion tooling
├── Dockerfile
├── docker-compose.yml
└── pyproject.toml
```

## Running locally (target: aarch64 Linux only)

```bash
# Install
python -m pip install .

# Start server
rkllama_server --models ./models

# Start client
rkllama_client
```

## Docker

```bash
# Pull
docker pull ghcr.io/notpunchnox/rkllama:main

# Run
docker compose up --detach --remove-orphans
# Server listens on port 8080
```

## Model naming convention

Models use the `name:size` format (e.g. `qwen2.5:3b`). Each model lives in its own subdirectory under `models/` and requires a `Modelfile` specifying `FROM`, `HUGGINGFACE_PATH`, `SYSTEM`, `TEMPERATURE`, and multimodal fields if applicable.

## Key constraints

- **Hardware only**: The NPU libraries are `aarch64` Linux binaries. The server cannot run on x86 or macOS outside Docker with the correct arch.
- **rknn-toolkit 2.3.2**: All `.rknn` model conversions must use exactly this version to match the bundled runtime.
- **Python 3.9–3.12**: Other versions are not supported (no matching `.whl` bundled).

## Deployment

The production instance is managed by ArgoCD in `base2labs-gitops/`. To deploy a new version, update the image tag in the corresponding Helm values and let ArgoCD sync. The Docker image is published via `.github/workflows/docker-publish.yml`.
