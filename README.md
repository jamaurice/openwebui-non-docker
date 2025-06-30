# 🧠 Open WebUI – Manual Setup Without Docker

This guide walks you through deploying Open WebUI **without Docker**, fixing common compatibility issues, and running it cleanly on your own infrastructure.

---

## 📦 Overview

**Goal**: Run Open WebUI directly using FastAPI + SvelteKit **without Docker or containerization**, fully self-hosted.

**Environment**:
- OS: Ubuntu 22.04+ (or similar)
- RAM: 16GB minimum (32GB+ preferred for build)
- Disk: 2GB free for dependencies + swap
- Internet access during setup

---

## ✅ Requirements & Manual Installs

Install the following system dependencies:

```bash
sudo apt update
sudo apt install -y \
  git curl wget unzip \
  build-essential python3.10 python3.10-venv python3-pip \
  nodejs npm
````

> If Node.js version is too old (check with `node -v`), use:

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

---

## 🧪 Python Virtual Environment Setup

```bash
cd open-webui/backend
python3.10 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

---

## 🛠️ Modifications Required to Run Successfully

### 1. ❌ Python 3.10 lacks `StrEnum`

**File**: `open_webui/retrieval/vector/type.py`

🔧 **Fix**:

```python
# Replace:
# from enum import StrEnum

# With:
from enum import Enum
class StrEnum(str, Enum):
    pass
```

---

### 2. ❌ `logging.getLevelNamesMapping()` is missing in 3.10

**File**: `open_webui/env.py` and possibly `utils/logger.py`

🔧 **Fix**:

```python
# Old (incompatible):
if SRC_LOG_LEVELS[source] not in logging.getLevelNamesMapping():

# Replace with a fallback check like:
valid_levels = {"DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"}
if SRC_LOG_LEVELS[source] not in valid_levels:
    ...
```

---

### 3. ✅ Make sure FastAPI entry point loads correctly

**File**: `open_webui/main.py`

Verify last line is:

```python
app = create_app()
```

Then run:

```bash
python3 -m uvicorn open_webui.main:app --host 0.0.0.0 --port 3000
```

---

## 🧼 `build.sh` Script to Handle Frontend + Memory Issues

Create a file in the project root named `build.sh`:

```bash
chmod +x build.sh
```

```bash
#!/bin/bash

set -e
set -o pipefail

echo "🧼 Cleaning old builds..."
rm -rf .svelte-kit dist node_modules/.vite

echo "📦 Setting memory limits for Node.js..."
export NODE_OPTIONS="--max-old-space-size=8192"

echo "💾 Ensuring swap file exists (8GB)..."
if ! grep -q '/swapfile' /etc/fstab; then
  sudo fallocate -l 8G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
else
  echo "Swap file already exists."
fi

echo "🚀 Running npm install..."
npm install

echo "🛠️  Building Open WebUI frontend..."
npm run build

echo "✅ Build complete!"
```

---

## 🔁 Run the App

```bash
cd backend
source venv/bin/activate
python3 -m uvicorn open_webui.main:app --host 0.0.0.0 --port 3000
```

Access it at: [http://localhost:3000](http://localhost:3000)

---

## 🗂️ List of Manually Modified Files

| File Path                                     | Modification                                                                |
| --------------------------------------------- | --------------------------------------------------------------------------- |
| `backend/open_webui/env.py`                   | Replaced `logging.getLevelNamesMapping()` with safe fallback logic          |
| `backend/open_webui/utils/logger.py`          | Same logging compatibility fix (if used there)                              |
| `backend/open_webui/retrieval/vector/type.py` | Added `StrEnum` compatibility patch for Python 3.10                         |
| `build.sh` (custom)                           | Added swap creation, heap allocation, clean build script for large frontend |

---

## 🔐 Not Using Docker or Cloudflare

This deployment skips:

* Docker
* Docker Compose
* Any Cloudflare Tunnels or config files

You manage access yourself using:

* Reverse proxy (e.g., NGINX)
* TLS certs (e.g., Let’s Encrypt)
* Systemd service

---

## 🧠 Optional: Add to systemd

If you want to run it at boot:

```ini
# /etc/systemd/system/openwebui.service
[Unit]
Description=Open WebUI Backend Service
After=network.target

[Service]
Type=simple
User=user-name
WorkingDirectory=/home/user-name/open-webui/backend
ExecStart=/home/user-name/open-webui/backend/venv/bin/python3 -m uvicorn open_webui.main:app --host 0.0.0.0 --port 3000
Restart=on-failure
Environment=OLLAMA_API_BASE_URL=http://127.0.0.1:11434

[Install]
WantedBy=multi-user.target
```

Then enable:

```bash
sudo systemctl daemon-reexec
sudo systemctl enable openwebui
sudo systemctl start openwebui
```

---

## ✅ Final Checklist

* [x] Python venv initialized
* [x] Dependencies installed
* [x] `StrEnum` patched
* [x] Logging fixed
* [x] Frontend built with memory limits
* [x] Backend launched via `uvicorn`

---

## 📜 License

MIT
