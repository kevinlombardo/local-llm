# Project: High-Performance Headless AI Inference Server

## 1. Project Overview
This project details the engineering, configuration, and deployment of a private, enterprise-grade Generative AI server hosted on Apple Silicon.

The primary objective was to leverage the **Unified Memory Architecture (UMA)** of the **Mac Studio (M4 Max)** to run large-scale Large Language Models (LLMs)—specifically 70B parameter models—at speeds exceeding human reading capabilities, without relying on cloud APIs or consumer-grade GPU VRAM constraints.

The system is architected as a **headless, zero-touch appliance**. It resides on the local network, managed securely via SSH, and exposes an industry-standard API and a containerized web frontend for network-wide consumption.

---

## 2. System Architecture

* **Hardware Layer:** Mac Studio (M4 Max, 128GB Unified Memory)
* **Operating System:** macOS Sequoia (Headless Configuration)
* **Inference Engine:** Ollama (Service Daemon)
* **Container Runtime:** Colima (QEMU-based Docker Runtime)
* **Application Layer:** Open WebUI (Docker Container)

---

## 3. Engineering Documentation
The full implementation details are broken down into four distinct engineering phases.

### [Phase 1: Infrastructure & Security](./phase1_infrastructure.md)
* **Hardware Validation:** Leveraging 128GB Unified Memory for high-precision quantization.
* **Remote Access:** Establishing a secure SSH environment using Ed25519 keys.
* **Network Security:** Configuring static addressing and disabling password authentication.

### [Phase 2: Software Layer & Model Deployment](./phase2_software.md)
* **Service Orchestration:** Managing the inference engine as a background daemon using `launchd`.
* **Model Strategy:** Acquisition and deployment of Meta's **Llama 3.3 70B**.
* **Performance Benchmarking:** Achieving ~11 tokens/second generation speed.

### [Phase 3: API Exposure & Persistence](./phase3_api_setup.md)
* **Network Engineering:** Modifying bind addresses to expose the API to the LAN (`0.0.0.0`).
* **Persistence Strategy:** Injecting environment variables directly into system service definitions.
* **Troubleshooting:** Resolving Apple Silicon specific pathing issues for service configuration.

### [Phase 4: Frontend Deployment](./phase4_frontend_setup.md)
* **Containerization:** Deploying **Open WebUI** using Colima for headless operation.
* **Microservices Networking:** Bridging the Docker container to the bare-metal host API.
* **User Interface:** Providing a ChatGPT-like web experience accessible from any browser on the network.

---

## 4. Technology Stack
* **Host:** Apple M4 Max
* **OS:** macOS Sequoia
* **Shell/CLI:** Zsh, PuTTY (Client)
* **Package Management:** Homebrew
* **AI Backend:** Ollama
* **Containerization:** Docker & Colima
* **Frontend:** Open WebUI
