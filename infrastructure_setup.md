# Infrastructure & Environment Setup

## 1. Hardware Architecture
This project leverages the Unified Memory Architecture (UMA) of Apple Silicon to perform high-fidelity local inference without the VRAM bottlenecks typical of consumer GPUs.

* **Host:** Mac Studio (2025 Model)
* **SoC:** Apple M4 Max (16-Core CPU / 40-Core GPU)
* **Memory:** 128GB Unified Memory
    * *Significance:* The 128GB UMA allows for loading quantization-heavy 70B+ parameter models (approx. 40-48GB memory footprint) entirely into high-bandwidth memory, leaving ample headroom for the OS and concurrent API requests.
* **Storage:** 1TB SSD (NVMe speed for fast model loading)

## 2. Operating System Configuration
The host runs **macOS Tahoe**, configured for headless operation (no physical display or peripherals attached).

### Power Management
To ensure 24/7 availability as an API server:
* Prevented sleep when display is off: `sudo pmset -a sleep 0`
* Enabled automatic restart on power failure to ensure resilience.

## 3. Network & Security Layer
The server is accessed via a Windows client using **PuTTY**, adhering to strict security protocols for remote administration.

### Remote Access (SSH)
* **Service:** OpenSSH Daemon enabled via macOS Sharing Preferences.
* **Addressing:** DHCP Reservation configured at the router level to enforce a static internal IP (`192.168.x.x`), ensuring a consistent endpoint for API calls.

### Authentication Protocol
Password authentication was bypassed in favor of public key infrastructure (PKI) to facilitate secure, automated access.

1.  **Key Generation:**
    * Algorithm: **Ed25519** (Twisted Edwards curve).
    * *Rationale:* Selected over RSA-4096 for superior performance and smaller key size with equivalent security.

2.  **Client Configuration (Windows/PuTTY):**
    * Private key converted to `.ppk` format via PuTTYgen.
    * Session profile configured with persistent keep-alive packets to prevent timeouts during long model downloads.

3.  **Server-Side Hardening:**
    * Enforced strict POSIX permissions on the host to satisfy OpenSSH security requirements:
        ```bash
        chmod 700 ~/.ssh
        chmod 600 ~/.ssh/authorized_keys
        ```
