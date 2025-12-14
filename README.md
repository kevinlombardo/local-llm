# Project: Headless Local LLM Inference Server

## Project Overview
This project documents the engineering and deployment of a high-performance, local AI inference server. The goal was to leverage Apple Silicon's Unified Memory Architecture to run large-scale (70B+) parameter models in a headless, zero-touch environment, exposing an API for network-wide consumption.

---

## 1. Infrastructure & Environment Setup

### 1.1 Hardware Architecture
The system is built on the Mac Studio (2025 Model) to utilize Unified Memory, which allows for loading massive model weights that exceed standard consumer GPU VRAM limits.

* **Host:** Mac Studio
* **SoC:** Apple M4 Max (16-Core CPU / 40-Core GPU)
* **Memory:** 128GB Unified Memory
    * *Significance:* Enables full loading of 70B parameter models (approx. 40-48GB memory footprint) with high quantization precision, maintaining high token/sec throughput.
* **Storage:** 1TB SSD (NVMe)

### 1.2 Operating System Configuration
The host runs **macOS Sequoia**, configured for headless operation (no physical display or peripherals attached).

* **Power Management:**
    * Prevented system sleep when display is off: `sudo pmset -a sleep 0`
    * Enabled automatic restart on power failure to ensure high availability.

### 1.3 Network & Security Layer
To enable secure remote administration without a physical console, a strict SSH environment was configured.

* **Remote Access:** OpenSSH Daemon enabled via macOS Sharing Preferences.
* **Addressing:** DHCP Reservation configured at the network level to enforce a static internal IP (`192.168.x.x`).
* **Authentication Protocol:**
    * **Algorithm:** Ed25519 (Twisted Edwards curve) selected for superior security over RSA.
    * **Implementation:** Password authentication disabled in favor of Public Key Infrastructure (PKI).
    * **Client Side:** Private key managed via PuTTY/Pageant on Windows 11.
    * **Server Hardening:** Enforced strict POSIX permissions (`chmod 700 ~/.ssh` / `600 authorized_keys`) to satisfy OpenSSH security requirements.

---

## 2. Software Layer & Model Deployment

### 2.1 Service Orchestration
The inference engine (Ollama) is deployed as a background daemon using Homebrew Services, leveraging macOS's native `launchd` system.

* **Package Manager:** Homebrew
* **Service Manager:** `launchd` (via `brew services`)
* **Benefit:** Ensures the API endpoint is live immediately upon system boot without user intervention.

### 2.2 Model Selection: Llama 3.3 70B
Selected Meta's Llama 3.3 70B Instruct model to maximize the utility of the hardware's 128GB Unified Memory.

* **Model Architecture:** 70 Billion Parameters
* **Quantization:** 4-bit (Q4_K_M)
* **Memory Footprint:** ~42 GB
* **Context Window:** 128k tokens

### 2.3 Deployment Commands
```bash
# Install Service
brew install ollama
brew services start ollama

# Model Acquisition
# Pulling the 70B parameter model requires stable throughput for ~40GB binary
ollama pull llama3.3:70b
```

---

## 3. Performance Benchmarks
**Model:** Llama 3.3 70B (4-bit Quantization)
**Hardware:** Apple M4 Max (128GB Unified Memory)

### Telemetry Data
* **Token Generation Rate (Eval Rate):** 11.01 tokens/s
    * *Significance:* Exceeds average human reading speed (~5-8 t/s), ensuring a fluid user experience for real-time chat.
* **Prompt Processing (Prefill):** 44.72 tokens/s
* **Cold Boot Load Time:** ~11.1 seconds
    * *Note:* Time required to load ~42GB model weights from NVMe SSD to Unified Memory. Subsequent inference requests eliminate this latency (hot-swapping).
 
---

## 4. Frontend Deployment (Open WebUI)

### 4.1 Container Architecture
To provide a user-friendly interface without the overhead of a GUI-based hypervisor, **Colima** was utilized as the headless container runtime.

* **Runtime:** Colima (QEMU-based) via Docker CLI.
* **Orchestration:** Docker Container running `ghcr.io/open-webui/open-webui:main`.
* **Networking Strategy:**
    * Used `--add-host=host.docker.internal:host-gateway` to bridge the containerized application to the bare-metal Ollama API.
    * Port Mapping: Host `3000` $\rightarrow$ Container `8080`.

### 4.2 Deployment Command
The application was deployed in detached mode with an auto-restart policy to ensure resilience.

```bash
docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
