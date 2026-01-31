# Running LLMs Locally on Windows with AMD RX 7800

## Hardware Setup
- **GPU**: AMD Radeon RX 7800 (16GB VRAM)
- **Storage**: Dual SSDs (ideal for dual boot or WSL2)
- **RAM**: 32GB+ recommended
- **OS**: Windows 10/11

## ✅ Next Steps After Installing WSL + Ollama

### Step 1: Install AMD ROCm Drivers in WSL
```bash
# Update your system
sudo apt update && sudo apt upgrade -y

# Install ROCm for WSL
# Download and install AMD GPU driver for WSL
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

### Step 2: Test GPU Acceleration
```bash
# Test ROCm installation
cd /opt/rocm/bin
./rocminfo

# Test Ollama GPU detection
ollama --version
```

### Step 3: Install Qwen Models
```bash
# Install Qwen models (start with smaller one)
ollama pull qwen2.5:7b

# For better quality (if your GPU can handle it)
ollama pull qwen2.5:14b

# Code-focused model
ollama pull qwen2.5-coder:7b
```

### Step 4: Run Qwen
```bash
# Start interactive chat
ollama run qwen2.5:7b

# Or run as server for API access
ollama serve
```

### Step 5: Test GPU Usage
```bash
# Monitor GPU usage while running
nvidia-smi  # Wait, use rocm-smi for AMD
rocm-smi

# Or use radeontop for monitoring
sudo apt install radeontop
radeontop
```

## Troubleshooting

### If GPU isn't detected:
```bash
# Restart WSL
wsl --shutdown
wsl

# Check ROCm installation
/opt/rocm/bin/rocminfo
```

### Performance Issues:
- Ensure Windows GPU drivers are up to date
- Close other GPU-intensive applications
- Monitor temperature: `rocm-smi --showtemp`

### Memory Issues:
- Start with smaller models (7B instead of 14B)
- Use quantization if needed
- Close other applications using RAM

## Recommended Approach: WSL2 (Easiest)

### Quick WSL2 Setup
```powershell
# Enable WSL2 and install Ubuntu
wsl --install -d Ubuntu-22.04

# Install GPU drivers (AMD ROCm)
# Download from: https://www.amd.com/en/developer/resources/rocm-hub/hip-sdk.html

# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh
ollama pull qwen2.5:7b
ollama run qwen2.5:7b
```

### Alternative: Dual Boot (Maximum Performance)
- **SSD 1**: Windows (primary)
- **SSD 2**: Ubuntu 22.04 LTS (ML workloads)

**Setup Steps:**
1. Backup Windows system
2. Download Ubuntu 22.04: https://ubuntu.com/download/desktop
3. Create bootable USB with Rufus
4. Install Ubuntu on second SSD
5. Configure GRUB dual boot

## LLM Options

### Ollama (Recommended)
```bash
# Popular models for RX 7800
ollama pull qwen2.5:7b          # ~14GB, fast inference
ollama pull qwen2.5:14b         # ~28GB, better quality
ollama pull qwen2.5-coder:7b    # Code-focused
```

### Alternative Tools
- **LM Studio**: GUI interface, download from https://lmstudio.ai/
- **Text Generation WebUI**: Advanced web interface
- **GPT4All**: User-friendly GUI

## Performance Tips

### For AMD GPU
- Use latest AMD drivers
- Enable ROCm for WSL2
- Monitor GPU temperature during ML workloads
- Use SSD with fastest read speeds for model storage

### Memory Management
| Model Size | VRAM Required | Recommended Setup |
|------------|---------------|-------------------|
| 7B params  | 8GB+ | All configurations |
| 14B params | 16GB+ | WSL2 or Dual Boot |
| 32B params | 24GB+ | Dual Boot + 4-bit quantization |

## Why Linux?
- Superior AMD GPU support (ROCm)
- Native ML framework compatibility
- Better memory management
- Most ML tools are Linux-first

## Web UIs
- **Ollama WebUI**: `pip install ollama-webui`
- **Open WebUI**: https://github.com/open-webui/open-webui
- **Text Generation WebUI**: https://github.com/oobabooga/text-generation-webui

## Getting Help
- Ollama docs: https://github.com/ollama/ollama
- AMD ROCm: https://docs.amd.com/
- Qwen models: https://qwenlm.github.io/
