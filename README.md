# local_inference

Local inference using machines equipped with the AMD AI Max-395 processor

## 📌 Context

As of June 2025, the local inference market for businesses is booming, particularly in China. Users seek complete data privacy, which major providers like Deepseek or ChatGPT cannot fully guarantee, as protecting sensitive business data remains crucial.

In response to this need, AMD unveiled its **AIMax+395** processor in early 2025, alongside NVIDIA’s DIGI-X. The key advantage of AMD’s solution is that, by June 2025, machines are already available through at least one supplier, with more on the way.

Machines based on the AMD64 architecture are typically compatible with most operating systems and FOSS (Free and Open Source Software), offering users a wide range of choices for software tools.

On the other hand, NVIDIA’s ARM-based architecture still lags in tool availability—though it’s catching up. However, users might have to wait for ARM-compatible versions of certain tools or be locked into NVIDIA’s ecosystem.

For AMD64-based systems, **GMKtec** was first to market. For under €2,000, you can buy the EVO-X2 with 128GB RAM and an integrated GPU capable of inference—that is, running AI models within the machine’s “limited” resources.

---

## ⚙️ Machine Choice

As of June 2025, the top pick is GMKTEC’s [EVO-X2](https://www.gmktec.com/products/amd-ryzen%E2%84%A2-ai-max-395-evo-x2-ai-mini-pc):

- First manufacturer to deliver systems  
- Multiple batches shipped since late May with good feedback  
- Several BIOS updates already released  

💰 Price: Under €2,000 for the 128GB RAM version

The motherboard is an [OEM product by Sixunted](https://strixhalo-homelab.d7.wtf/Hardware/Boards/Sixunited-AXB35), used in several machines:

- GMKtec EVO-X2 (SU_AXB35-02_FP11)  
- [Bosgame M5](https://www.bosgamepc.com/products/bosgame-m5-ai-mini-desktop-ryzen-ai-max-395) – €1,440 pre-order  
- [Peladn YO1](https://peladn.com/products/yo1-395-mini-pc) – ¥12,000 in China  
- FEVM FA-EX9  

GMKtec is **available now**.

---

## 🧩 Software Stack

- OS of your choice (e.g., [PopOS](https://system76.com/pop/))  
- GPU drivers: [ROCm](https://rocm.docs.amd.com/en/latest/)  
- [Docker for Ubuntu](https://docs.docker.com/engine/install/ubuntu/)  
- [Portainer](https://docs.portainer.io/) for web-based Docker UI  
- AI Tools:
  - [Ollama](https://ollama.com/) ([AMD GPU installation guide](https://hub.docker.com/r/ollama/ollama))  
  - [Open WebUI](https://github.com/open-webui/open-webui)

---

## 🔗 Additional Resources

- 💬 Discord: Strix Halo Homelab → https://discord.com/channels/1384139280020148365  
- 🌐 Strix Halo WTF → https://strixhalo-homelab.d7.wtf/  
- 🧵 Reddit: [AMD AI MAX 395](https://www.reddit.com/r/Amd/comments/1jdgq69/amd_ryzen_ai_max_395_processor_breakthrough_ai/?tl=fr)

---

## 🚀 Installation Guide

### 🔧 BIOS Settings

- Allocate max RAM to GPU = 96GB  
- Enable "Performance" mode → 120W average, 140W peak CPU  

---

### 🐧 PopOS Installation

Installed in June 2025 using PopOS, but any Linux distro works.

PopOS is optimized for performance and ease of use, and while it supports NVIDIA GPUs very well, **we’re using AMD here**!

📥 [Download PopOS](https://system76.com/pop/download)

---

### 📦 ROCm Drivers Installation

```bash
wget https://repo.radeon.com/amdgpu-install/6.4.1/ubuntu/jammy/amdgpu-install_6.4.60401-1_all.deb
sudo apt install ./amdgpu-install_6.4.60401-1_all.deb
sudo apt update
sudo apt install python3-setuptools python3-wheel
sudo usermod -a -G render,video $LOGNAME
sudo apt install rocm
wget https://repo.radeon.com/amdgpu-install/6.4.1/ubuntu/jammy/amdgpu-install_6.4.60401-1_all.deb
sudo apt install ./amdgpu-install_6.4.60401-1_all.deb
sudo apt update
sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
sudo apt install amdgpu-dkms
sudo reboot
```

---

### 🐳 Docker for Ubuntu

📄 [Docs](https://docs.docker.com/engine/install/ubuntu/)

#### Set up the repo

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

#### Install Docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

### 📊 Portainer Setup

📄 [Docs](https://docs.portainer.io/)

```bash
sudo mkdir -p /portainer/Files/AppData/Config/portainer
sudo docker pull portainer/portainer-ce:latest
sudo docker run -d -p 29000:9000 -p 9443:9443 --name=portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /portainer/Files/AppData/Config/portainer:/data \
  portainer/portainer-ce:latest
```

---

### 🤖 Ollama Installation (AMD GPU)

📄 [Docs](https://hub.docker.com/r/ollama/ollama)

Note 0.10.1-rocm in 2025-08-01

```bash
sudo docker run -d --device /dev/kfd --device /dev/dri \
  -v ollama:/root/.ollama -p 11434:11434 \
  --name ollama ollama/ollama:0.10.1-rocm
```

---

### 🧠 Open WebUI

- 📘 Repo: https://github.com/open-webui/open-webui  
- 📖 Guide: https://docs.openwebui.com/

Example `docker-compose.yml`:

```yaml
services:
  open-webui:
    container_name: open-webui
    image: ghcr.io/open-webui/open-webui:0.6.15
    restart: on-failure:3
    environment:
      - OLLAMA_BASE_URL=http://host.docker.internal:11434
      # - WEBUI_AUTH=false
      - USE_OLLAMA_DOCKER=true
    volumes:
      - /portainer/Files/AppData/Config/open-webui:/app/backend/data
    ports:
      - 42182:8080
    extra_hosts:
      - host.docker.internal:host-gateway
```

#### 🔐 Authentication Notes

- If `WEBUI_AUTH=false`, the site is open to all users  
- If not set, the first login becomes the admin  
  - The admin must manage user accounts  
- In production, enable authentication and secure the network access

Start the container:

```bash
sudo docker compose up -d
```

---

### ✅ Getting Started

- Inside the Ollama container, pull a model:

```bash
ollama pull mistral-small3.2:24b
```

- In your browser:
  - Go to the mapped port
  - Create the admin account
  - Start using the system!
