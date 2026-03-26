# Remote LLM Server — Dockerized Ollama API
[![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?logo=docker&logoColor=white)](https://docker.com)
[![Ollama](https://img.shields.io/badge/ollama-000000?logo=ollama)](https://ollama.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
> A lightweight, GPU-accelerated Ollama server in Docker, designed to serve local LLMs to multiple clients on your network. Perfect for powering AI apps like 
> [Anagnosi](https://github.com/Pavloffm/Anagnosi) without installing models on every device.
## Prerequisites

| Requirement                           | Why               | Verify             |
|---------------------------------------|-------------------|--------------------|
| **Docker + Compose**                  | Container runtime | `docker --version` |
| **NVIDIA GPU + drivers** *(optional)* | GPU acceleration  | `nvidia-smi`       |
| **~10–30 GB disk**                    | Model storage     | `df -h`            |

> **No GPU?** Ollama runs on CPU — slower, but fully functional for testing and small models `≤4B`.

>  **GPU Setup**: If `nvidia-smi` works on your host but not in Docker, install the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html):
> ```bash
> # Ubuntu/Debian example
> sudo apt install nvidia-container-toolkit && sudo systemctl restart docker
> ```
## Quick Start
### 1. Clone & Configure
```bash
git clone https://github.com/Pavloffm/remote-llm-server.git
cd remote-llm-server
```
### 2. Start the Server
Build and start the Ollama container
```bash
docker compose up --build
```
View logs to confirm startup
```bash
docker compose logs -f ollama
```
### 3. Pull Your Models
Enter the container
```bash
docker exec -it ollama bash
```
Pull models
```bash
ollama pull qwen3.5:9b-q4_K_M  
ollama pull qwen3.5:4b
```

### Test run of models
```bash
curl http://localhost:11434/api/tags
```

Verify installed models and exit
```bash
ollama list
exit
```
## API Access
### Testing on server machine
List available models
```bash
curl http://localhost:11434/api/tags | jq
```
```bash
curl http://localhost:11434/api/generate -H "Content-Type: application/json" -d '{
  "model": "qwen3.5:4b",
  "prompt": "Explain quantum computing in one sentence.",
  "stream": false
}' | jq -r '.response'
```
### Remote Access (from other devices on your network)
#### 1: Find Your Server's IP
Linux/macOS
```bash
ip addr show | grep inet
```
Windows (PowerShell)
```powershell
ipconfig | findstr "IPv4"
```
Example output: `192.168.1.100`
#### 2: Configure Firewall (Windows Only)
Run PowerShell as Administrator
```powershell
New-NetFirewallRule -DisplayName "Ollama Docker" -Direction Inbound -Protocol TCP -LocalPort 11434 -Action Allow
```
#### 3: Test from Client Device
From your laptop, phone, or Raspberry Pi (replace `192.168.1.100` with your server's IP):
```bash
curl http://192.168.1.100:11434/api/tags
```
### Management Commands
View server logs
```bash
docker compose logs -f ollama
```
Restart the server
```bash
docker compose restart ollama
```
Stop the server
```bash
docker compose down
```
Update Ollama to latest version
```bash
docker compose pull ollama && docker compose up -d --force-recreate
```
Free disk space (remove unused models)
```bash
docker exec -it ollama ollama rm <model-name>
```
### Quick Troubleshooting
| Issue                         | Solution                                           |
|-------------------------------|----------------------------------------------------|
| `Connection refused` remotely | Check firewall + ensure Ollama binds to `0.0.0.0`  |
| Slow responses                | Verify GPU: `docker exec ollama nvidia-smi`        |
| `Out of memory`               | Use `-q4_K_M` quantized models or reduce `num_ctx` |

### Security Considerations
| Risk                             | Mitigation                                                                         |
|----------------------------------|------------------------------------------------------------------------------------|
| **Unauthorized network access**  | Use firewall rules to restrict access to trusted IPs/subnets                       |
| **No authentication by default** | Place behind a reverse proxy (Nginx, Caddy) with basic auth if exposing beyond LAN |
| **Model prompt injection**       | Sanitize inputs in client apps before sending to LLM                               |
| **Data persistence**             | Models stored in Docker volume `ollama`; back up `~/.ollama` if needed             |

> **Do not expose port 11434 to the public internet** without authentication and rate limiting.