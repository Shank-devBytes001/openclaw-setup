# 🦀 OpenClaw: VPS Set Up

[![Docker](https://img.shields.io/badge/Docker-24.0+-blue.svg?style=for-the-badge&logo=docker)](https://www.docker.com/)
[![NVIDIA](https://img.shields.io/badge/NVIDIA-NIM-green.svg?style=for-the-badge&logo=nvidia)](https://build.nvidia.com/)
[![Telegram](https://img.shields.io/badge/Telegram-Bot-blue.svg?style=for-the-badge&logo=telegram)](https://t.me/BotFather)

**OpenClaw** is a high-performance AI gateway designed to bridge powerful LLM providers (via NVIDIA NIM) with user-friendly interfaces like Telegram.

This deployment focuses on:

- ⚡ Low latency
- 🔒 Secure containerized infrastructure
- 🤖 Seamless Telegram integration
- 🧠 Kimi K2.5 model optimization
- 🐳 Production-grade Docker orchestration

This repository documents my real VPS production setup, including deployment challenges and solutions.

---

# 🏗 Architecture Overview

Production Stack:

- Linux VPS
- Docker + Docker Compose
- OpenClaw Gateway
- OpenClaw Browser Engine
- NVIDIA NIM (Moonshot API)
- Telegram Bot Integration
- Persistent volumes for state
- Secure environment variable handling

Everything runs containerized with controlled ports and isolated services.

---

# 🚀 Step-by-Step Production Setup

## 1️⃣ Environment Configuration

Create a `.env` file in the project root to securely store all secrets.

```bash
# API Keys
MOONSHOT_API_KEY='nvapi-your-nvidia-key'
TELEGRAM_BOT_TOKEN='your-telegram-token'

# Security
OPENCLAW_GATEWAY_TOKEN='your-secure-ui-password'
OPENCLAW_GATEWAY_BIND='lan'
```

> ⚠ **Important**
>
> All values are wrapped in single quotes (`' '`).
>
> This prevents Docker Compose interpolation errors caused by special characters like:
> `^` `&` `:` `$`
>
> Never hardcode secrets inside `docker-compose.yml`.

## 2️⃣ Docker Deployment

Create a `docker-compose.yml` file:

```yaml
services:
  openclaw-gateway:
    image: coollabsio/openclaw:latest
    container_name: openclaw-gateway
    restart: unless-stopped
    ports:
      - "18789:18789"
    env_file: .env
    environment:
      - NODE_ENV=production
      - OPENCLAW_PRIMARY_MODEL=kimi
      - OPENCLAW_AUTH_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - NVIDIA_API_KEY=${MOONSHOT_API_KEY}
      - TELEGRAM_BOT_TOKEN=${TELEGRAM_BOT_TOKEN}
    volumes:
      - ./data:/home/node/.openclaw
    shm_size: '2gb'
    links:
      - browser

  browser:
    image: coollabsio/openclaw-browser:latest
    container_name: openclaw-browser
    restart: unless-stopped
    shm_size: '2gb'
```

### Why This Configuration?

- `restart: unless-stopped` → Ensures uptime after VPS reboots
- `env_file` → Keeps secrets out of version control
- `volumes` → Persists chat memory and config
- `shm_size: 2gb` → Prevents browser engine crashes
- Separate browser service → Cleaner architecture & isolation

## 3️⃣ Launching the System

Pull images and start containers:

```bash
docker compose up -d --force-recreate
```

Verify running services:

```bash
docker ps
```

Access Gateway UI:

```
http://your-vps-ip:18789
```

---

# 🛠 Challenges Faced & Real Fixes

This wasn't plug-and-play. Here are the real issues I encountered and how I solved them.

<details>
<summary><b>1️⃣ Environment Variable Interpolation Error</b></summary>

### ❌ Problem

Docker Compose failed with:

```
invalid interpolation format
```

The issue was special characters inside API tokens.

### ✅ Fix

- Wrapped every value inside `.env` with single quotes.
- Removed any hardcoded keys inside `docker-compose.yml`.
- Referenced only variable names using `${VARIABLE_NAME}`.

This immediately stabilized container startup.

</details>

<details>
<summary><b>2️⃣ The "Silent Bot" (Telegram Connectivity Issue)</b></summary>

### ❌ Problem

The Telegram bot showed as online but never responded.
Gateway UI displayed "Missing env var" errors.

### ✅ Fix

- Added explicit channels configuration inside `openclaw.json` (Raw Config).
- Enabled Telegram plugin manually.
- Verified `${TELEGRAM_BOT_TOKEN}` mapping correctly passed to container.

After restarting containers, Telegram sync worked instantly.

</details>

<details>
<summary><b>3️⃣ UI Clutter (Untrusted Metadata JSON)</b></summary>

### ❌ Problem

Every response was followed by large blocks of JSON metadata, making chats unreadable.

### ✅ Fix

Disabled:
- Debug Mode
- Show Metadata

This cleaned the UI completely and made it feel like a proper AI assistant instead of a developer console.

</details>

---

# 🔒 Production Practices Implemented

This deployment was built with production thinking:

- No secrets inside repository
- Persistent storage mounted externally
- Container restart policy enforced
- Isolated services
- No unnecessary exposed ports
- Controlled environment configuration
- Clean separation of Gateway & Browser engine

---

# 🤖 Model Integration: Kimi K2.5 (via NVIDIA)

The intelligence layer runs Moonshot AI Kimi K2.5, served through NVIDIA NIM.

| Feature         | Status         |
| --------------- | -------------- |
| Context Window  | 200k Tokens    |
| Streaming       | Enabled ✅      |
| Telegram Sync   | Active ✅       |
| Response Speed  | Optimized ✅    |

Latency is consistently low and streaming responses are stable.

---

# 📦 Project Structure

```
.
├── docker-compose.yml
├── .env
├── data/              # Persistent OpenClaw state
└── README.md
```
