# Software Layer & Model Deployment

## 1. Service Orchestration
The inference engine (Ollama) is deployed as a background daemon using Homebrew Services.

* **Package Manager:** Homebrew
* **Service Manager:** `launchd` (via `brew services`)
* **Benefit:** Ensures the API endpoint is live immediately upon system boot without user intervention (Headless/Zero-touch deployment).

## 2. Model Selection: Llama 3.3 70B
Selected Meta's Llama 3.3 70B Instruct model to maximize the utility of the hardware's 128GB Unified Memory.

* **Model Architecture:** 70 Billion Parameters
* **Quantization:** 4-bit (Q4_K_M)
* **Memory Footprint:** ~42 GB
* **Context Window:** 128k tokens

## 3. Deployment Commands
```bash
# Install Service
brew install ollama
brew services start ollama

# Model Acquisition
# Pulling the 70B parameter model requires stable throughput for ~40GB binary
ollama pull llama3.3:70b
```

## 4. Performance Benchmarks
**Model:** Llama 3.3 70B (4-bit Quantization)
**Hardware:** Apple M4 Max (128GB Unified Memory)

### Telemetry Data
* **Token Generation Rate (Eval Rate):** 11.01 tokens/s
    * *Significance:* Exceeds average human reading speed (~5-8 t/s), ensuring a fluid user experience for real-time chat.
* **Prompt Processing (Prefill):** 44.72 tokens/s
* **Cold Boot Load Time:** ~11.1 seconds
    * *Note:* Time required to load ~42GB model weights from NVMe SSD to Unified Memory. Subsequent inference requests eliminate this latency (hot-swapping).
 
