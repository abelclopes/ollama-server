# Ollama Server - Multi-GPU Configurations

Configurações Docker para rodar Ollama com diferentes GPUs.

## Servidor Principal (NVIDIA)

| Arquivo | GPUs | Portas |
|---------|------|--------|
| `docker-compose.yml` | RTX 3060 12GB | 11434 |
| `docker-compose.dual-gpu.yml` | RTX 3060 + GTX 1060 | 11434, 11435 |

### Uso NVIDIA

```bash
# GPU única
docker compose up -d

# Dual GPU
docker compose -f docker-compose.dual-gpu.yml up -d
```

---

## Servidor Secundário (AMD)

| Arquivo | GPUs | Portas |
|---------|------|--------|
| `docker-compose.amd-gpu.yml` | RX 9060 16GB | 11434 |
| `docker-compose.amd-dual.yml` | RX 9060 + RX 6600 XT | 11434, 11435, 11430 (LB) |

### Pré-requisitos AMD

```bash
# Instalar drivers ROCm
wget https://repo.radeon.com/amdgpu-install/latest/ubuntu/jammy/amdgpu-install_6.0.60000-1_all.deb
sudo dpkg -i amdgpu-install_6.0.60000-1_all.deb
sudo amdgpu-install --usecase=rocm

# Adicionar usuário aos grupos
sudo usermod -a -G video,render $USER
sudo reboot

# Verificar instalação
rocm-smi
```

### Uso AMD

```bash
# GPU única (RX 9060)
docker compose -f docker-compose.amd-gpu.yml up -d

# Dual GPU (RX 9060 + RX 6600 XT)
docker compose -f docker-compose.amd-dual.yml up -d
```

---

## Arquitetura

```
┌─────────────────────────────────────────────────────────────┐
│                    Servidor Principal                        │
│                     (NVIDIA GPUs)                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────┐                 │
│  │   RTX 3060      │    │   GTX 1060      │                 │
│  │   12GB VRAM     │    │   6GB VRAM      │                 │
│  │   :11434        │    │   :11435        │                 │
│  └─────────────────┘    └─────────────────┘                 │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   Servidor Secundário                        │
│                      (AMD GPUs)                             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐    ┌─────────────────┐                 │
│  │   RX 9060       │    │   RX 6600 XT    │                 │
│  │   16GB VRAM     │    │   8GB VRAM      │                 │
│  │   :11434        │    │   :11435        │                 │
│  └────────┬────────┘    └────────┬────────┘                 │
│           │                      │                           │
│           └──────────┬───────────┘                          │
│                      │                                       │
│           ┌──────────▼──────────┐                           │
│           │   Load Balancer     │                           │
│           │      :11430         │                           │
│           └─────────────────────┘                           │
└─────────────────────────────────────────────────────────────┘
```

---

## Modelos Recomendados por GPU

| GPU | VRAM | Modelos |
|-----|------|---------|
| RX 9060 | 16GB | llama3:13b, mixtral, codellama:13b, qwen3:14b |
| RTX 3060 | 12GB | llama3:8b, mistral, codellama:7b, qwen3 |
| RX 6600 XT | 8GB | qwen3, mistral, deepseek-coder, llama3:8b |
| GTX 1060 | 6GB | qwen3, phi3, gemma:2b |

---

## Endpoints

| Serviço | URL |
|---------|-----|
| Ollama GPU 0 | http://localhost:11434 |
| Ollama GPU 1 | http://localhost:11435 |
| Load Balancer | http://localhost:11430 |

---

## Comandos Úteis

```bash
# Verificar GPUs NVIDIA
nvidia-smi

# Verificar GPUs AMD
rocm-smi

# Logs
docker compose logs -f ollama-gpu0

# Baixar modelo
docker exec ollama-gpu0 ollama pull qwen3

# Testar
curl http://localhost:11434/api/generate -d '{
  "model": "qwen3",
  "prompt": "Hello!",
  "stream": false
}'
```
