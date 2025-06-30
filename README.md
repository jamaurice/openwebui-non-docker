# 🧠 Open WebUI – Manual Deployment (Non-Docker)

A lightweight local-first web-based UI for running LLMs via Ollama. This setup runs completely outside Docker and is optimized for high-memory servers.

---

## ✅ Requirements

- Python 3.10+ (preferably 3.11+ to support `StrEnum`)
- Node.js (v18+)
- npm
- `virtualenv`

---

## 🧩 Setup Overview

This guide helps you:

1. Manually configure and run the backend using FastAPI (uvicorn).
2. Build the frontend using Vite/SvelteKit.
3. Avoid Docker entirely.
4. Enable large model builds on 64GB+ RAM environments.

---

## 🔧 Manual Setup

### 1. Clone the repo and navigate:

```bash
git clone https://github.com/YOUR_USERNAME/open-webui.git
cd open-webui
````

### 2. Create a Python virtual environment:

```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3. Build frontend (from root of repo):

```bash
chmod +x build.sh
./build.sh
```

This script will:

* Clean `.svelte-kit` and Vite cache
* Set `--max-old-space-size=8192` for large frontend builds
* Allocate 8GB swap if missing
* Run `npm install` and `npm run build`

---

## 🛠️ Modified Files

To get Open WebUI running outside Docker, the following files were updated:

### ✅ `backend/open_webui/env.py`

```python
# Fixed invalid attribute:
# Replaced: logging.getLevelNamesMapping()
# With fallback or removed if unnecessary for your use case
```

### ✅ `backend/open_webui/retrieval/vector/type.py`

```python
# Replaced Python 3.11+ only import:
# from enum import StrEnum
# With:
from enum import Enum
class StrEnum(str, Enum):
    pass
```

---

## ▶️ Running Backend

```bash
cd backend
source venv/bin/activate
python3 -m uvicorn open_webui.main:app --host 0.0.0.0 --port 3000
```

Now access: [http://localhost:3000](http://localhost:3000)

---

## 🔁 build.sh Reference

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
fi

echo "🚀 Running npm install..."
npm install

echo "🛠️  Building Open WebUI frontend..."
npm run build

echo "✅ Build complete!"
```

---

## 🧠 Notes

* Make sure NodeJS has at least 8GB heap or large builds will fail with OOM errors.
* This setup is ideal for **bare-metal servers** with high RAM allocations.

---

## 📜 License

MIT

```
