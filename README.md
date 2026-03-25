# remote-llm-server
## Build
### Install docker
```
docker compose up --build
```

### Load models
```
docker exec -it ollama ollama pull qwen3.5:9b-q4_K_M  
docker exec -it ollama ollama pull qwen3.5:4b
```

### Test run of models (Windows)
```
curl http://localhost:11434/api/tags
```

### Turn on Firewall
```
New-NetFirewallRule -DisplayName "Ollama Docker" -Direction Inbound -Protocol TCP -LocalPort 11434 -Action Allow
```

### Find your local IP
```
ipconfig
```