# Project: Headless Local LLM Inference Server

## 1. Project Overview
This project documents the engineering and deployment of a high-performance, local AI inference server. The goal was to leverage Apple Silicon's Unified Memory Architecture to run large-scale (70B+) parameter models in a headless, zero-touch environment, exposing an API for network-wide consumption.

## 2. Infrastructure & Environment Setup

### 2.1 Hardware Architecture
The system is built on the Mac Studio (2025 Model) to utilize Unified Memory, which allows for loading massive model weights that exceed standard consumer GPU VRAM limits.
* **Host:** Mac Studio
* **SoC:** Apple M4 Max (16-Core CPU / 40-Core GPU)
* **Memory:** 128GB Unified Memory
* **Storage:** 1TB SSD (NVMe)

### 2.2 Network & Security Layer
To enable secure remote administration without a physical console, a strict SSH environment was configured.
* **Remote Access:** OpenSSH Daemon enabled via macOS Sharing Preferences.
* **Authentication Protocol:**
    * **Algorithm:** Ed25519 (Twisted Edwards curve).
    * **Implementation:** Password authentication disabled in favor of Public Key Infrastructure (PKI).
    * **Server Hardening:** Enforced strict POSIX permissions (`chmod 700 ~/.ssh` / `600 authorized_keys`).

## 3. Software Layer & Model Deployment

### 3.1 Service Orchestration
The inference engine (Ollama) is deployed as a background daemon using Homebrew Services.
* **Service Manager:** `launchd` (via `brew services`).
* **Benefit:** Ensures the API endpoint is live immediately upon system boot without user intervention.

### 3.2 Model Selection: Llama 3.3 70B
Selected Meta's Llama 3.3 70B Instruct model to maximize the utility of the hardware's 128GB Unified Memory.
* **Model Architecture:** 70 Billion Parameters
* **Quantization:** 4-bit (Q4_K_M)
* **Memory Footprint:** ~42 GB

### 3.3 Performance Benchmarks
* **Token Generation Rate:** ~11.01 tokens/s (Exceeds average human reading speed).
* **Cold Boot Load Time:** ~11.1 seconds (Time to load 42GB from SSD to RAM).

## 4. API Exposure & Service Persistence

### 4.1 Network Architecture
By default, the inference server binds to `127.0.0.1`. The bind address was modified to `0.0.0.0` to accept external LAN traffic.

### 4.2 Implementation: Persistence via Launchd
To ensure the `OLLAMA_HOST` environment variable persists across reboots, the configuration was injected directly into the service definition.
* **File:** `/opt/homebrew/opt/ollama/homebrew.mxcl.ollama.plist`
* **Configuration:**
    ```xml
    <key>EnvironmentVariables</key>
    <dict>
        <key>OLLAMA_HOST</key>
        <string>0.0.0.0</string>
    </dict>
    ```

### 4.3 Troubleshooting & Resolution
* **Issue:** Service definition file missing from standard user library.
* **Root Cause:** Homebrew on Apple Silicon stores master definitions in the Cellar.
* **Resolution:** Edited the source file in `/opt/homebrew/opt/ollama/` directly.

## 5. Frontend Deployment (Open WebUI)

### 5.1 Container Architecture
To provide a user-friendly interface without the overhead of a GUI-based hypervisor, **Colima** was utilized as the headless container runtime.
* **Runtime:** Colima (QEMU-based) via Docker CLI.
* **Orchestration:** Docker Container running `ghcr.io/open-webui/open-webui:main`.

### 5.2 Deployment Command
The application was deployed with a bridge to the bare-metal host.
```bash
docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
