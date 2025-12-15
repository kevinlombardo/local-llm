# Phase 4: Frontend Deployment (Open WebUI)

## 1. Container Architecture
To provide a user-friendly, web-based interface for the LLM without the overhead of a GUI-based hypervisor (like Docker Desktop), **Colima** was selected as the headless container runtime.

* **Runtime:** Colima (QEMU-based)
* **Orchestration:** Docker CLI via Homebrew
* **Benefit:** Allows the container service to run entirely in the background on a headless server, surviving reboots without user login.

## 2. Infrastructure Setup
The container environment was initialized with specific resource limits to ensure the majority of system RAM remains reserved for the inference model (Ollama).

**Installation & Startup:**
```bash
# Install Engine and Client
```
```bash
brew install docker colima
```

# Start Runtime (Allocating minimal resources to the UI layer)
```bash
colima start --cpu 2 --memory 4
```

## 3. Application Deployment
```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --restart always \
  -e OLLAMA_API_BASE_URL=http://host.docker.internal:11434 \
  ghcr.io/open-webui/open-webui:main
```

## 4. Validation

URL: http://192.168.1.XXX:3000
