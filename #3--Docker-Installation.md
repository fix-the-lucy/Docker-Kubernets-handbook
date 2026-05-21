# 🐳 Installing Docker – Complete Guide

> **Topic:** How to Install Docker on Mac, Windows & Linux  
> **Date:** May 21, 2026

---

## 📋 Overview

| Topic | Description |
|---|---|
| ✅ Prerequisites | What you need before installing |
| 🍎 Mac | Docker Desktop for Mac |
| 🪟 Windows | Docker Desktop for Windows |
| 🐧 Linux | Docker Engine via terminal |
| 🧰 Docker Toolbox | For older Mac & Windows machines |

---

## ✅ Step 1 — Prerequisites (Before Installing)

Before installing Docker, check the following:

- Your **OS version** is supported (64-bit required)
- **Virtualization** is enabled in your BIOS/system settings
- You have **admin/sudo** privileges on your machine
- Sufficient **RAM**: minimum 4 GB recommended

---

## 🍎 Step 2 — Install Docker on Mac

> Requires: macOS 11 (Big Sur) or newer

1. Go to 👉 [https://docs.docker.com/desktop/install/mac-install/](https://docs.docker.com/desktop/install/mac-install/)
2. Download **Docker Desktop for Mac**
   - Choose **Apple Silicon** (M1/M2/M3) or **Intel Chip** based on your Mac
3. Open the `.dmg` file and drag Docker to **Applications**
4. Launch **Docker Desktop** from Applications
5. Verify installation:

```bash
docker --version
docker run hello-world
```

---

## 🪟 Step 3 — Install Docker on Windows

> Requires: Windows 10/11 (64-bit), WSL 2 enabled

1. Go to 👉 [https://docs.docker.com/desktop/install/windows-install/](https://docs.docker.com/desktop/install/windows-install/)
2. Download **Docker Desktop for Windows**
3. Run the installer — make sure **"Use WSL 2"** is checked
4. Restart your computer after installation
5. Launch **Docker Desktop** from the Start menu
6. Verify installation:

```bash
docker --version
docker run hello-world
```

> 💡 **WSL 2** (Windows Subsystem for Linux) must be enabled. Run this in PowerShell as Admin:
> ```powershell
> wsl --install
> ```

---

## 🐧 Step 4 — Install Docker on Linux

> Example for Ubuntu/Debian

Run the following commands in your terminal:

```bash
# 1. Update packages
sudo apt-get update

# 2. Install dependencies
sudo apt-get install ca-certificates curl gnupg

# 3. Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 4. Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# 6. Verify
sudo docker run hello-world
```

---

## 🧰 Step 5 — Docker Toolbox (For Older Machines)

> Use this if your machine **doesn't support** Docker Desktop

### 🍎 Docker Toolbox for Older Mac
- Supports macOS **10.13 or older**
- Uses **VirtualBox** instead of native hypervisor
- Download: [https://github.com/docker-archive/toolbox/releases](https://github.com/docker-archive/toolbox/releases)
- After install, use **Docker Quickstart Terminal** instead of regular terminal

### 🪟 Docker Toolbox for Older Windows
- Supports Windows **7, 8, or 10 Home (older builds)**
- Also uses **VirtualBox** underneath
- Download same link above
- After install, launch **Docker Quickstart Terminal**

> ⚠️ Docker Toolbox is **legacy** and no longer actively maintained. Upgrade your OS if possible.

---

## ✔️ Verify Everything Works

Run this on any OS after installation:

```bash
docker --version       # shows Docker version
docker ps              # lists running containers (empty = Docker is working)
docker run hello-world # pulls and runs a test container
```

Expected output from `hello-world`:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

---

## 📝 Quick Summary

```
Mac (new)      → Docker Desktop → drag to Applications
Windows (new)  → Docker Desktop → enable WSL 2
Linux          → Install via terminal commands
Old Mac/Win    → Docker Toolbox (uses VirtualBox)
```

---

## 🔗 Official Resources

- 📖 [Docker Docs](https://docs.docker.com)
- 🐋 [Docker Hub](https://hub.docker.com)
- 💬 [Docker Community Forums](https://forums.docker.com)
- 
