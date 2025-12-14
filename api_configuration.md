# API Exposure & Service Persistence

## 1. Network Architecture
By default, the Ollama inference server binds strictly to the loopback interface (`127.0.0.1`), effectively sandboxing it to the local machine. To function as a central inference node for the network, the bind address required modification to accept external traffic.

* **Default Bind:** `127.0.0.1` (Localhost only)
* **Target Bind:** `0.0.0.0` (All available network interfaces)
* **Port:** `11434` (Standard API port)

## 2. Implementation: Service Configuration
To ensure the `OLLAMA_HOST` environment variable persists across system reboots without requiring a user login session, the configuration was injected directly into the `launchd` service definition.

### Configuration File
**Path:** `/opt/homebrew/opt/ollama/homebrew.mxcl.ollama.plist`

### XML Injection
The `EnvironmentVariables` dictionary was inserted into the property list to define the host binding at the service level:

```xml
<key>EnvironmentVariables</key>
<dict>
    <key>OLLAMA_HOST</key>
    <string>0.0.0.0</string>
</dict>
