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
