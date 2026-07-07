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
- If you want your AI stack components to have access to your GPU, ensure you have installed all of the latest drivers for your GPU card
  - For NVIDIA, once I've installed the appropriate and latest driver for my card, I check using nvidia-smi as follows...
```
nvidia-smi --query-gpu=timestamp,name,temperature.gpu,utilization.gpu,utilization.memory,memory.total,memory.free,memory.used --format=csv

# eg...
timestamp, name, temperature.gpu, utilization.gpu [%], utilization.memory [%], memory.total [MiB], memory.free [MiB], memory.used [MiB]
2026/06/09 23:21:09.412, NVIDIA GeForce RTX 3070, 50, 1 %, 1 %, 8192 MiB, 6759 MiB, 1081 MiB
```

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

- MCPO
  - `podman pull ghcr.io/open-webui/mcpo:latest`
  - install uv & npx on system
  - `mkdir -p $HOME/mcpo/config ; mkdir $HOME/mcpo/filesystem ; chmod a+rwx $HOME/mcpo/filesystem`
  - sample config file `$HOME/mcpo/config/config.json` ...
```
{
  "mcpServers": {
    "time-server": {
      "command": "uvx",
      "args": ["mcp-server-time", "--local-timezone=America/Toronto"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/mnt/filesystem"]
    },
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```
  - Start mcpo podman container...
```
podman run --rm -d --name mcpo \
  -v $HOME/mcpo/config:/app/config:Z \
  -v $HOME/mcpo/filesystem:/mnt/filesystem:Z \
  ghcr.io/open-webui/mcpo:latest \
  --port 8001 --config /app/config/config.json
```
  - configure Open WebUI for MCPO
    - **Admin Panel / Settings / Integrations**
      - Add a connection by clicking on the plus sign (+) to the right of **Manage Tool Servers**
      - eg. 'mcpo-time-server' for Name, ID, and Description (or tweak as you wish), then URL = `http://localhost:8001/time-server`, then **Save**

---

## Other components in my AI stack that I still need to document are...
- pgvector
  - just to replace the default chromadb that Open WebUI comes configured with
  - also, I only use pgvector for document indexing, not for Open WebUI configuration...at least not yet
- docling-serve
  - **Content Extraction Engine** in Open WebUI
  - I've run both GPU and CPU only modes for docling-serve container
  - Unfortunately I don't have any side-by-side test results, but I can say from what I've observed anecdotaly, GPU runs much better when loading docs that need docling for processing (like PDFs with images, maps, and charts that are not real text)
- searxng
  - as the name implies, this allows you to search the web more effectively from a chat session within Open WebUI
  - but the default config throws a lot of warnings and errors, so I tweaked mine
  - I also restricted my searxng config to NOT use Google for searches. Main reasoning is that there are a lot of errors and warnings generated as Google detects bots, of course.
  - Also, if I want to search Google, then I'll search Google !
  - Lastly, I purposely restricted searxng to NOT search Reddit
- playwright
  - Works nicely with searxng once web searches have hits to read. Playwright is good at reading web content and getting around bot restrictions
- open-terminal
  - playground to run commands and test things out while in a chat session
