# AI Home Lab Glossary

*A reference guide for the M4 Max Local LLM environment.*

## 1. The Core Hierarchy (The "Russian Doll" Architecture)
Understanding the relationship between the raw file and the final chatbot.

| Layer | Term | Analogy | Technical Location |
| :--- | :--- | :--- | :--- |
| **Layer 1** | **Base Model** | The Engine | `~/.ollama/models` (Heavy .gguf file) |
| **Layer 2** | **Custom Model** | The Tuned Engine | Ollama CLI (Manifest file) |
| **Layer 3** | **Workspace Model** | The Dashboard | Open WebUI Database (Browser settings) |

---

## 2. Definitions

### Base Model
**Also known as:** Foundation Model, LLM.
**Definition:** The massive, static binary file containing the neural network's weights. It is "read-only" intelligence. It takes up significant disk space (GBs) and RAM when loaded.
* **Example:** `qwen2.5:32b`, `llama3.3:70b`.
* **Command:** `ollama pull <name>`

### Custom Model (The Manifest)
**Also known as:** Ollama Model.
**Definition:** A permanent entry in the Ollama system that wraps a **Base Model** with a **Modelfile**. It does not duplicate the heavy model weights; it simply points to them and applies new instructions.
* **Why use it:** Allows you to create distinct backend agents (e.g., "Saturn", "Jupiter") that persist even if you delete your User Interface.
* **Command:** `ollama create <name> -f Modelfile`

### Workspace Model (The Persona)
**Also known as:** Open WebUI Model.
**Definition:** The user-facing "character" created inside the Open WebUI browser interface. It wraps the **Custom Model** with UI-specific features like **Web Search**, **Knowledge Bases**, and **User Tools**.
* **Key Distinction:** If you delete a Workspace Model, the underlying Custom Model (Saturn) remains safe in the terminal.

### Modelfile
**Definition:** The blueprint or "script" used to build a Custom Model. It is a simple text file containing instructions.
* **Key Components:**
    * `FROM`: Which Base Model to use (e.g., `qwen2.5:32b`).
    * `SYSTEM`: The personality and rules (e.g., "You are a Fiduciary...").
    * `PARAMETER`: Settings like temperature or context window size.

---

## 3. Hardware & Performance Terms

### Qwen (Tongyi Qianwen)
**Definition:** A model family created by Alibaba Cloud.
* **Strengths:** Exceptional at **STEM** (Math, Coding, Logic).
* **Versions:**
    * **72B:** Massive "PhD-level" intelligence. Slow on consumer hardware (~10 tokens/sec).
    * **32B:** The "Goldilocks" model. High intelligence, extremely fast on Apple Silicon (~40+ tokens/sec).

### Parameter Count (e.g., 32b, 72b)
**Definition:** The number of "neurons" in the model's brain.
* **Rule of Thumb:** Higher count = Higher intelligence = Slower speed = More RAM required.
* **M4 Max Context:** The M4 Max (128GB) can technically run 72B models, but 32B models offer a better balance of speed vs. intelligence for daily tasks.

### Unified Memory
**Definition:** Apple's architecture where the CPU and GPU share the same RAM pool.
* **Relevance:** Unlike PC users who are limited by "VRAM" (video card memory), your Mac can use up to ~90GB of system RAM purely for AI models, allowing you to run massive enterprise-grade models locally.

---

## 4. Monitoring Tools

### `asitop`
**Definition:** A command-line tool for Apple Silicon that visualizes GPU, CPU, and Neural Engine usage.
* **Use Case:** "Hacker style" monitoring. Requires `sudo`.
* **Install:** `pip3 install asitop`

### `macmon`
**Definition:** A lightweight, modern alternative to `asitop` written in Rust.
* **Use Case:** Quick, permission-less monitoring.
* **Install:** `brew install macmon`
