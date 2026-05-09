# OpenClaw Production Deployment Guide

![OpenClaw](https://img.shields.io/badge/OpenClaw-Production-blue)
![Google Cloud](https://img.shields.io/badge/Google%20Cloud-GCP-4285F4?logo=google-cloud)
![Node.js](https://img.shields.io/badge/Node.js-24.x-339933?logo=node.js)
![Cloudflare](https://img.shields.io/badge/Cloudflare-CDN-F38020?logo=cloudflare)
![PM2](https://img.shields.io/badge/PM2-Process%20Manager-2B037A)

Complete production deployment guide for **OpenClaw** AI agent platform on Google Cloud Platform with enterprise-grade infrastructure including Cloudflare CDN, Caddy reverse proxy, and PM2 process management.

> **Target Audience**: DevOps engineers and developers deploying OpenClaw to production. All commands are copy-pasteable. Replace placeholder values like `SEU_TOKEN`, `SEU_DOMINIO`, and `SUA_CHAVE` with your actual credentials.

## 🎯 What You'll Build

By following this guide, you'll deploy a production-ready OpenClaw instance with:

- ✅ **Secure HTTPS** access via Cloudflare
- ✅ **24/7 uptime** with PM2 process manager  
- ✅ **Reverse proxy** with automatic SSL via Caddy
- ✅ **AI integrations**: OpenAI, Whisper, Google Places, Notion, Telegram, Firecrawl, ElevenLabs
- ✅ **Scalable infrastructure** on Google Cloud Platform
- ✅ **Production monitoring** and logging
- ✅ **Device pairing** security
- ✅ **Automatic restarts** on failure


## 📑 Table of Contents

- [Tech Stack](#-tech-stack)
- [Prerequisites](#-prerequisites)
- [Architecture Overview](#-architecture-overview)
- [Cost Considerations](#-cost-considerations)
- [Getting Started](#-getting-started)
  - [1. Infrastructure Setup](#1-infrastructure-setup)
  - [2. OpenClaw Installation](#2-openclaw-installation)
  - [3. Gateway Configuration](#3-gateway-configuration)
  - [4. Process Management](#4-process-management)
- [API Integrations](#-api-integrations)
- [Testing & Validation](#-testing--validation)
- [Troubleshooting](#-troubleshooting)
- [Maintenance](#-maintenance)
- [Security Checklist](#-security-checklist)
- [Reference](#-reference)

---

## 🛠 Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Platform** | Google Cloud Platform | VM hosting and infrastructure |
| **OS** | Ubuntu 22.04/24.04 LTS | Server operating system |
| **Runtime** | Node.js 24.x | JavaScript runtime for OpenClaw |
| **AI Platform** | OpenClaw | AI agent orchestration |
| **Reverse Proxy** | Caddy 2.x | Automatic HTTPS and routing |
| **CDN/Security** | Cloudflare | DDoS protection, SSL, caching |
| **Process Manager** | PM2 | 24/7 uptime and monitoring |
| **AI APIs** | OpenAI, Whisper, Google Places, Notion, Telegram, Firecrawl, ElevenLabs | AI capabilities and integrations |

---

## 📋 Prerequisites

Before starting, ensure you have:

### Required

- [ ] **Google Cloud Account** with Compute Engine enabled
- [ ] **Domain or subdomain** (e.g., `openclaw.alobexpress.com.br`)
- [ ] **Cloudflare Account** managing your domain's DNS
- [ ] **OpenAI API Key** for AI model access
- [ ] **SSH access** to your local terminal

### Optional (for extended features)

- [ ] Google Places API key (for location-based searches)
- [ ] Notion API integration token (for database operations)
- [ ] Telegram Bot Token (for chat integrations)
- [ ] Firecrawl API key (for web scraping)
- [ ] ElevenLabs API key (for voice synthesis)

---

## 🏗 Architecture Overview

### System Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                    User / Browser                            │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              https://openclaw.alobexpress.com.br             │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│         Cloudflare (Proxy Mode - Orange Cloud)               │
│         • DDoS Protection                                    │
│         • SSL/TLS Termination                                │
│         • CDN Caching                                        │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Google Cloud VM (e2-standard-2)                 │
│              Ubuntu 22.04 LTS                                │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Caddy Reverse Proxy (Ports 80/443)                    │ │
│  │  • Automatic HTTPS                                     │ │
│  │  • HTTP/2 Support                                      │ │
│  └──────────────────────┬─────────────────────────────────┘ │
│                         │                                    │
│                         ▼                                    │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  OpenClaw Gateway (127.0.0.1:18789)                    │ │
│  │  • Managed by PM2                                      │ │
│  │  • Auto-restart on failure                             │ │
│  │  • Log aggregation                                     │ │
│  └──────────────────────┬─────────────────────────────────┘ │
│                         │                                    │
│                         ▼                                    │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  AI Agents & Integrations                              │ │
│  │  • OpenAI/Whisper                                      │ │
│  │  • Google Places                                       │ │
│  │  • Notion                                              │ │
│  │  • Telegram                                            │ │
│  │  • Firecrawl                                           │ │
│  │  • ElevenLabs                                          │ │
│  │  • Browser Automation                                  │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Why This Architecture?

| Benefit | Explanation |
|---------|-------------|
| **Security** | Port 18789 never exposed publicly; all traffic through Cloudflare + Caddy |
| **Reliability** | PM2 ensures gateway restarts automatically on crashes |
| **Performance** | Cloudflare CDN caches static assets globally |
| **Scalability** | Easy to upgrade VM size as usage grows |
| **Maintainability** | Separate concerns: Cloudflare (edge), Caddy (proxy), OpenClaw (app) |
| **SSL/TLS** | Automatic certificate management via Caddy |

---

## 💰 Cost Considerations


### Google Cloud Platform

**Recommended Starting Configuration:**

| Spec | Value | Purpose |
|------|-------|---------|
| **Machine Type** | e2-standard-2 | Cost-effective for initial deployment |
| **vCPUs** | 2 | Handles moderate agent workloads |
| **RAM** | 8 GB | Sufficient for OpenClaw + Node.js |
| **Disk** | 50 GB SSD | OS + OpenClaw + logs |
| **OS** | Ubuntu 22.04/24.04 LTS | Long-term support |
| **Region** | us-central1 | Lower cost option |

**Estimated Monthly Cost**: ~$50-70 USD (varies by region and usage)

**When to Scale Up:**

Upgrade to `e2-standard-4` (4 vCPU, 16 GB RAM, 100 GB disk) if you experience:
- Multiple concurrent AI agents
- Heavy browser automation workloads
- High memory usage (check with `pm2 monit`)
- Slow response times

**How to Scale:**
```bash
# Stop VM
gcloud compute instances stop openclaw-alobexpress --zone=us-central1-c

# Change machine type
gcloud compute instances set-machine-type openclaw-alobexpress \
  --machine-type=e2-standard-4 \
  --zone=us-central1-c

# Start VM
gcloud compute instances start openclaw-alobexpress --zone=us-central1-c
```

### OpenAI API Costs

**Important**: ChatGPT subscription (Plus/Team/Business) is **separate** from OpenAI API billing.

| Resource | Link | Purpose |
|----------|------|---------|
| **Usage Dashboard** | https://platform.openai.com/usage | Monitor API consumption |
| **Billing** | https://platform.openai.com/settings/organization/billing/overview | Add payment method |
| **Rate Limits** | https://platform.openai.com/settings/organization/limits | Set spending caps |

**Recommendations:**
- Start with a **$5-10 USD monthly limit** to avoid surprises
- Monitor usage daily during initial testing
- Free tier includes $5 credit (expires after 3 months)
- Production workloads: budget $50-200/month depending on usage

**Cost Optimization Tips:**
- Use `gpt-4o-mini` for simple tasks (cheaper than `gpt-4`)
- Cache responses when possible
- Implement rate limiting in your agents
- Use Whisper only when necessary (audio transcription costs add up)

---

## 🚀 Getting Started

### 1. Infrastructure Setup

#### Step 1.1: Create Google Cloud VM

1. Navigate to [Google Cloud Console](https://console.cloud.google.com/)
2. Go to **Compute Engine** → **VM instances**
3. Click **Create Instance**

**Configuration:**

```text
Name: openclaw-alobexpress
Region: us-central1 (or your preferred region)
Zone: us-central1-c
Machine type: e2-standard-2
Boot disk: Ubuntu 22.04 LTS, 50 GB SSD
```

4. Under **Firewall**, check:
   - ✅ Allow HTTP traffic
   - ✅ Allow HTTPS traffic

5. Click **Create**

**Firewall Rules:**

Ensure these ports are open in your VPC firewall:

| Port | Protocol | Purpose | Public? |
|------|----------|---------|---------|
| 22 | TCP | SSH access | Yes (restricted to your IP recommended) |
| 80 | TCP | HTTP (redirects to HTTPS) | Yes |
| 443 | TCP | HTTPS | Yes |
| 18789 | TCP | OpenClaw Gateway | **NO** (internal only) |

**Security Note**: Never expose port 18789 publicly. It should only be accessible via `127.0.0.1` (localhost).

#### Step 1.2: Reserve Static IP

A static IP prevents your DNS from breaking when the VM restarts.

**Via Console:**
1. Go to **VPC Network** → **IP addresses**
2. Click **Reserve Static Address**
3. Name: `openclaw-static-ip`
4. Attach to: `openclaw-alobexpress`
5. Click **Reserve**

**Via gcloud CLI:**
```bash
# Reserve static IP
gcloud compute addresses create openclaw-static-ip \
  --region=us-central1

# Attach to VM
gcloud compute instances delete-access-config openclaw-alobexpress \
  --access-config-name="External NAT" \
  --zone=us-central1-c

gcloud compute instances add-access-config openclaw-alobexpress \
  --access-config-name="External NAT" \
  --address=openclaw-static-ip \
  --zone=us-central1-c
```

**Get your static IP:**
```bash
gcloud compute addresses describe openclaw-static-ip \
  --region=us-central1 \
  --format="get(address)"
```

#### Step 1.3: Configure Cloudflare DNS

1. Log in to [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Select your domain (e.g., `alobexpress.com.br`)
3. Go to **DNS** → **Records**

**Add DNS Records:**

| Type | Name | Content | Proxy Status | TTL |
|------|------|---------|--------------|-----|
| A | advanced | `YOUR_VM_STATIC_IP` | Proxied (🟠) | Auto |
| CNAME | openclaw | advanced.alobexpress.com.br | Proxied (🟠) | Auto |

**Example:**
```text
A     advanced    34.123.45.67    Proxied    Auto
CNAME openclaw    advanced.alobexpress.com.br    Proxied    Auto
```

**Proxy Status**: The orange cloud (🟠 Proxied) means traffic routes through Cloudflare's CDN.

**SSL/TLS Configuration:**

1. Go to **SSL/TLS** → **Overview**
2. Set encryption mode to **Full**
   - ❌ Not `Flexible` (insecure)
   - ✅ Use `Full` or `Full (strict)`

**Why Full?**
- `Flexible`: Cloudflare ↔ User is HTTPS, but Cloudflare ↔ Server is HTTP (insecure)
- `Full`: HTTPS end-to-end (Caddy handles SSL on server side)

**DNS Propagation:**

After adding records, wait 5-10 minutes for DNS propagation. Verify with:

```bash
# Check DNS resolution
nslookup openclaw.alobexpress.com.br

# Or use dig
dig openclaw.alobexpress.com.br
```

**Note**: With Cloudflare proxy enabled, DNS tools will show Cloudflare IPs, not your VM IP. This is expected.

---

### 2. OpenClaw Installation

#### Step 2.1: Access Your VM

**Via Google Cloud Console:**
1. Go to **Compute Engine** → **VM instances**
2. Click **SSH** next to `openclaw-alobexpress`

**Via gcloud CLI (recommended):**

```bash
# Set your project
gcloud config set project YOUR_PROJECT_ID

# SSH into VM
gcloud compute ssh openclaw-alobexpress --zone=us-central1-c
```

**Via standard SSH:**
```bash
ssh -i ~/.ssh/google_compute_engine USERNAME@YOUR_VM_IP
```

#### Step 2.2: Prepare the Environment

Update system packages and install dependencies:

```bash
# Update package lists
sudo apt update

# Upgrade existing packages
sudo apt upgrade -y

# Install essential tools
sudo apt install -y \
  curl \
  ca-certificates \
  gnupg \
  git \
  build-essential \
  unzip \
  nano \
  htop
```

**What each package does:**
- `curl`: Download files and make HTTP requests
- `ca-certificates`: SSL certificate validation
- `gnupg`: GPG key management
- `git`: Version control (useful for future updates)
- `build-essential`: Compilers for native Node.js modules
- `unzip`: Extract compressed files
- `nano`: Text editor
- `htop`: Process monitoring

#### Step 2.3: Install Node.js

OpenClaw requires Node.js 22.14+ or 24.x.

**Install via NodeSource (recommended):**

```bash
# Add NodeSource repository for Node.js 24.x
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -

# Install Node.js and npm
sudo apt install -y nodejs

# Verify installation
node -v  # Should show v24.x.x
npm -v   # Should show 10.x.x or higher
```

**Expected output:**
```text
v24.0.0
10.2.3
```

**Alternative: Install via nvm (for multiple Node versions):**

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Load nvm
source ~/.bashrc

# Install Node.js 24
nvm install 24
nvm use 24
nvm alias default 24
```

#### Step 2.4: Install OpenClaw

**Method 1: Official Installer (recommended)**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

This script:
- Detects your OS and architecture
- Installs OpenClaw globally
- Sets up initial configuration
- Adds OpenClaw to your PATH

**Method 2: npm Global Install**

```bash
sudo npm install -g openclaw@latest
```

**Verify Installation:**

```bash
# Check version
openclaw --version

# Run diagnostics
openclaw doctor
```

**Expected output from `openclaw doctor`:**
```text
✓ Node.js version: v24.0.0
✓ npm version: 10.2.3
✓ OpenClaw version: 1.x.x
✓ Configuration directory: /home/username/.openclaw
✓ Gateway: Not running
```

#### Step 2.5: Initial Setup

Run the setup wizard:

```bash
openclaw setup --wizard
```

**Or use the interactive configure command:**

```bash
openclaw configure
```

**Configuration Prompts:**

| Prompt | Recommended Value | Notes |
|--------|-------------------|-------|
| **Gateway Mode** | Local | For single-server deployment |
| **AI Provider** | OpenAI | Or Claude, Gemini, etc. |
| **Model** | gpt-4o-mini | Cost-effective for testing |
| **Channels** | Telegram | Add others as needed |
| **Plugins** | Default | Customize later |

**Configuration File Location:**

```text
~/.openclaw/openclaw.json          # If running as regular user
/root/.openclaw/openclaw.json      # If running as root
```

**Important**: Always use the same user for setup and running OpenClaw. Mixing users causes permission issues.

**View Configuration:**

```bash
cat ~/.openclaw/openclaw.json
```

---

### 3. Gateway Configuration

#### Step 3.1: Configure Gateway Token

The gateway token authenticates dashboard access.

**Open configuration file:**

```bash
nano ~/.openclaw/openclaw.json
```

**Or if running as root:**

```bash
sudo nano /root/.openclaw/openclaw.json
```

**Find the gateway section:**

```json
{
  "gateway": {
    "auth": {
      "token": "your-secure-token-here"
    }
  }
}
```

**Important**: 
- This token is auto-generated during setup
- Use this exact token in the dashboard login
- Never share this token publicly
- Regenerate if compromised: `openclaw gateway --reset-token`

#### Step 3.2: Allow Public Domain Access

By default, OpenClaw blocks requests from unknown origins. Configure allowed domains:

**Edit configuration:**

```bash
nano ~/.openclaw/openclaw.json
```

**Add `controlUi` section:**

```json
{
  "gateway": {
    "auth": {
      "token": "your-secure-token-here"
    },
    "controlUi": {
      "allowInsecureAuth": true,
      "allowedOrigins": [
        "https://openclaw.alobexpress.com.br"
      ]
    }
  }
}
```

**Configuration Options:**

| Option | Type | Purpose |
|--------|------|---------|
| `allowInsecureAuth` | boolean | Allow token-based auth (set `true` for production with HTTPS) |
| `allowedOrigins` | array | Whitelist of domains that can access the gateway |

**Multiple Domains:**

```json
"allowedOrigins": [
  "https://openclaw.alobexpress.com.br",
  "https://openclaw-staging.alobexpress.com.br",
  "http://localhost:3000"
]
```

**Validate JSON Syntax:**

```bash
# Check for syntax errors
node -e "JSON.parse(require('fs').readFileSync(process.env.HOME + '/.openclaw/openclaw.json','utf8')); console.log('✓ JSON is valid')"
```

**If running as root:**

```bash
node -e "JSON.parse(require('fs').readFileSync('/root/.openclaw/openclaw.json','utf8')); console.log('✓ JSON is valid')"
```

**Common JSON Errors:**
- Missing comma between properties
- Trailing comma after last property
- Unmatched braces `{}`
- Unmatched brackets `[]`

#### Step 3.3: Install and Configure Caddy

Caddy is a modern web server with automatic HTTPS.

**Install Caddy:**

```bash
# Update package list
sudo apt update

# Install Caddy
sudo apt install -y caddy

# Verify installation
caddy version
```

**Configure Caddyfile:**

```bash
sudo nano /etc/caddy/Caddyfile
```

**Add this configuration:**

```caddy
openclaw.alobexpress.com.br {
    reverse_proxy 127.0.0.1:18789
}
```

**What this does:**
- Listens on ports 80 and 443
- Automatically obtains SSL certificate from Let's Encrypt
- Proxies all requests to OpenClaw gateway on localhost:18789
- Handles HTTP → HTTPS redirects

**Advanced Caddyfile (with logging and headers):**

```caddy
openclaw.alobexpress.com.br {
    reverse_proxy 127.0.0.1:18789 {
        header_up X-Real-IP {remote_host}
        header_up X-Forwarded-For {remote_host}
        header_up X-Forwarded-Proto {scheme}
    }
    
    log {
        output file /var/log/caddy/openclaw.log
        format json
    }
    
    encode gzip
}
```

**Restart Caddy:**

```bash
# Restart to apply changes
sudo systemctl restart caddy

# Check status
sudo systemctl status caddy

# Enable auto-start on boot
sudo systemctl enable caddy
```

**Expected output:**
```text
● caddy.service - Caddy
     Loaded: loaded (/lib/systemd/system/caddy.service; enabled)
     Active: active (running) since...
```

**View Caddy logs:**

```bash
sudo journalctl -u caddy -f
```

#### Step 3.4: Test the Gateway

Start OpenClaw gateway manually to verify configuration:

```bash
openclaw gateway --allow-unconfigured
```

**Expected output:**
```text
[INFO] OpenClaw Gateway starting...
[INFO] Gateway ready
[INFO] HTTP server listening on 127.0.0.1:18789
[INFO] Browser control listening
[INFO] Heartbeat started
```

**Test local access:**

```bash
# In another terminal
curl http://127.0.0.1:18789/health
```

**Test public access:**

Open your browser and navigate to:

```text
https://openclaw.alobexpress.com.br
```

**You should see:**
- OpenClaw dashboard login screen
- Prompt for gateway token

**Enter the token from `~/.openclaw/openclaw.json`**

**Stop the manual gateway:**

Press `Ctrl+C` in the terminal running `openclaw gateway`

#### Step 3.5: Device Pairing

OpenClaw uses device pairing for security.

**On first dashboard access, you'll see:**

```text
Device pairing required
Request ID: 1e0ea8bf-2b7d-41ac-aca4-42e64f78ec70
```

**In your VM terminal, approve the device:**

```bash
# List pending requests
openclaw devices list

# Approve the request
openclaw devices approve 1e0ea8bf-2b7d-41ac-aca4-42e64f78ec70
```

**Expected output:**
```text
✓ Device approved successfully
```

**Important Notes:**
- Request IDs expire after 5 minutes
- If expired, refresh the dashboard to generate a new request
- Approve immediately after seeing the request ID
- Each browser/device needs separate approval

**Manage devices:**

```bash
# List all approved devices
openclaw devices list --approved

# Revoke a device
openclaw devices revoke DEVICE_ID

# Clear all devices
openclaw devices clear
```

---

### 4. Process Management with PM2

PM2 keeps OpenClaw running 24/7 with automatic restarts.

#### Step 4.1: Install PM2

```bash
sudo npm install -g pm2
```

#### Step 4.2: Start OpenClaw with PM2

```bash
pm2 start "openclaw gateway --allow-unconfigured" --name openclaw-gateway
```

**Expected output:**
```text
[PM2] Starting openclaw gateway --allow-unconfigured in fork_mode (1 instance)
[PM2] Done.
┌─────┬──────────────────┬─────────┬─────────┬──────────┬────────┐
│ id  │ name             │ mode    │ ↺      │ status   │ cpu    │
├─────┼──────────────────┼─────────┼─────────┼──────────┼────────┤
│ 0   │ openclaw-gateway │ fork    │ 0       │ online   │ 0%     │
└─────┴──────────────────┴─────────┴─────────┴──────────┴────────┘
```

#### Step 4.3: Save PM2 Configuration

```bash
# Save current process list
pm2 save

# Generate startup script
pm2 startup
```

**PM2 will output a command like:**

```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u username --hp /home/username
```

**Copy and run that exact command.**

This ensures OpenClaw starts automatically after server reboots.

#### Step 4.4: PM2 Commands Reference

| Command | Purpose |
|---------|---------|
| `pm2 status` | Show all processes |
| `pm2 logs openclaw-gateway` | View real-time logs |
| `pm2 logs openclaw-gateway --lines 100` | View last 100 log lines |
| `pm2 restart openclaw-gateway` | Restart the gateway |
| `pm2 stop openclaw-gateway` | Stop the gateway |
| `pm2 delete openclaw-gateway` | Remove from PM2 |
| `pm2 monit` | Real-time monitoring dashboard |
| `pm2 flush` | Clear all logs |

**After configuration changes:**

```bash
# Restart to apply new config
pm2 restart openclaw-gateway

# View logs to verify
pm2 logs openclaw-gateway --lines 50
```

---

## 🔌 API Integrations

### 6.1. OpenAI API

Crie uma chave em:

```text
https://platform.openai.com/api-keys
```

Crie uma chave com nome tipo:

```text
openclaw-alobexpress
```

Depois configure no OpenClaw pelo wizard:

```bash
openclaw configure
```

ou usando variável de ambiente, conforme o método aceito pela sua configuração:

```bash
export OPENAI_API_KEY="sk-..."
```

Para persistir, o ideal é usar o próprio `openclaw configure` ou um arquivo de secrets do OpenClaw. Evite deixar chaves em README, GitHub ou prints.

#### Custos da OpenAI

- ChatGPT Business e OpenAI API têm billing separado.
- API é cobrada por uso.
- Monitore em: https://platform.openai.com/usage
- Configure limites em: https://platform.openai.com/settings/organization/limits

### 6.2. Whisper/Transcrição

Whisper/transcrição serve para transformar áudio em texto.

Útil para:

- Telegram com áudio.
- WhatsApp com áudio.
- Atendimento por voz.
- Resumo de reuniões.

No OpenClaw, configure a chave OpenAI e selecione o provider de transcrição quando solicitado.

Modelos comuns:

```text
whisper-1
gpt-4o-mini-transcribe
gpt-4o-transcribe
```

Para uso leve, normalmente o custo é baixo, mas monitore no painel da OpenAI.

### 6.3. Google Places API

A Google Places API serve para buscar empresas, lugares, endereços, telefones e leads locais.

Exemplo de uso:

```text
Encontrar clínicas odontológicas em Campinas com site fraco e Instagram parado.
```

Passos:

1. Acesse: https://console.cloud.google.com/apis/library
2. Pesquise por `Places API`.
3. Clique em `Ativar`.
4. Vá em: https://console.cloud.google.com/apis/credentials
5. Crie uma `Chave de API`.
6. Restrinja a chave:
   - API restrictions: somente `Places API`.
   - Application restrictions: IP da VM, se estiver usando chamadas server-side.

Na VM, descubra o IP externo:

```bash
curl ifconfig.me
```

Configure no OpenClaw com o nome que o wizard pedir, geralmente algo como:

```text
GOOGLE_PLACES_API_KEY
```

ou:

```text
GOOGLE_API_KEY
```

Importante: coloque budget/alertas na Google Cloud para evitar gasto inesperado.

### 6.4. Notion API

A API do Notion permite que o OpenClaw leia e escreva em páginas/databases.

Passos:

1. Acesse: https://www.notion.so/my-integrations
2. Clique em `New integration`.
3. Nomeie como `OpenClaw`.
4. Copie o `Internal Integration Token`, algo como `secret_...`.
5. No Notion, abra a página ou database.
6. Clique em `Share`.
7. Convide a integração `OpenClaw`.

Se o agente não encontrar a página, quase sempre é porque a página/database não foi compartilhada com a integração.

### 6.5. Telegram

1. No Telegram, fale com `@BotFather`.
2. Use:

```text
/newbot
```

3. Escolha nome e username.
4. Copie o token do bot.
5. Configure no OpenClaw pelo wizard:

```bash
openclaw configure
```

Depois reinicie o gateway:

```bash
pm2 restart openclaw-gateway
```

### 6.6. Firecrawl

O Firecrawl serve para:

- Buscar páginas.
- Raspar sites.
- Transformar páginas em markdown limpo.
- Alimentar agentes com conteúdo de sites.

Passos:

1. Crie conta em: https://www.firecrawl.dev/
2. Pegue a API key.
3. Configure no OpenClaw quando solicitado.

Use com cuidado. Crawling massivo pode consumir créditos rápido.

### 6.7. ElevenLabs

A ElevenLabs serve para voz IA:

- Responder em áudio.
- Criar narração.
- Gerar conteúdo para vídeos.
- Fazer agentes “falarem”.

Passos:

1. Crie conta em: https://elevenlabs.io/
2. Pegue a API key.
3. Configure no OpenClaw quando solicitado.

É opcional. Para o MVP inicial, OpenAI + Telegram + Places + Firecrawl já resolvem bastante coisa.

---

## 7. Testes e Validação

### Ver se OpenClaw está rodando

```bash
pm2 status
```

### Ver logs

```bash
pm2 logs openclaw-gateway
```

### Ver se a porta local está aberta

```bash
ss -lntp | grep 18789
```

Deve aparecer algo escutando em `127.0.0.1:18789`.

### Testar Caddy

```bash
sudo systemctl status caddy
```

### Testar URL pública

Abra:

```text
https://openclaw.alobexpress.com.br
```

---

## 8. Troubleshooting

### Erro: `Unable to locate package npm`

Rode:

```bash
sudo apt update
sudo apt install -y nodejs npm
```

Melhor ainda: instale Node via NodeSource:

```bash
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt install -y nodejs
```

---

### Erro: `HTTP ERROR 502`

Significa que Cloudflare/Caddy estão funcionando, mas o OpenClaw não está respondendo em `127.0.0.1:18789`.

Resolva:

```bash
pm2 status
pm2 logs openclaw-gateway
ss -lntp | grep 18789
pm2 restart openclaw-gateway
```

Se não usa PM2 ainda:

```bash
openclaw gateway --allow-unconfigured
```

---

### Erro: `origin not allowed`

Adicione o domínio em `allowedOrigins` no `openclaw.json`:

```json
"allowedOrigins": [
  "https://openclaw.alobexpress.com.br"
]
```

Reinicie:

```bash
pm2 restart openclaw-gateway
```

---

### Erro: `gateway token missing`

Você esqueceu de colocar o token na tela do dashboard.

Use o token que está em:

```text
~/.openclaw/openclaw.json
```

Campo correto:

```text
Token do Gateway
```

Deixe o campo senha vazio, a menos que tenha configurado modo senha.

---

### Erro: `device pairing required`

Aprove o dispositivo:

```bash
openclaw devices list
openclaw devices approve REQUEST_ID_AQUI
```

Se expirar, gere outro request clicando em conectar novamente.

---

### Erro: JSON inválido

Valide:

```bash
node -e "JSON.parse(require('fs').readFileSync(process.env.HOME + '/.openclaw/openclaw.json','utf8')); console.log('JSON OK')"
```

Se estiver como root:

```bash
node -e "JSON.parse(require('fs').readFileSync('/root/.openclaw/openclaw.json','utf8')); console.log('JSON OK')"
```

Procure vírgula faltando ou chave `{}` mal fechada.

---

### Erro do gcloud: `project is not currently set`

Configure o projeto:

```bash
gcloud projects list
gcloud config set project SEU_PROJECT_ID
```

---

### Erro do gcloud: `insufficient authentication scopes`

Refaça o login:

```bash
gcloud auth login
gcloud auth application-default login
```

Depois tente de novo.

---

## 9. Manutenção

### 9.1. Backup

Faça backup periódico destas pastas:

```text
~/.openclaw/openclaw.json
~/.openclaw/workspace
~/.openclaw/credentials
~/.openclaw/agents
```

Se usa root:

```text
/root/.openclaw/openclaw.json
/root/.openclaw/workspace
/root/.openclaw/credentials
/root/.openclaw/agents
```

Exemplo:

```bash
tar -czf openclaw-backup-$(date +%F).tar.gz ~/.openclaw
```

Atenção: esse backup pode conter tokens e credenciais. Não suba em repositório público.

### 9.2. Atualização

Se instalou via npm:

```bash
sudo npm install -g openclaw@latest
pm2 restart openclaw-gateway
openclaw doctor
```

Antes de atualizar, faça backup:

```bash
tar -czf openclaw-backup-before-update-$(date +%F).tar.gz ~/.openclaw
```

### 9.3. Checklist de Segurança

Antes de considerar produção, confira:

- [ ] Cloudflare com proxy laranja ativo.
- [ ] SSL/TLS em modo `Full`.
- [ ] Porta `18789` não exposta publicamente.
- [ ] Gateway token forte.
- [ ] Device pairing aprovado somente para seus dispositivos.
- [ ] OpenAI usage limit configurado.
- [ ] Google Cloud Budget Alerts configurados.
- [ ] Google Places API key restrita.
- [ ] Notion integration compartilhada só com páginas necessárias.
- [ ] Nenhum token salvo em GitHub público.
- [ ] Backup seguro da pasta `.openclaw`.

---

## 10. Referências

### Comandos Mais Usados

```bash
# Status do OpenClaw
pm2 status

# Logs do gateway
pm2 logs openclaw-gateway

# Reiniciar OpenClaw
pm2 restart openclaw-gateway

# Parar OpenClaw
pm2 stop openclaw-gateway

# Ver se porta está aberta
ss -lntp | grep 18789

# Abrir configuração
nano ~/.openclaw/openclaw.json

# Validar configuração
node -e "JSON.parse(require('fs').readFileSync(process.env.HOME + '/.openclaw/openclaw.json','utf8')); console.log('JSON OK')"

# Listar dispositivos pendentes
openclaw devices list

# Aprovar dispositivo
openclaw devices approve REQUEST_ID_AQUI

# Diagnóstico
openclaw doctor
```

### Fontes Oficiais

- Instalação OpenClaw: https://docs.openclaw.ai/install
- OpenClaw no GCP: https://docs.openclaw.ai/install/gcp
- Control UI: https://docs.openclaw.ai/web/control-ui
- Dashboard: https://docs.openclaw.ai/web/dashboard
- Remote access: https://docs.openclaw.ai/gateway/remote
- Devices CLI: https://docs.openclaw.ai/cli/devices
- Node.js para OpenClaw: https://docs.openclaw.ai/install/node
- OpenAI API usage: https://platform.openai.com/usage
- OpenAI billing: https://platform.openai.com/settings/organization/billing/overview

### Resumo da Arquitetura

O funcionamento ideal é:

```text
Cloudflare → Caddy → OpenClaw Gateway → Agentes/APIs
```

O que mantém tudo de pé:

```text
PM2
```

O que autentica o dashboard:

```text
Token do Gateway + device pairing
```

O que mais costuma dar erro:

```text
JSON inválido, origin not allowed, token errado, device pairing expirado ou gateway parado
```

Se aparecer `502`, quase sempre o gateway caiu ou não está ouvindo na porta `18789`.
