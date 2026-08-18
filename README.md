# ai-stack

<p>
Want to try building your own AI stack to run in a homelab?<br>
Well, this might help.
</p>

## Overview

![ai-stack-overview](docs/ai-stack-overview.jpeg)

---

## Components

<details>
  <summary>NVIDIA drivers</summary>

<br>

- If you want your AI stack components to have access to your GPU, ensure you have installed all of the latest drivers for your GPU card
  - For NVIDIA, once I've installed the appropriate and latest driver for my card, I usually check using nvidia-smi as follows...
```
nvidia-smi --query-gpu=timestamp,name,temperature.gpu,utilization.gpu,utilization.memory,memory.total,memory.free,memory.used --format=csv

# eg...
timestamp, name, temperature.gpu, utilization.gpu [%], utilization.memory [%], memory.total [MiB], memory.free [MiB], memory.used [MiB]
2026/06/09 23:21:09.412, NVIDIA GeForce RTX 3070, 50, 1 %, 1 %, 8192 MiB, 6759 MiB, 1081 MiB
```

- If you want to continuously monitor your card's usage...
```
nvidia-smi --query-gpu=timestamp,name,temperature.gpu,utilization.gpu,memory.total,memory.free,memory.used --format=csv -l 5
```
  - Sorry, I don't have any AMD Radeon GPUs to test with.

---

</details>

<details>
  <summary>Ollama</summary>

<br>

- [Ollama](https://github.com/ollama/ollama)
  - Ollama is the engine/framework used to run and manage local Large Language Models (LLMs)
  - Think of the LLM as a high-end video game and Ollama as the gaming console; while the model provides the "content" and intelligence, Ollama handles all the complex technical infrastructure required to run it on your hardware.
  - You can chat with a model running within Ollama, but it's not pretty.
  - Install however best works for you based on the link provided
  - I've installed Ollama on Linux via `curl -fsSL https://ollama.com/install.sh | sh` as well as containers running in podman
    - Both options work fine, but running ollama in a container (docker or podman) can sometimes require a little more set up for GPU access
  - If you installed Ollama using the above 'curl...' command, it should set up a systemd service and dedicated user and group (ollama:ollama) for you
  - If you want to open up Ollama to your home network from a **firewalld** perspective, just add port 11434/tcp.
  ```
  firewall-cmd --add-port=11434/tcp
  firewall-cmd --add-port=11434/tcp --permanent
  ```
  - If you need to upate any configuration in the ollama.service unit file, first stop ollama.service...
  ```
  systemctl stop ollama
  ```
  - If you need ollama.service to allow external host access, edit the /etc/systemd/system/ollama.service file and append to the `[Service]` section
  ```
  Environment="OLLAMA_HOST=0.0.0.0"
  ```
  - If you'd like to store the ollama models in a specific directory/path/filesystem, then append to the `[Service]` section...
  ```
  Environment="OLLAMA_MODELS=/ollama/models"
  ```
  - Once you've updated the ollama.service unit file, run...
  ```
  systemctl daemon-reload
  systemctl start ollama
  ```
  - If you want to have a little fun chatting with Ollama before you dive into Open WebUI, you can simply do this...
  ```
  ollama pull qwen3.5:2b-q4_K_M

  ollama run qwen3.5:2b-q4_K_M
  ```
    - type '/bye' to exit ollama chat
  - Here are more environment variables to set in ollama.service that you may want to consider tweaking for your use cases...
  ```
  Environment="OLLAMA_NUM_PARALLEL=2"
  Environment="OLLAMA_MAX_QUEUE=1024"
  Environment="OLLAMA_MAX_LOADED_MODELS=2"
  Environment="OLLAMA_KV_CACHE_TYPE=q4_0"
  ```

<br>

- Now that was fun, or maybe not. But what's next in order to do something meaningful with this?
- **Open WebUI** helps a lot with that!

---

</details>

<details>
  <summary>Podman AI Stack pod</summary>

<br>

- Begin by creating a pod just for your AI stack within podman...
```
podman pod create --name ai-stack -p 3000:8080
```

---

</details>

<details>
  <summary>Open WebUI</summary>

<br>

- [Open WebUI](https://github.com/open-webui/open-webui)
  - The primary interface for interacting with models, managing documents, and using integrated tools.
  - If Ollama is the console providing the power behind the scenes, Open WebUI is the User Interface: it provides a polished, easy-to-use dashboard so you can interact with the AI visually rather than through complex code.
  - This is where you chat with a model, but Open WebUI provides so much more.
  - In order to install Open WebUI, you can read the official doc in the link above, or I just do it this way...
  ```
  podman pull ghcr.io/open-webui/open-webui:main
  
  podman run --rm -d --pod ai-stack --name open-webui \
    -v open-webui:/app/backend/data:Z \
    -e USER_AGENT="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
    ghcr.io/open-webui/open-webui:main  
  ```

  - The `-e USER_AGENT="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \` line is for later, once you have `searxng/yacy` and `playwright` up and running, but I'm getting ahead of myself
  - If your Ollama instance runs on a different system than where you have installed/setup Open WebUI, then you'll need to add connection(s)
    - Under **Admin Panel / Settings / Connections**
    - Ensure that **Ollama API** is enabled
    - Under **Manage Ollama API Connections**, click on the `+` (plus) sign to create a new connection definition
    - **Connection Type** should be `External`
    - **URL** should be set to where your Ollama instance is running (eg. IP 10.0.0.123) then `http://10.0.0.123:11434`
    - Click the recycle button next to URL definition to do a connectivity test.
      - If successful, Open WebUI now has access to all of the models you have pulled into Ollama thus far (eg. `qwen3.5:2b-q4_K_M`). 
      - If not, then Google. Firewall, routes, wrong IPs, are all things to chase down and figure out why you can't connect.
  - Hopefully at this point you're ready to play with **Open WebUI**'s chat sessions
    - Open a **New Chat** session from the left-hand menu (Sidebar)
      - if you don't see the **Sidebar**, click on the **OI** at the top left to expand the **Sidebar** in **Open WebUI**
    - Assuming you've followed this doc to some degree, you probably only have the `qwen3.5:2b-q4_K_M` model available in a chat session.
      - The model should show up automagically in the pull-down list at the top left of the chat session itself
    - Start chatting with your AI stack
    - It'll be limited on what it can do for you, but it's a start. The good stuff comes later as you add on more tools and capabilities.
    - If you start playing with **Worspace/Knowledge** bases, just keep in mind that later in this article the underpinning of document storage and retrieval changes significantly enough that if you have any docs stored in **Open WebUI**, they'll get nuked. So, fair warning.

---

</details>

<details>
  <summary>MCPO</summary>

<br>

- [MCPO](https://github.com/open-webui/mcpo)
  - Provides Model Context Protocol (MCP) capabilities, allowing for integration of external tools like time-servers or file systems.
  - mcpo isn't absolutely necessary to get started with Open WebUI. You can postpone the installation and set up of mcpo for later if you prefer.
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
podman run --rm -d --pod ai-stack --name mcpo \
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

</details>

<details>
  <summary>searxng (no longer using)</summary>

<br>

- [searxng](https://github.com/searxng/searxng)
  - Note: I am no longer using searxng, but I'm leaving the documented below for reference (Who knows - I may come back to this some day)
  - A self-hosted metasearch engine used to provide web search results within the interface.
  - `podman pull docker.io/searxng/searxng:latest`
  - `mkdir -p $HOME/searxng/data ; mkdir -p $HOME/searxng/config`
  - copy the minimalist example [settings.yml](https://github.com/claudeahoule/ai-stack/blob/main/components/searxng/config/settings.yml) file into `$HOME/searxng/config/settings.yml`
    - note the following search engines have been disabled in this configuration file:
      - google
      - reddit
  - Start searxng podman container...
```
podman run --rm -d --pod ai-stack --name searxng \
  -e GRANIAN_PORT=8888 \
  -v "$HOME/searxng/config/:/etc/searxng:Z" \
  -v "$HOME/searxng/data/:/var/cache/searxng:Z" \
  docker.io/searxng/searxng:latest
```

  - In **Open WebUI**, under **Admin Panel / Settings / Web Search**...
    - ensure that **Web Search** is **enabled**
    - **Web Search Engine** is set to **searxng**
    - Under **Searxng Query URL**, set to `http://localhost:8888/search?q=<query>`
    - Leave all other settings at defaults for now
    - Click **Save** at the bottom of the page

  - By itself under Open WebUI, searxng is...okay, but not great...yet. The power of this search engine comes into fruition when you couple it with `playwright`

---

</details>

<details>
  <summary>yacy</summary>

<br>

- [yacy](https://github.com/yacy)
  - A self-hosted metasearch engine used to provide web search results within the interface.
  - `podman pull docker.io/yacy/yacy_search_server:latest`
  - Start yacy container...
```
  podman run --rm -d --pod ai-stack --name yacy \
    -v yacy_data:/opt/yacy_search_server/DATA \
    yacy/yacy_search_server:latest
```

  - In **Open WebUI**, under **Admin Panel / Settings / Web Search**...
    - ensure that **Web Search** is **enabled**
    - **Web Search Engine** is set to **yacy**
    - Under **Yacy URL**, set to `http://localhost:8090`
    - Set **Yacy Username** to `admin` and **Yacy Password** to `yacy`
    - Set **Search Result Count** to `5` and **Concurrent Requests** to `1`
    - Click **Save** at the bottom of the page

---

</details>

<details>
  <summary>playwright</summary>

<br>

- [playwright](https://github.com/microsoft/playwright)
  - Enhances the power of search engines (like searxng/yacy) by providing more meaningful results through automated browser interactions.
  - Check **Open WebUI**'s requirement for the version of playwright under `https://raw.githubusercontent.com/open-webui/open-webui/refs/heads/main/backend/requirements.txt`
    - eg.
    ```
    curl -sk https://raw.githubusercontent.com/open-webui/open-webui/refs/heads/main/backend/requirements.txt | \
    awk '/^playwright/{v=$1;n=split(v,a,"=");print a[n]}'
    ```
  - `podman pull mcr.microsoft.com/playwright:v1.60.0`
  - Start playwright podman container...
```
podman run --rm -d --pod ai-stack --name playwright \
  --security-opt label=disable \
  --ipc=host \
  --user pwuser \
  --workdir /home/pwuser \
  mcr.microsoft.com/playwright:v1.60.0 npx -y playwright@1.60.0 run-server --port 8002 --host 0.0.0.0
```

  - In **Open WebUI**, under **Admin Panel / Settings / Web Search**...
    - Under **Loader** section, ensure that **Web Loader Engine** is set to `playwright`
    - Set **Playwright WebSocket URL** to `ws://localhost:8002`
    - Leave all other settings at defaults for now
    - Click **Save** at the bottom of the page

  - Now when you do web searches from within a chat session in Open WebUI, you'll get much more meaningful results than with just yacy alone.

---

</details>

<details>
  <summary>pgvector (and setting embedding model)</summary>

<br>

- [pgvector](https://github.com/pgvector/pgvector)
  - A specialized database extension used to index and store documents for Retrieval-Augmented Generation (RAG).
  - `podman pull pgvector/pgvector:pg16`
  - `mkdir $HOME/postgres_data`
  - Start pgvector podman container...
```
podman run --rm -d --pod ai-stack --name pgvector \
  -v $HOME/postgres_data:/var/lib/postgresql/data:Z \
  -e POSTGRES_DB=ragdb \
  -e POSTGRES_USER=openwebui \
  -e POSTGRES_PASSWORD=XXXXXXXX \
  pgvector/pgvector:pg16
```

  - Your `podman run... open-webui` will need to be tweaked a little...
```
podman run --rm -d --pod ai-stack --name open-webui \
  -v open-webui:/app/backend/data:Z \
  -e VECTOR_DB="pgvector" \
  -e PGVECTOR_DB_URL="postgresql://openwebui:XXXXXXXX@localhost:5432/ragdb" \
  -e DATABASE_URL_PGVECTOR="postgresql://openwebui:XXXXXXXX@localhost:5432/ragdb" \
  -e USER_AGENT="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  ghcr.io/open-webui/open-webui:main
```

  - From what I understand of how **Open WebUI** is using pgvector with the aforementioned options in starting open-webui podman container, pgvector is now the index for all documents loaded from this point on within **Open WebUI**.
    - So if you had already loaded some documents into **Open WebUI**, then you'll likely need to reload all of your documents again.
      - I did warn you earlier about it under the **Open WebUI** installation section. But yes, the warning did come at the end of the spell. Sorry, but not sorry.
    - This is a good time to change your document embedding option with **Open WebUI**...
      - First, go to your Ollama instance, and run `ollama pull bge-m3:latest`
      - Go back to **Open WebUI**, under **Admin Panel / Settings / Documents**
      - Set the **Embedding Model Engine** to Ollama
      - Configur the **URL** to point to your Ollama instance (eg. `http://10.0.0.123:11434`)
      - Set the **Embedding Model** to `bge-m3:latest`
      - Set the **Embedding Batch Size** to 16 for an GPU with 8GB of VRAM loaded with the aforementioned embedding model
      - Click **Save** at the bottom of the page
      - You may have to do the **Reset Vector Storage/Knowledge** and **Reindex Knowledge Base Vectors** at the bottom of this page. Yes, it's under the **Danger Zone** section. And it's dangerous if you want to keep anything. But at this point, following this doc thus far will put you into a scenario where it really doesn't matter. We're nuking existing docs and reloading them into **Open WebUI** at this point. Just pull off the bandaid. Rip it off. Go ahead.

---

</details>

<details>
  <summary>docling-serve</summary>

<br>

- [docling-serve](https://github.com/docling-project/docling-serve)
  - Used as a content extraction engine to process and parse various document types.
  - Here you have a choice of running a CPU only or GPU enabled version of docling-serve
    - If you intend to run docling-serve on system with an NVIDIA GPU, then...
      - `podman pull quay.io/docling-project/docling-serve-cu128:latest`
    - If you intend to run docling-serve on system with no GPU and rely solely on CPU, then...
      - `podman pull quay.io/docling-project/docling-serve-cpu:latest`
  - In my homelab, I chose to run docling-serve on a GPU enabled system, which is not the same as where my **Open WebUI** instance runs. Hence, this is how I start up my docling-serve-cu124 container...
```
podman run --rm -d --name docling-serve \
  -p 5001:5001 \
  --device nvidia.com/gpu=all \
  -e DOCLING_DEVICE=cuda \
  -e DOCLING_NUM_THREADS=8 \
  -e UVICORN_WORKERS=1 \
  -e DOCLING_SERVE_MAX_SYNC_WAIT=900 \
  quay.io/docling-project/docling-serve-cu128
```

  - Of course, I had to open fw rules for port 5001 on that system...
    - `firewall-cmd --add-port=5001/tcp ; firewall-cmd --add-port=5001/tcp --permanent`

  - In **Open WebUI**, under **Admin Panel / Settings / Documents**
  - Set **Content Extraction Engine** to `Docling`
  - Set the **URL** to the IP address where docling-serve container is running (eg. 10.0.0.123) then `http://10.0.0.123:5001`
  - Under **Parameters**, at the time of writing this, I have the following defined...
```
{
  "do_ocr": true,
  "force_ocr": false,
  "ocr_engine": "tesseract",
  "ocr_lang": [
    "eng"
  ],
  "pdf_backend": "dlparse_v4",
  "table_mode": "accurate",
  "do_table_structure": true,
  "table_cell_matching": true,
  "document_timeout": 900,
  "abort_on_error": false
}
```

  - Set **Chunk Size** to 1500
  - Set **Chunk Overlap** to 200
  - Both **Chunk Size** and **Chunk Overlap** are subject to your use cases, so you may want to test and tweak to what works best for you
  - DON'T FORGET TO CLICK **Save** at the bottom of the page.

---

</details>

<details>
  <summary>Open Terminal</summary>

<br>

- [open-terminal](https://github.com/open-webui/open-terminal)
  - Provides a terminal interface within the Open WebUI environment for executing commands.
  - [Open WebUI documenation for Open Terminal](https://docs.openwebui.com/features/open-terminal/setup/installation/)
  - `podman pull ghcr.io/open-webui/open-terminal`
  - chose a custom secret key to use with **Open Terminal**
    - **Open WebUI** will need this secret key in order to be able to communicate with **Open Terminal**
  - Start open-terminal in a podman container...
```
podman run --rm -d --pod ai-stack --name open-terminal \
  -v open-terminal:/home/user \
  -e OPEN_TERMINAL_API_KEY=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX \
  ghcr.io/open-webui/open-terminal
```

  - In **Open WebUI**, under **Admin Panel / Settings / Integrations**
  - Set **Open-Terminal (localhost)** to enabled
  - Click the configuration gear icon next to it
  - Set the **URL** to `http://localhost:8000`
  - Set **Auth** to `Bearer` and paste the secret-key for **Open Terminal**
  - Click **Save**

- Using **Open Terminal**
  - You can click on the small cloud icon near the bottom right of a chat session that has a model that supports tools
  - Or, you can enable it under **Workspace/Model** for a given custom model you create. It's near the bottom of the page, under **Terminal**. It'll likely be set to `None`, but there's a pull-down menu buttom to the right of that. Choose `Open Terminal (localhost)`
  - Don't forget to click **Save & Update**
---

</details>

<details>
  <summary>OpenClaw</summary>

<br>

- [OpenClaw](https://github.com/openclaw/openclaw)
  - `podman pull ghcr.io/openclaw/openclaw:latest`
  - [see my messy notes for openclaw thus far](components/openclaw/blah.txt)

---

</details

---
---

## Miscellaneous schtuff

<details>
  <summary>Custom Model configurations</summary>

<br>

| Custom Model Name | Base Model | Description | Function Calling | Temperature | max_tokens | top_k | top_p | repeat_penalty | num_ctx | num_thread | num_gpu | keep_alive |
|-------------------|------------|-------------|------------------|-------------|------------|-------|-------|----------------|---------|------------|---------|------------|
| VAI | gemma4:12b-it-q4_K_M | Virutal Assistant Intelligence | Native | 0.65 | 8192 | 40 | 0.9 | 1.1 | 32768 | Default | Default | 20m |
| Code Assistant | qwen2.5-coder:14b-instruct-q4_K_M | Code Assistant | Default | 0.1 | 8192 | Default | Default | Default | 32768 | Default | Default | Default |
| Novelist | mistral-small:24b-instruct-2501-q4_K_M | Model to help write stories | Default | 0.75 | 16384 | Default | 0.9 | 1.15 | 16384 | 10 | 10 | Default |

</details>

<details>
  <summary>My hardware</summary>

<br>

### System 1 (Primary Ollama instance)
- i9-12900 CPU
- 128 GB DDR5 RAM
- NVIDIA RTX 3070 (8GB VRAM)
- Ollama instance #1 (Primary)
- docling-serve (GPU enabled) - podman container
- Fedora Workstation

### System 2 (Secondary Ollama instance)
- i9-12900K CPU
- 32 GB DDR5 RAM
- NVIDIA RTX 3080 TI (12GB VRAM)
- Ollama instance #2 (Secondary)
- MS Windows 11 Home

### System 3 (Open WebUI instance)
- Virtual Machine (KVM/libvirt)
- 8 virtual CPUs
- 24 GB of virtual RAM
- 100 GB of disk space for Open WebUI and documents
  - openwebui userid is under /home/openwebui
- Open WebUI container instance (open-webui) running in podman
- Additional podman containers running in same pod as open-webui...
  - pgvector
  - open-terminal
  - yacy
  - playwright
  - mcpo

---

</details>

<details>
  <summary>LLMs that I have tested and had fun with</summary>

<br>

### Models I'm currently using (as of 2026-07-07)
- gemma4:12b-it-q4_K_M
- qwen2.5-coder:7b-base-q4_K_M
- bge-m3:latest
- nomic-embed-text:latest
- deepseek-r1:14b-qwen-distill-q4_K_M
- mistral-small:24b-instruct-2501-q4_K_M
- qwen3.5:9b-q4_K_M

### Models I've tried and replaced/consolidated with the above at some point
- command-r:35b-08-2024-q4_K_M
- qwen2.5:7b-instruct-q4_K_M
- qwen2.5:14b-instruct-q4_K_M
- qwen3.6:27b-q4_K_M
- ministral-3:14b-instruct-2512-q4_K_M
- llama3.1:8b-instruct-q4_K_M
- granite4.1:8b-q4_K_M
- llama3.2:3b-instruct-q4_K_M
- and many, many others, some large, some small

---

</details>

<details>
  <summary>Prompts</summary>

  <blockquote>

  <details>
    <summary>VAI (Virtual Assitant Inteligence)</summary>

    ```
    ### Identity:
    1. **Personality**: Model after JARVIS: calm, professional, witty, loyal.
    2. **Components**: Refer to VAI_System_Info.md in the Knowledge Base (KB) 'VAI' for factual information about hardware, services, and tools.
    
    ### Operational Guidelines:
    1. **Identity**: Always identify as VAI.
    2. **Tone**: Maintain a refined, helpful demeanor.
    3. **Efficiency**: Provide concise, high-utility answers; expand with detailed explanations if asked.
    4. **Tool Use**: Prioritize structured thinking and technical solutions for homelab management, coding, and automation tasks.
    5. **Information Quality**: Prefer authoritative sources; use community forums sparingly and clearly label them.
    
    ### Enhanced Guidelines for Informative Responses:
    1. **Contextual Understanding**: Ensure responses are well-structured and comprehensive.
    2. **Technical Accuracy**: Verify technical accuracy, especially for homelab management tasks.
    3. **Clarity**: Use clear language; avoid jargon unless necessary.
    4. **Step-by-Step Guidance**: Provide actionable steps for multi-step tasks.
    5. **Examples and Analogies**: Use examples to illustrate complex concepts.
    
    ### Web Search Guidelines:
    1. **Default Tool**: Use 'search_web' for broad queries.
    2. **Single Source Queries**: Use 'fetch_url' for specific webpage information.
    ```

  </details>

  <details>
    <summary>Expert Linux SysAdm</summary>

    ```
    You are an expert Linux Systems Administrator and Bash scripting specialist. 
    Your goal is to provide efficient, secure, and POSIX-compliant scripts when possible.
    
    ### Guidelines:
    1.  **Safety First**: Always include 'set -e', 'set -u', and 'set -o pipefail' in scripts to ensure they fail gracefully.
    2.  **Modularity**: Write clean, reusable functions. Use local variables ('local var=value') to avoid global scope pollution.
    3.  **Portability**: Prefer standard tools (sed, awk, grep) available in most distributions (Debian, RHEL, Arch). If a solution requires a specific package (e.g., 'jq'), mention it.
    4.  **Documentation**: Provide concise comments for complex regex or logic blocks. Explain the purpose of each flag used in commands.
    5.  **Security**: Sanitize user inputs and use double-quotes around variables ("$VAR") to prevent word splitting and globbing issues.
    6.  **Formatting**: Return code in clear markdown blocks with the language specified (e.g., ```bash).
    
    If the user's request is ambiguous, ask for clarification regarding the specific Linux distribution or shell (bash, zsh, sh) being used.
    ```

  </details>

  <details>
    <summary>Code Assistant</summary>

    ```
    As an AI code assistant, my primary function is to provide assistance with coding tasks, explain technical concepts, and help debug issues. I'm designed to understand and generate code in various programming languages, including Python, JavaScript, TypeScript, C++, C#, Java, Rust, Go, Swift, PHP, and others.
    
    Here are some key aspects of my functionality:
    
    1. **Code Generation**: I can generate code snippets, functions, or even entire scripts based on your description of what you need. Please provide clear instructions about the desired functionality.
    
    2. **Explanations**: I can explain complex coding concepts, data structures, algorithms, and programming language features in a simple and easy-to-understand manner.
    
    3. **Debugging Assistance**: If you're having trouble with your code, I can help identify issues, suggest debugging strategies, and provide solutions to resolve them.
    
    4. **Code Refactoring and Optimization**: I can review your code and suggest improvements to enhance readability, performance, or best practices adherence.
    
    5. **Library and API Assistance**: I can help you understand how to use specific libraries, frameworks, APIs, or SDKs in various programming languages.
    
    6. **Multilingual Support**: While my strength lies in understanding and generating code in several popular programming languages, I may not be able to provide the same level of assistance for all languages.
    
    Please provide me with the details of your coding task, and let's get started! If you have any specific questions or need clarifications on certain concepts, feel free to ask.
    
    Prefer authoritative and official sources for search-based answers. Use information from community forums like Reddit only as a last resort or for anecdotal context, and clearly label it as such.
    ```

  </details>

  <details>
    <summary>Narrative Architect - Fantasy</summary>

    ```
    You are a Narrative Architect and Logic Engine. Your goal is to provide structural analysis, world-building consistency, and plot-hole detection for fiction writers.
    
    ### CORE OPERATING GUIDELINES
    - REASONING: Use your internal thinking process to stress-test ideas before answering. Look for logical flaws, clichés, and missed opportunities.
    - CONTINUITY: Prioritize internal consistency above all else. If a magic system or character action contradicts a previous rule, flag it immediately.
    - STRUCTURE: When asked to outline, focus on beats, stakes, and narrative tension rather than just plot events.
    - FEEDBACK: Be analytical and objective. Do not just praise ideas; identify where they might fail or how they can be strengthened.
    
    ### OUTPUT FORMAT
    - Start every response by identifying the "Core Problem" or "Primary Objective" of the user's query.
    - Use structured Markdown (headers, bullet points, tables) for readability.
    - When suggesting changes, explain the "Why" behind the suggestion based on narrative theory.
    ```

  </details>

  <details>
    <summary>Novelist</summary>

    ```
    Act as a literary novelist. Write exclusively in Third Person Limited POV, staying tethered to [Character Name].
    
    Eliminate all filter words (e.g., "he saw," "she felt"). Instead, describe the world through the character's unique biases and sensory experiences. If the character is cold, do not say "he felt cold"; describe his shivering limbs and the bite of the wind.
    
    Use a "Deep POV" style where the narration adopts the character's voice without using first-person pronouns. Prioritize subtext and internal monologue over external summary. Vary sentence rhythm to mirror the character's heart rate and tension.
    ```

  </details>

  <details>
    <summary>RPG Dungeon Master</summary>

    ```
    You are an expert tabletop RPG Dungeon Master and world-builder specializing in J.R.R. Tolkien’s Middle-earth. Your job is to help me design, prep, and run adventures for my players. You are intimately familiar with the lore, themes, and tone of Tolkien's work, as well as systems like "The One Ring RPG" and "Adventures in Middle-earth".
    
    When answering, adhere strictly to the following rules:
    
    1. THEMATIC INTEGRITY: Ensure all encounters, NPCs, and plots emphasize Tolkien's core themes—the corrupting nature of shadow, the beauty of the natural world, the value of fellowship, and "hope against hope." Avoid high-fantasy tropes like flashy, common magic, or overly cynical "grimdark" anti-heroes. Magic must feel rare, subtle, and wondrous.
    
    2. MECHANICAL BALANCING: When asked for mechanics, stats, or encounter design, provide balanced challenges. Structure your mechanical suggestions to include explicit "Combat Capabilities", "Exploration Hazards", and "Social Interaction Triggers". 
    
    3. SCANNABILITY FOR THE DM: Organize all adventure outlines, locations, and NPCs using clear headers, bullet points, and bold text. Break down locations into: Atmosphere, Notable NPCs, and Potential Complications. 
    
    4. DYNAMIC SANDBOXING: When I ask for adventure hooks or plot points, always provide 3 distinct options: one focusing on Exploration/Wilderness, one on Social/Intrigue, and one on Combat/Shadow-creatures.
    
    Acknowledge your role, ask me what Era/Year the adventure takes place in, and ask what specific region of Middle-earth we are building today.
    ```

  </details>

  <details>
    <summary>Heavy Document (RAG) Work</summary>

    ```
    # Role
    You are an expert Research Analyst specializing in complex document synthesis, technical analysis, and information extraction from large-scale datasets. Your goal is to provide precise, high-utility answers based strictly on the provided context.
    
    # Operational Constraints (RAG Protocol)
    1. **Strict Grounding**: Base your answers ONLY on the information provided in the retrieved snippets. If the information is not present in the context, state: "I do not have sufficient information in the provided documents to answer this." Do not use external knowledge or make assumptions.
    2. **Citation & Traceability**: When possible, refer to specific sections, document names, or key terms from the source text to support your claims.
    3. **No Hallucinations**: Do not invent facts, dates, figures, or technical specifications that are not explicitly stated in the retrieved data.
    4. **Handling Conflict**: If two pieces of information in the context contradict each other, point out the discrepancy rather than choosing one arbitrarily.
    
    # Task Instructions for Heavy Documents
    - **Comprehensive Synthesis**: When asked about a broad topic, synthesize information from multiple snippets into a cohesive summary.
    - **Structure & Formatting**: Use Markdown formatting (bolding, headers, and bullet points) to make complex data easy to digest. 
    - **Technical Accuracy**: Maintain the technical integrity of the original text. Do not over-simplify complex engineering or legal terminology unless specifically asked to do so.
    - **Conciseness vs. Detail**: Provide a detailed response for complex queries, but remain concise and avoid repetitive "filler" language.
    
    # Response Style
    - Professional, objective, and analytical.
    - Use tables for comparing data points if it improves clarity.
    - If the user's request is ambiguous, ask clarifying questions before providing a full analysis.
    
    # Execution Workflow
    1. Analyze the retrieved context to identify all relevant facts.
    2. Organize these facts into logical categories.
    3. Draft the response ensuring every claim is backed by the provided text.
    4. Review the output against the "Strict Grounding" rule before final delivery.
    ```

  </details>

  <details>
    <summary>ChefRemy</summary>

    ```
    You are chef Remy from the Pixar film Ratatouille. You provide detailed, accurate recipes based on available ingredients.
    You can offer substitutions, suggest cooking techniques, and adhere to dietary restrictions (e.g., gluten-free, vegan).
    ```

  </details>

  <details>
    <summary>Argyle</summary>

    ```
    You are Argyle from Stranger Things Netflix series.
    
    You try to assist with RPG adventures, but have difficulties with ideas and staying coherent
    ```

  </details>

  </blockquote>

</details>

<details>
  <summary>Tools and Functions</summary>

  <blockquote>

  <details>
    <summary>consult_expert</summary>

  - Consult a specialized expert Workspace/Model in Open WebUI that has access to specialized Workspace/Knowlege base material
    - Download the tool-consult_expert.json file:
      - `curl -o tool-consult_expert.json https://raw.githubusercontent.com/claudeahoule/ai-stack/refs/heads/main/components/open-webui/workspace/tools/tool-consult_expert.json`
    - In 'Open WebUI', under 'Workspace / Tools'
    - Click on **Import** (top-right of page, near `+ New Tool` button)
    - Use the `function-token_count_display.json` file you just save above
    - Click on Save
    - Once saved, click on 'Tools' again at the top
    - Click on the gears icon next to 'consult_expert'
    - Paste the 'Open WebUI' API KEY you have in your account into 'Api Key' custom valve
    - Set 'Base Url' to 'http://localhost:8080'
    - Click Save

---

  - Using `consult_expert` tool in custom models
    - For a custom Virtual Assistant models, such as VAI (Virtual Assistant Intelligence), which is based on J.A.R.V.I.S., add `consult_expert` tool and save the custom model
    - Now your Virual Assistant model doesn't need to have every Knowledge base document attached to it. Your Virtual Assistant can simply consult with another custom model that has specific Knowledge base attached to it. The other custom models can also have completely different base models (perhaps some models are better are coding than your Virtual Assistant, or is specialized in medical subjects, or better for translating, etc...).
    - The idea here is that you build as many custom expert models, using appropriate base models for their use cases, and only attach appropriate Knowledge base content needed for that model.
    - Using the following enhancement to your Virtual Assistant's main prompt...
    ```
    You are the primary interface. You have access to a suite of expert models via the consult_expert tool
    ```
    - Your Virtual Assistant will consult with other custom models that are appropriate for the use case
    - You may need to define the list of other custom models and a brief description of what they are used for

  </details>

  <details>
    <summary>WIP</summary>

  - Token Counter Display: Shows per-message and cumulative session token counts as a status notification after each response.
  - save_to_knowledge - Saves the current message or the whole chat into an Open WebUI Knowledge Base for later RAG use.
  - Local Weather - Fetches current weather and forecast.
  - openwebui_ssh - Access Open WebUI node via ssh in order to run a curated suite of commands for health status and troubleshooting.
  - ollama_ssh - Access Ollama node via ssh in order to run a curated suite of commands for health status and troubleshooting.
  - nagios - fetch and summarize Nagios service alerts.

  </details>

  </blockquote>
      
</details>

<details>
  <summary>Don't have Linux, well...</summary>

  - If you're a Windows user, you could...
    - Install Ollama directly onto your PC/laptop. Hopefully you have a GPU there. It'll help.
      - Just make sure to open up port 11434 on your Windows PC/Laptop firewall
    - Install Oracle Virtualbox on your PC as well, and then download and build a Linux VM
      - I don't know how well the installation docs I provided would work on a Ubuntu image, but I know they'll work on Fedora 43 and 44 Workstation or Server
    - Might work under WSL2 on Windows
    - Once you have Open WebUI running in a VM, just configure **Admin Panel / Settings / Connections** to point to your PC/Laptop internal virtual IP (you may need to open up that firewall as well for port 11434...not sure).

</details>

<details>
  <summary>FAQ</summary>

<blockquote>
WIP - easier to do once someone starts asking me questions about it

</blockquote>

</details>

<details>
  <summary>Troubleshooting</summary>

<blockquote>

### Monitoring NVIDIA GPU usage
```
nvidia-smi --query-gpu=timestamp,name,temperature.gpu,utilization.gpu,memory.total,memory.free,memory.used --format=csv

# OR

nvidia-smi --query-gpu=timestamp,name,temperature.gpu,utilization.gpu,memory.total,memory.free,memory.used --format=csv -l 5
```

### Monitor podman containers on Open WebUI
```
podman stats -a -i 5
```

### Check for Open WebUI container errors in logs
```
podman logs open-webui | tail -5000 | grep -i error
```

### Monitor Ollama model usage
```
ollama ps
```

### Monitor remote Ollama model usage
```
OLLAMA_HOST=10.0.0.234:11434 ollama ps
```

### Check for Ollama issues
```
journalctl -u ollama --since "15 minutes ago" --no-pager
```

</blockquote>

</details>
