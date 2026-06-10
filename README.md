# ai-stack

## Components
- Ollama
- Open WebUI
- pgvector
- searxng
- playwright
- mcpo
- docling-serve

## Installing components
- [Ollama](https://github.com/ollama/ollama)
  - Install however best works for you based on the link provided
  - I've installed Ollama on Linux via `curl -fsSL https://ollama.com/install.sh | sh` as well as containers running in podman
    - Both options work fine, but running ollama in a container (docker or podman) can sometimes require a little more set up for GPU access
- [Open WebUI](https://github.com/open-webui/open-webui)
  - You can read the official doc in the link above, or I just do it this way...
```
podman pull ghcr.io/open-webui/open-webui:main

podman run --rm -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main

```

