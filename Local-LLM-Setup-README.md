# Running LLMs Locally on Windows with AMD RX 7800 XT

## Hardware Setup (Verified)
- **GPU**: AMD Radeon RX 7800 XT (16GB VRAM)
- **Architecture**: gfx1101 (RDNA 3)
- **CPU**: AMD Ryzen 7 7700X 8-Core Processor
- **Compute Units**: 60
- **Storage**: Dual SSDs (ideal for dual boot or WSL2)
- **RAM**: 32GB+ recommended
- **OS**: Windows 10/11 with WSL2 Ubuntu

## Quick Start (WSL2 + Ollama)

### Step 1: Install WSL2 Ubuntu
```powershell
# Run in PowerShell as Administrator
wsl --install -d Ubuntu-22.04
```

### Step 2: Install ROCm Drivers in WSL
```bash
# Update your system
sudo apt update && sudo apt upgrade -y

# Install ROCm for WSL
wget https://repo.radeon.com/amdgpu-install/6.1/ubuntu/jammy/amdgpu-install_6.1.60101-1_all.deb
sudo dpkg -i amdgpu-install_6.1.60101-1_all.deb
sudo apt update
sudo apt install -y amdgpu-dkms rocm-hip-runtime

# Add user to render group
sudo usermod -a -G render $USER

# Install HIP runtime
sudo apt install -y hip-runtime-amd rocm-device-libs

# Verify installation
rocminfo | grep -E "(Name|Device Type)"
```

### Step 3: Configure GPU Override for RDNA 3 (gfx1101)
```bash
# Required for RX 7800 XT (gfx1101) compatibility
echo 'export HSA_OVERRIDE_GFX_VERSION=11.0.0' >> ~/.bashrc
source ~/.bashrc
```

### Step 4: Install Ollama
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Step 5: Run Ollama with GPU
```bash
# Start Ollama service (runs in background)
ollama serve &

# Pull a model
ollama pull qwen2.5:7b

# Run the model
ollama run qwen2.5:7b
```

### Step 6: Verify GPU Usage
```bash
# Monitor GPU while running models
rocm-smi

# Or use radeontop for real-time monitoring
sudo apt install radeontop
radeontop
```

## Recommended Models for 16GB VRAM

```bash
# Fast inference (fits easily in 16GB)
ollama pull qwen2.5:7b
ollama pull llama3.2:7b
ollama pull mistral:7b

# Code-focused
ollama pull qwen2.5-coder:7b
ollama pull codellama:7b

# Larger models (use more VRAM)
ollama pull qwen2.5:14b
ollama pull llama3.1:13b
```

## Model Size vs VRAM

| Model Size | VRAM Required | Fits RX 7800 XT? |
|------------|---------------|------------------|
| 7B params  | ~8GB          | ✅ Easily        |
| 13B params | ~12GB         | ✅ Yes           |
| 14B params | ~14GB         | ✅ Yes           |
| 32B params | ~20GB+        | ⚠️ Needs quantization |
| 70B params | ~40GB+        | ❌ Too large     |

## Troubleshooting

### GPU Not Detected
```bash
# Restart WSL completely
wsl --shutdown
wsl

# Verify ROCm sees your GPU
rocminfo | grep "Marketing Name"

# Should output: AMD Radeon RX 7800 XT
```

### Ollama Not Using GPU
```bash
# Ensure HSA override is set
echo $HSA_OVERRIDE_GFX_VERSION
# Should output: 11.0.0

# If not set, add it
export HSA_OVERRIDE_GFX_VERSION=11.0.0

# Restart Ollama
pkill ollama
ollama serve &
```

### Windows Driver Warning
If `rocminfo` shows "Warning: Windows driver is old":
1. Download latest drivers from [AMD Drivers](https://www.amd.com/en/support)
2. Install on Windows (not WSL)
3. Restart computer
4. Run `wsl --shutdown` then start WSL again

### Performance Issues
- Ensure Windows GPU drivers are up to date
- Close other GPU-intensive applications
- Monitor temperature: `rocm-smi --showtemp`

### Memory Issues
- Start with smaller models (7B instead of 14B)
- Use quantized models (e.g., `qwen2.5:7b-q4_0`)
- Close other applications using RAM

## Alternative: Dual Boot (Maximum Performance)
- **SSD 1**: Windows (primary)
- **SSD 2**: Ubuntu 22.04 LTS (ML workloads)

Native Linux provides ~10-15% better performance than WSL2.

## Web UIs for Ollama

### Open WebUI (Recommended)
```bash
# Using Docker
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:main
```
Access at: http://localhost:3000

### Alternative Tools
- **LM Studio**: GUI interface, download from https://lmstudio.ai/
- **Text Generation WebUI**: https://github.com/oobabooga/text-generation-webui

## ROCm Info Reference

Your verified GPU configuration:
```
Agent 2: AMD Radeon RX 7800 XT
  Architecture:     gfx1101 (RDNA 3)
  VRAM:            16GB
  Compute Units:   60
  Max Clock:       2124 MHz
  Fast F16:        TRUE
  Wavefront Size:  32
```

## Resources
- Ollama: https://github.com/ollama/ollama
- AMD ROCm: https://docs.amd.com/
- Qwen models: https://qwenlm.github.io/
- Open WebUI: https://github.com/open-webui/open-webui
