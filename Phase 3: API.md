# Phase 3: API Exposure & Service Persistence

## 1. Network Architecture Challenge
By default, the Ollama inference engine binds strictly to the loopback interface (`127.0.0.1`). This "sandbox" behavior prevents external devices (like the Windows workstation or the Docker container) from accessing the API.

To enable the Mac Studio to function as a central inference server, the bind address required modification.

* **Default Configuration:** `127.0.0.1` (Localhost only)
* **Target Configuration:** `0.0.0.0` (All available network interfaces)
* **Port:** `11434` (Standard Ollama API port)

## 2. Implementation: Persistence via Launchd
To ensure the `OLLAMA_HOST` environment variable persists across system reboots without requiring a user login session, the configuration was injected directly into the macOS `launchd` service definition.

### 2.1 Configuration Injection
The `EnvironmentVariables` dictionary was inserted into the property list (`.plist`) to define the host binding at the system service level.

Note: The file /opt/homebrew/opt/ollama/homebrew.mxcl.ollama.plist was edited directly.

**XML Block Added:**
```xml
<key>EnvironmentVariables</key>
<dict>
    <key>OLLAMA_HOST</key>
    <string>0.0.0.0</string>
</dict>
```
## 3. Validation

```bash
curl http://192.168.1.XXX:11434/api/tags

{"models":[{"name":"llama3.3:70b","model":"llama3.3:70b","modified_at":"2025-12-14T10:54:51.954265177-06:00","size":42520413916,"digest":"a6eb4748fd2990ad2952b2335a95a7f952d1a06119a0aa6a2df6cd052a93a3fa","details":{"parent_model":"","format":"gguf","family":"llama","families":["llama"],"parameter_size":"70.6B","quantization_level":"Q4_K_M"}}]}

```


