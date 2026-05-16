# Guia de Implantação em Produção do OpenClaw

![OpenClaw](https://img.shields.io/badge/OpenClaw-Produção-blue)
![Google Cloud](https://img.shields.io/badge/Google%20Cloud-GCP-4285F4?logo=google-cloud)
![Node.js](https://img.shields.io/badge/Node.js-24.x-339933?logo=node.js)
![Cloudflare](https://img.shields.io/badge/Cloudflare-CDN-F38020?logo=cloudflare)
![PM2](https://img.shields.io/badge/PM2-Gerenciador%20de%20Processos-2B037A)

Guia completo de implantação em produção da plataforma de agentes IA **OpenClaw** no Google Cloud Platform com infraestrutura de nível empresarial incluindo CDN Cloudflare, proxy reverso Caddy e gerenciamento de processos PM2.

> **Público-alvo**: Engenheiros DevOps e desenvolvedores implantando OpenClaw em produção. Todos os comandos podem ser copiados e colados. Substitua valores de exemplo como `SEU_TOKEN`, `SEU_DOMINIO` e `SUA_CHAVE` pelas suas credenciais reais.

> **Nota sobre idioma**: Este guia mantém comandos, termos técnicos e nomes de configuração em inglês (padrão da indústria), com descrições e explicações em português.

## 📚 Conjunto de Documentação

Este repositório contém documentação abrangente para implantar o OpenClaw:

- **[README.md](README.md)** (este arquivo) - Guia completo de implantação com ambos os métodos
- **[QUICK_START.md](QUICK_START.md)** - Guia de instalação rápida
- **[COMPARISON.md](COMPARISON.md)** - Comparação detalhada entre métodos de implantação  
- **[PRE_INSTALL_CHECKLIST.md](PRE_INSTALL_CHECKLIST.md)** - Checklist de preparação pré-instalação

**Novo no OpenClaw?** Comece com [PRE_INSTALL_CHECKLIST.md](PRE_INSTALL_CHECKLIST.md) → [COMPARISON.md](COMPARISON.md) → [QUICK_START.md](QUICK_START.md)

## 🎯 O Que Você Vai Construir

Seguindo este guia, você implantará uma instância OpenClaw pronta para produção com:

- ✅ **Acesso HTTPS seguro** via Cloudflare
- ✅ **Disponibilidade 24/7** com gerenciador de processos PM2  
- ✅ **Proxy reverso** com SSL automático via Caddy
- ✅ **Integrações IA**: OpenAI, Whisper, Google Places, Notion, Telegram, Firecrawl, ElevenLabs
- ✅ **Infraestrutura escalável** no Google Cloud Platform
- ✅ **Monitoramento e logs** de produção
- ✅ **Segurança de pareamento** de dispositivos
- ✅ **Reinicializações automáticas** em caso de falha


## 📑 Índice

- [Stack Tecnológica](#-stack-tecnológica)
- [Pré-requisitos](#-pré-requisitos)
- [Visão Geral da Arquitetura](#-visão-geral-da-arquitetura)
- [Considerações de Custo](#-considerações-de-custo)
- [Métodos de Implantação](#-métodos-de-implantação)
  - [Método 1: VM Tradicional com PM2](#método-1-vm-tradicional-com-pm2-recomendado-para-iniciantes)
  - [Método 2: Docker Swarm com Traefik](#método-2-docker-swarm-com-traefik-recomendado-para-produção)
- [Integrações de API](#-integrações-de-api)
- [Testes e Validação](#-testes-e-validação)
- [Troubleshooting](#-troubleshooting)
- [Manutenção](#-manutenção)
- [Checklist de Segurança](#-checklist-de-segurança)
- [Referência](#-referência)

---

## 🛠 Stack Tecnológica

### Infraestrutura Principal

| Componente | Tecnologia | Propósito |
|-----------|-----------|---------|
| **Plataforma** | Google Cloud Platform | Hospedagem de VM e infraestrutura |
| **SO** | Ubuntu 22.04/24.04 LTS | Sistema operacional do servidor |
| **Runtime** | Node.js 24.x | Runtime JavaScript para OpenClaw |
| **Plataforma IA** | OpenClaw | Orquestração de agentes IA |

### Opções de Implantação

#### Opção 1: Configuração Tradicional de VM

| Componente | Tecnologia | Propósito |
|-----------|-----------|---------|
| **Proxy Reverso** | Caddy 2.x | HTTPS automático e roteamento |
| **CDN/Segurança** | Cloudflare | Proteção DDoS, SSL, cache |
| **Gerenciador de Processos** | PM2 | Disponibilidade 24/7 e monitoramento |

#### Opção 2: Configuração Docker Swarm

| Componente | Tecnologia | Propósito |
|-----------|-----------|---------|
| **Runtime de Contêiner** | Docker Engine 24.x | Orquestração de contêineres |
| **Orquestração** | Docker Swarm | Gerenciamento e escalabilidade de serviços |
| **Proxy Reverso** | Traefik 2.x/3.x | Roteamento dinâmico e SSL |
| **UI de Gerenciamento** | Portainer | Gerenciamento visual de contêineres |

### Integrações IA

| Serviço | Propósito |
|---------|---------|
| **OpenAI** | Modelos GPT, embeddings |
| **Whisper** | Transcrição de áudio |
| **Google Places** | Buscas baseadas em localização |
| **Notion** | Operações de banco de dados |
| **Telegram** | Integração de chatbot |
| **Firecrawl** | Web scraping |
| **ElevenLabs** | Síntese de voz |

---

## 📋 Prerequisites

### Common Requirements (Both Methods)

- [ ] **Google Cloud Account** with Compute Engine enabled
- [ ] **Domain or subdomain** (e.g., `openclaw.alobexpress.com.br`)
- [ ] **OpenAI API Key** for AI model access
- [ ] **SSH access** to your local terminal

### Method 1: Traditional VM Requirements

- [ ] **Cloudflare Account** managing your domain's DNS
- [ ] Basic Linux command line knowledge

### Method 2: Docker Swarm Requirements

- [ ] **Docker Engine** 24.x or higher
- [ ] **Docker Swarm** initialized
- [ ] **Traefik** reverse proxy configured
- [ ] **Portainer** (optional, for visual management)
- [ ] External network `network_swarm_public` created

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

## 🚀 Deployment Methods

Choose the deployment method that best fits your needs:

### Quick Comparison

| Aspect | Traditional VM + PM2 | Docker Swarm + Traefik |
|--------|---------------------|------------------------|
| **Setup Time** | 30-45 minutes | 15-20 minutes (if Swarm ready) |
| **Complexity** | Low | Medium |
| **Best For** | Beginners, small teams, learning | Production, DevOps teams, scaling |
| **Scalability** | Manual (upgrade VM) | Automatic (add nodes) |
| **Management** | Command line (SSH) | Portainer UI + CLI |
| **Updates** | `npm install -g openclaw@latest` | `docker service update --image` |
| **Rollback** | Manual backup restore | Built-in (`docker service rollback`) |
| **Resource Usage** | Lower overhead | Slightly higher (Docker layer) |
| **Monitoring** | PM2 monit | Docker stats + Portainer |
| **Multi-instance** | Requires manual setup | Native support |
| **SSL Management** | Caddy (automatic) | Traefik (automatic) |
| **Cost** | VM only | VM + potential orchestration overhead |

### Decision Guide

**Choose Traditional VM + PM2 if:**
- ✅ You're new to OpenClaw or Docker
- ✅ You want simple, straightforward setup
- ✅ You're running a single instance
- ✅ You prefer direct file system access
- ✅ You want lower resource overhead

**Choose Docker Swarm + Traefik if:**
- ✅ You already have Docker infrastructure
- ✅ You need easy scaling and high availability
- ✅ You want visual management (Portainer)
- ✅ You're comfortable with containers
- ✅ You need quick rollbacks and updates
- ✅ You're running multiple services with Traefik

---

## Method 1: Traditional VM with PM2 (Recommended for Beginners)

This method uses a traditional VM setup with PM2 for process management and Caddy for reverse proxy.

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

## Method 2: Docker Swarm with Traefik (Recommended for Production)

This method uses Docker Swarm for orchestration and Traefik for automatic SSL and routing.

### Prerequisites for Docker Method

Ensure you have:

| Requirement | Minimum Version |
|-------------|-----------------|
| Docker Engine | 24.x |
| Docker Compose / Swarm | v2.x |
| Traefik (reverse proxy) | v2.x or v3.x |
| Portainer (optional) | Latest |

**Before starting:**

```bash
# Initialize Docker Swarm (if not already done)
docker swarm init

# Create external network
docker network create --driver overlay --attachable network_swarm_public

# Verify Traefik is running
docker service ls | grep traefik
```

### Step 1: Prepare Data Directory

```bash
# Create directory for OpenClaw data
mkdir -p /opt/infra/alobexpress/openclaw

# Set correct permissions (uid 1000 = node user in container)
chown -R 1000:1000 /opt/infra/alobexpress/openclaw
```

**Why uid 1000?** The official OpenClaw Docker image runs as the `node` user (uid 1000). Without correct permissions, the container cannot write configuration files.

### Step 2: Create docker-compose.yml

Create `/opt/infra/alobexpress/docker-compose.yml`:

```yaml
version: "3.7"

services:
  openclaw_gateway:
    image: ghcr.io/openclaw/openclaw:2026.5.7
    hostname: "{{.Service.Name}}.{{.Task.Slot}}"

    environment:
      NODE_ENV: production
      OPENCLAW_CONFIG: /home/node/.openclaw/openclaw.json
      OPENCLAW_DISABLE_BONJOUR: "1"
      OPENCLAW_GATEWAY_BIND: lan

      # Configure these in Portainer Environment Variables
      OPENCLAW_GATEWAY_TOKEN: ${OPENCLAW_GATEWAY_TOKEN}
      OPENCLAW_HOOKS_TOKEN: ${OPENCLAW_HOOKS_TOKEN}
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      FIRECRAWL_API_KEY: ${FIRECRAWL_API_KEY:-}
      GOPLACES_API_KEY: ${GOPLACES_API_KEY:-}
      NOTION_API_KEY: ${NOTION_API_KEY:-}
      SUPABASE_ACCESS_TOKEN: ${SUPABASE_ACCESS_TOKEN:-}
      FIGMA_API_KEY: ${FIGMA_API_KEY:-}
      GITHUB_PERSONAL_ACCESS_TOKEN: ${GITHUB_PERSONAL_ACCESS_TOKEN:-}
      N8N_BEARER_TOKEN: ${N8N_BEARER_TOKEN:-}
      TELEGRAM_DEFAULT_BOT_TOKEN: ${TELEGRAM_DEFAULT_BOT_TOKEN}
      TELEGRAM_PABLIN_BOT_TOKEN: ${TELEGRAM_PABLIN_BOT_TOKEN:-}
      TELEGRAM_MARCOS_BOT_TOKEN: ${TELEGRAM_MARCOS_BOT_TOKEN:-}

    volumes:
      - /opt/infra/alobexpress/openclaw:/home/node/.openclaw

    tmpfs:
      - /tmp:size=1g

    networks:
      - network_swarm_public

    deploy:
      mode: replicated
      replicas: 1

      placement:
        constraints:
          - node.role == manager

      resources:
        reservations:
          cpus: "0.25"
          memory: 512M
        limits:
          cpus: "2.0"
          memory: 6144M

      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 5
        window: 120s

      update_config:
        parallelism: 1
        delay: 30s
        order: stop-first
        failure_action: rollback

      labels:
        traefik.enable: "true"
        traefik.swarm.network: "network_swarm_public"
        traefik.http.routers.openclaw.rule: "Host(`openclaw.seudominio.com.br`)"
        traefik.http.routers.openclaw.entrypoints: "websecure"
        traefik.http.routers.openclaw.priority: "1"
        traefik.http.routers.openclaw.tls.certresolver: "letsencryptresolver"
        traefik.http.routers.openclaw.service: "openclaw_gateway"
        traefik.http.services.openclaw_gateway.loadbalancer.server.port: "18789"
        traefik.http.services.openclaw_gateway.loadbalancer.passHostHeader: "true"

    healthcheck:
      test: ["CMD-SHELL", "curl -fsS http://127.0.0.1:18789/healthz >/dev/null || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 120s

networks:
  network_swarm_public:
    external: true
    name: network_swarm_public
```

**Important Configuration Notes:**

| Setting | Value | Why |
|---------|-------|-----|
| `image` | `ghcr.io/openclaw/openclaw:2026.5.7` | Official pre-built image (don't use `node:22-bookworm`) |
| `OPENCLAW_GATEWAY_BIND` | `lan` | Allows Traefik to connect |
| `OPENCLAW_DISABLE_BONJOUR` | `1` | Disables mDNS (not needed in containers) |
| `volumes` | `/opt/infra/alobexpress/openclaw` | Persistent data storage |
| `tmpfs` | `/tmp:size=1g` | Temporary files in memory |
| `healthcheck` | `/healthz` endpoint | Docker monitors service health |

### Step 3: Configure Environment Variables

**Option A: Using Portainer (Recommended)**

1. Open Portainer
2. Go to **Stacks** → **Add Stack**
3. Name: `openclaw`
4. Paste the docker-compose.yml content
5. Scroll to **Environment Variables**
6. Add these variables:

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENCLAW_GATEWAY_TOKEN` | Yes | Gateway authentication token |
| `OPENCLAW_HOOKS_TOKEN` | Yes | Webhook authentication token |
| `OPENAI_API_KEY` | Yes | OpenAI API key |
| `TELEGRAM_DEFAULT_BOT_TOKEN` | If using Telegram | Main bot token |
| `NOTION_API_KEY` | Optional | Notion integration |
| `FIRECRAWL_API_KEY` | Optional | Web scraping |
| `GITHUB_PERSONAL_ACCESS_TOKEN` | Optional | GitHub integration |

**Option B: Using .env File**

Create `/opt/infra/alobexpress/.env`:

```bash
OPENCLAW_GATEWAY_TOKEN=your-secure-token-here
OPENCLAW_HOOKS_TOKEN=your-hooks-token-here
OPENAI_API_KEY=sk-...
TELEGRAM_DEFAULT_BOT_TOKEN=123456:ABC...
```

### Step 4: Deploy the Stack

**Via Portainer:**

Click **Deploy the stack** button.

**Via CLI:**

```bash
cd /opt/infra/alobexpress
docker stack deploy -c docker-compose.yml openclaw
```

### Step 5: Verify Deployment

```bash
# Check service status
docker service ls | grep openclaw

# View logs
docker service logs -f openclaw_openclaw_gateway

# Check container health
docker ps --filter name=openclaw
```

**Healthy logs should show:**

```text
[gateway] ready
[heartbeat] started
[http] server listening on 0.0.0.0:18789
```

### Step 6: Configure OpenClaw CLI Alias

The `openclaw` CLI only exists inside the container. Create an alias for easy access:

```bash
# Add to ~/.bashrc
echo 'alias openclaw="docker exec -it \$(docker ps --filter name=openclaw -q) node dist/index.js"' >> ~/.bashrc

# Reload shell
source ~/.bashrc

# Test
openclaw --version
```

### Step 7: Configure CORS and Trusted Proxies

Allow dashboard access from your domain:

```bash
# Get Traefik internal IP from logs
docker service logs openclaw_openclaw_gateway | grep "peer="
# Example output: peer=10.0.1.3:54321

# Configure allowed origins and trusted proxies
openclaw config set \
  --batch-json '[
    {"path":"gateway.controlUi.allowedOrigins","value":["http://localhost:18789","http://127.0.0.1:18789","https://openclaw.seudominio.com.br"]},
    {"path":"gateway.trustedProxies","value":["10.0.1.3"]}
  ]'
```

**Replace `10.0.1.3` with your actual Traefik IP.**

### Step 8: Configure Gateway Mode and API Keys

```bash
# Set gateway mode
openclaw config set gateway.mode local

# Configure OpenAI API key (if not using env var)
openclaw config set agents.main.auth.openai.apiKey sk-...

# Set command owner (your Telegram ID)
openclaw config set commands.ownerAllowFrom '["telegram:123456789"]'
```

### Step 9: Clean Up Inapplicable Skills

Remove skills that require binaries not available in containers:

```bash
openclaw doctor --fix
```

### Step 10: First Dashboard Access

1. Navigate to `https://openclaw.seudominio.com.br`
2. Confirm WebSocket URL: `wss://openclaw.seudominio.com.br`
3. Enter gateway token:

```bash
# View token
docker exec -it $(docker ps --filter name=openclaw -q) printenv OPENCLAW_GATEWAY_TOKEN

# Or from config
openclaw config get gateway.auth.token
```

4. Click **Connect**
5. If device pairing is required:

```bash
# Approve device
openclaw devices approve YOUR_REQUEST_ID
```

### Docker-Specific Commands

```bash
# View logs
docker service logs -f openclaw_openclaw_gateway

# Enter container shell
docker exec -it $(docker ps --filter name=openclaw -q) bash

# Force redeploy (after config changes)
docker service update --force openclaw_openclaw_gateway

# Scale service
docker service scale openclaw_openclaw_gateway=2

# View service details
docker service inspect openclaw_openclaw_gateway

# Remove stack
docker stack rm openclaw
```

### Docker File Structure

```text
/opt/infra/alobexpress/openclaw/
├── openclaw.json              # Main config (auto-managed)
├── openclaw.json.bak          # Backup before each change
├── openclaw.json.last-good    # Last known good config
├── agents/
│   └── main/
│       └── agent/
│           └── auth-profiles.json
├── canvas/
├── credentials/               # OAuth tokens
└── logs/
    └── stability/
```

### Docker Troubleshooting

#### ❌ `bash: openclaw: command not found` (exit 127)

**Cause:** Using wrong base image or missing CLI alias.

**Solution:**
```bash
# Ensure using official image
image: ghcr.io/openclaw/openclaw:2026.5.7

# Create alias
echo 'alias openclaw="docker exec -it \$(docker ps --filter name=openclaw -q) node dist/index.js"' >> ~/.bashrc
source ~/.bashrc
```

#### ❌ `Proxy headers detected from untrusted address`

**Cause:** Traefik IP not in trusted proxies list.

**Solution:**
```bash
# Find Traefik IP in logs
docker service logs openclaw_openclaw_gateway | grep "peer="

# Add to trusted proxies
openclaw config set --batch-json '[{"path":"gateway.trustedProxies","value":["TRAEFIK_IP"]}]'
```

#### ❌ `JSON5: invalid character` — gateway won't start

**Cause:** Manual edit introduced invalid JSON (typographic quotes).

**Solution:**
```bash
# Restore backup
cp /opt/infra/alobexpress/openclaw/openclaw.json.last-good \
   /opt/infra/alobexpress/openclaw/openclaw.json

# Or run doctor
openclaw doctor --fix

# Restart service
docker service update --force openclaw_openclaw_gateway
```

**Never edit `openclaw.json` manually. Always use `openclaw config set`.**

#### ❌ `404 Not Found` at domain

**Cause:** Volume path mismatch or permission issues.

**Solution:**
```bash
# Verify path exists
ls -la /opt/infra/alobexpress/openclaw

# Fix permissions
chown -R 1000:1000 /opt/infra/alobexpress/openclaw

# Restart service
docker service update --force openclaw_openclaw_gateway
```

#### ❌ `gateway.mode is unset`

**Cause:** Gateway mode not configured.

**Solution:**
```bash
openclaw config set gateway.mode local
docker service update --force openclaw_openclaw_gateway
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

## 🔧 Troubleshooting

This section covers common issues for both deployment methods.

### Common Issues (Both Methods)

#### ❌ `origin not allowed` on WebSocket

**Cause:** Gateway blocks connections from unknown origins.

**Solution for PM2:**
```bash
nano ~/.openclaw/openclaw.json
```

Add to `gateway.controlUi`:
```json
"allowedOrigins": ["https://openclaw.seudominio.com.br"]
```

Restart:
```bash
pm2 restart openclaw-gateway
```

**Solution for Docker:**
```bash
openclaw config set \
  --batch-json '[{"path":"gateway.controlUi.allowedOrigins","value":["https://openclaw.seudominio.com.br"]}]'

docker service update --force openclaw_openclaw_gateway
```

---

#### ❌ `No API key found for provider "openai"`

**Cause:** OpenAI API key not configured.

**Solution for PM2:**
```bash
openclaw configure
# Or
export OPENAI_API_KEY="sk-..."
```

**Solution for Docker:**
```bash
# Check if env var is set
docker exec -it $(docker ps --filter name=openclaw -q) printenv OPENAI_API_KEY

# If empty, set in config
openclaw config set agents.main.auth.openai.apiKey sk-...

# Or update Portainer environment variables and redeploy
```

---

#### ❌ `device pairing required (requestId: ...)`

**Cause:** New browser/device needs approval.

**Solution (Both Methods):**
```bash
# List pending requests
openclaw devices list

# Approve device
openclaw devices approve REQUEST_ID_HERE
```

**Note:** Request IDs expire after 5 minutes. If expired, refresh the dashboard to generate a new one.

---

#### ❌ `gateway.mode is unset; gateway start will be blocked`

**Cause:** Gateway mode not configured.

**Solution (Both Methods):**
```bash
openclaw config set gateway.mode local
```

Then restart the service.

---

### Method 1 Specific Issues (PM2)

#### ❌ `Unable to locate package npm`

**Cause:** Node.js not installed or outdated repository.

**Solution:**
```bash
# Install via NodeSource
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node -v
npm -v
```

---

#### ❌ `HTTP ERROR 502`

**Cause:** Caddy is running but OpenClaw gateway is not responding.

**Solution:**
```bash
# Check PM2 status
pm2 status

# View logs
pm2 logs openclaw-gateway

# Check if port is listening
ss -lntp | grep 18789

# Restart if needed
pm2 restart openclaw-gateway
```

---

#### ❌ `gateway token missing`

**Cause:** Token not entered in dashboard.

**Solution:**
```bash
# View token
cat ~/.openclaw/openclaw.json | grep token

# Or
openclaw config get gateway.auth.token
```

Enter this token in the dashboard "Token do Gateway" field.

---

#### ❌ JSON inválido

**Cause:** Manual edit introduced syntax errors.

**Solution:**
```bash
# Validate JSON
node -e "JSON.parse(require('fs').readFileSync(process.env.HOME + '/.openclaw/openclaw.json','utf8')); console.log('✓ JSON is valid')"

# If invalid, restore backup
cp ~/.openclaw/openclaw.json.bak ~/.openclaw/openclaw.json

# Restart
pm2 restart openclaw-gateway
```

---

#### ❌ gcloud: `project is not currently set`

**Cause:** Google Cloud project not configured.

**Solution:**
```bash
# List projects
gcloud projects list

# Set project
gcloud config set project YOUR_PROJECT_ID
```

---

#### ❌ gcloud: `insufficient authentication scopes`

**Cause:** Authentication expired or insufficient permissions.

**Solution:**
```bash
gcloud auth login
gcloud auth application-default login
```

---

### Method 2 Specific Issues (Docker)

#### ❌ `bash: openclaw: command not found` (exit 127)

**Cause:** CLI alias not configured or using wrong image.

**Solution:**
```bash
# Verify using official image
docker service inspect openclaw_openclaw_gateway | grep Image
# Should show: ghcr.io/openclaw/openclaw:2026.5.7

# Create alias
echo 'alias openclaw="docker exec -it \$(docker ps --filter name=openclaw -q) node dist/index.js"' >> ~/.bashrc
source ~/.bashrc

# Test
openclaw --version
```

---

#### ❌ `Proxy headers detected from untrusted address`

**Cause:** Traefik IP not in trusted proxies list.

**Solution:**
```bash
# Find Traefik IP in logs
docker service logs openclaw_openclaw_gateway | grep "peer="
# Example: peer=10.0.1.3:54321

# Add to trusted proxies
openclaw config set \
  --batch-json '[{"path":"gateway.trustedProxies","value":["10.0.1.3"]}]'

# Restart
docker service update --force openclaw_openclaw_gateway
```

---

#### ❌ `JSON5: invalid character '"' at 11:2`

**Cause:** Manual edit introduced typographic quotes or invalid JSON.

**Solution:**
```bash
# Restore last good backup
cp /opt/infra/alobexpress/openclaw/openclaw.json.last-good \
   /opt/infra/alobexpress/openclaw/openclaw.json

# Or run doctor
openclaw doctor --fix

# Restart service
docker service update --force openclaw_openclaw_gateway
```

**Important:** Never edit `openclaw.json` manually. Always use `openclaw config set`.

---

#### ❌ `404 Not Found` at domain

**Cause:** Volume path mismatch or permission issues.

**Solution:**
```bash
# Verify path exists
ls -la /opt/infra/alobexpress/openclaw

# Fix permissions (uid 1000 = node user)
chown -R 1000:1000 /opt/infra/alobexpress/openclaw

# Verify volume in service
docker service inspect openclaw_openclaw_gateway | grep -A5 Mounts

# Restart service
docker service update --force openclaw_openclaw_gateway
```

---

#### ❌ Container keeps restarting

**Cause:** Configuration error or missing required environment variables.

**Solution:**
```bash
# View logs
docker service logs openclaw_openclaw_gateway --tail 100

# Check environment variables
docker exec -it $(docker ps --filter name=openclaw -q) env | grep OPENCLAW

# Verify required vars are set
docker exec -it $(docker ps --filter name=openclaw -q) printenv OPENAI_API_KEY
docker exec -it $(docker ps --filter name=openclaw -q) printenv OPENCLAW_GATEWAY_TOKEN

# Update environment variables in Portainer and redeploy
```

---

#### ❌ Healthcheck failing

**Cause:** Gateway not responding on port 18789.

**Solution:**
```bash
# Check if port is listening inside container
docker exec -it $(docker ps --filter name=openclaw -q) curl http://127.0.0.1:18789/healthz

# View detailed logs
docker service logs openclaw_openclaw_gateway --tail 200

# Check service health
docker service ps openclaw_openclaw_gateway

# If needed, increase start_period in healthcheck
# Edit docker-compose.yml:
# start_period: 180s  # Give more time for startup
```

---

### Diagnostic Commands

**For PM2 Method:**
```bash
# Full system check
openclaw doctor

# View all config
openclaw config get

# Check PM2 status
pm2 status
pm2 monit

# View logs
pm2 logs openclaw-gateway --lines 100

# Check port
ss -lntp | grep 18789

# Test local access
curl http://127.0.0.1:18789/healthz

# Check Caddy
sudo systemctl status caddy
sudo journalctl -u caddy -f
```

**For Docker Method:**
```bash
# Full system check
openclaw doctor

# View all config
openclaw config get

# Check service status
docker service ls | grep openclaw
docker service ps openclaw_openclaw_gateway

# View logs
docker service logs -f openclaw_openclaw_gateway

# Check container health
docker ps --filter name=openclaw

# Enter container
docker exec -it $(docker ps --filter name=openclaw -q) bash

# Test inside container
docker exec -it $(docker ps --filter name=openclaw -q) curl http://127.0.0.1:18789/healthz

# Check Traefik
docker service logs traefik | grep openclaw

# View service details
docker service inspect openclaw_openclaw_gateway
```

---

## 9. Maintenance

### 9.1. Backup

**What to Backup:**

| Path | Contains | Critical |
|------|----------|----------|
| `~/.openclaw/openclaw.json` | Main configuration | Yes |
| `~/.openclaw/workspace` | Agent workspaces | Yes |
| `~/.openclaw/credentials` | OAuth tokens | Yes |
| `~/.openclaw/agents` | Agent configurations | Yes |
| `~/.openclaw/logs` | Application logs | No |

**For PM2 Method:**

```bash
# Create backup
tar -czf openclaw-backup-$(date +%F).tar.gz ~/.openclaw

# If running as root
tar -czf openclaw-backup-$(date +%F).tar.gz /root/.openclaw

# Store backup securely (contains secrets!)
mv openclaw-backup-*.tar.gz /secure/backup/location/
```

**For Docker Method:**

```bash
# Backup from host (data is in volume)
tar -czf openclaw-backup-$(date +%F).tar.gz /opt/infra/alobexpress/openclaw

# Or backup from container
docker exec $(docker ps --filter name=openclaw -q) tar -czf /tmp/backup.tar.gz /home/node/.openclaw
docker cp $(docker ps --filter name=openclaw -q):/tmp/backup.tar.gz ./openclaw-backup-$(date +%F).tar.gz

# Store backup securely
mv openclaw-backup-*.tar.gz /secure/backup/location/
```

**Automated Backup Script:**

```bash
#!/bin/bash
# /usr/local/bin/backup-openclaw.sh

BACKUP_DIR="/secure/backups/openclaw"
DATE=$(date +%F)
RETENTION_DAYS=30

# Create backup
tar -czf "$BACKUP_DIR/openclaw-$DATE.tar.gz" ~/.openclaw

# Remove old backups
find "$BACKUP_DIR" -name "openclaw-*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: openclaw-$DATE.tar.gz"
```

**Schedule with cron:**

```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /usr/local/bin/backup-openclaw.sh
```

**⚠️ Security Warning:** Backups contain API keys, tokens, and credentials. Never commit to public repositories or store in unsecured locations.

### 9.2. Updates

#### Updating PM2 Method

```bash
# Backup first!
tar -czf openclaw-backup-before-update-$(date +%F).tar.gz ~/.openclaw

# Update OpenClaw
sudo npm install -g openclaw@latest

# Restart gateway
pm2 restart openclaw-gateway

# Verify
openclaw --version
openclaw doctor

# Check logs
pm2 logs openclaw-gateway --lines 50
```

#### Updating Docker Method

```bash
# Backup first!
tar -czf openclaw-backup-before-update-$(date +%F).tar.gz /opt/infra/alobexpress/openclaw

# Update to specific version
docker service update --image ghcr.io/openclaw/openclaw:2026.5.8 openclaw_openclaw_gateway

# Or update to latest
docker service update --image ghcr.io/openclaw/openclaw:latest openclaw_openclaw_gateway

# Monitor update
docker service ps openclaw_openclaw_gateway

# Check logs
docker service logs -f openclaw_openclaw_gateway

# Verify
openclaw --version
openclaw doctor
```

**Rollback if needed:**

```bash
# Docker has built-in rollback
docker service rollback openclaw_openclaw_gateway

# Or specify previous version
docker service update --image ghcr.io/openclaw/openclaw:2026.5.7 openclaw_openclaw_gateway
```

### 9.3. Monitoring

#### PM2 Monitoring

```bash
# Real-time monitoring
pm2 monit

# Process status
pm2 status

# View logs
pm2 logs openclaw-gateway

# CPU and memory usage
pm2 describe openclaw-gateway

# Web dashboard (optional)
pm2 web
# Access at http://localhost:9615
```

#### Docker Monitoring

```bash
# Service status
docker service ps openclaw_openclaw_gateway

# Resource usage
docker stats $(docker ps --filter name=openclaw -q)

# Logs
docker service logs -f openclaw_openclaw_gateway

# Health status
docker ps --filter name=openclaw --format "table {{.Names}}\t{{.Status}}"

# Portainer dashboard
# Access at https://portainer.yourdomain.com
```

### 9.4. Log Management

#### PM2 Logs

```bash
# View logs
pm2 logs openclaw-gateway

# Clear logs
pm2 flush

# Rotate logs (configure in ecosystem.config.js)
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
```

#### Docker Logs

```bash
# View logs
docker service logs openclaw_openclaw_gateway --tail 100

# Follow logs
docker service logs -f openclaw_openclaw_gateway

# Logs are automatically rotated by Docker
# Configure in /etc/docker/daemon.json:
```

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

### 9.5. Performance Tuning

#### VM Resources (Both Methods)

**Monitor resource usage:**

```bash
# CPU and memory
htop

# Disk usage
df -h
du -sh ~/.openclaw/*

# Network
netstat -tulpn | grep 18789
```

**When to scale up:**

| Metric | Threshold | Action |
|--------|-----------|--------|
| CPU usage | >80% sustained | Upgrade to e2-standard-4 |
| Memory usage | >90% | Add more RAM |
| Disk usage | >80% | Increase disk size |
| Response time | >2s | Check logs, optimize agents |

#### Docker Resource Limits

Edit `docker-compose.yml`:

```yaml
resources:
  reservations:
    cpus: "0.5"      # Increase if needed
    memory: 1024M    # Increase if needed
  limits:
    cpus: "4.0"      # Maximum CPUs
    memory: 8192M    # Maximum memory
```

Redeploy:

```bash
docker stack deploy -c docker-compose.yml openclaw
```

### 9.6. Health Checks

#### Manual Health Check

```bash
# Check gateway health
curl http://localhost:18789/healthz

# Expected response: 200 OK
```

#### Automated Monitoring

**For PM2 (using cron):**

```bash
#!/bin/bash
# /usr/local/bin/check-openclaw-health.sh

if ! curl -sf http://localhost:18789/healthz > /dev/null; then
    echo "OpenClaw health check failed, restarting..."
    pm2 restart openclaw-gateway
    # Send alert (email, Slack, etc.)
fi
```

**For Docker (built-in):**

Healthcheck is already configured in docker-compose.yml. Docker automatically restarts unhealthy containers.

### 9.7. Security Updates

```bash
# Update system packages (both methods)
sudo apt update
sudo apt upgrade -y

# Update Node.js (PM2 method)
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt install -y nodejs

# Update Docker (Docker method)
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Restart services after updates
# PM2:
pm2 restart openclaw-gateway

# Docker:
docker service update --force openclaw_openclaw_gateway
```

---

## 10. Security Checklist

---

## 10. Security Checklist

Before going to production, verify all security measures are in place:

### Infrastructure Security

#### Common (Both Methods)

- [ ] **Firewall configured** - Only ports 22, 80, 443 open publicly
- [ ] **Port 18789 NOT exposed** - Gateway only accessible via reverse proxy
- [ ] **Static IP reserved** - Prevents DNS breakage on VM restart
- [ ] **SSH key authentication** - Password authentication disabled
- [ ] **Strong gateway token** - Minimum 32 characters, randomly generated
- [ ] **Device pairing enabled** - Only approved devices can connect
- [ ] **HTTPS enforced** - All traffic encrypted (Cloudflare/Traefik)
- [ ] **Regular backups** - Automated daily backups to secure location
- [ ] **Backup encryption** - Backups contain secrets, must be encrypted
- [ ] **System updates** - OS and packages regularly updated

#### PM2 Method Specific

- [ ] **Cloudflare proxy active** - Orange cloud enabled for DDoS protection
- [ ] **SSL/TLS mode: Full** - Not Flexible (insecure)
- [ ] **Caddy auto-updates** - SSL certificates renew automatically
- [ ] **PM2 startup configured** - Gateway starts on boot
- [ ] **Log rotation enabled** - Prevents disk space issues

#### Docker Method Specific

- [ ] **Traefik SSL configured** - Let's Encrypt certificates working
- [ ] **Trusted proxies set** - Traefik IP in gateway config
- [ ] **Container user** - Running as non-root (uid 1000)
- [ ] **Volume permissions** - Correct ownership (1000:1000)
- [ ] **Resource limits** - CPU and memory limits configured
- [ ] **Health checks** - Container health monitoring active
- [ ] **Secrets management** - API keys in environment variables, not in image
- [ ] **Image pinned** - Using specific version tag, not `latest`

### API Security

- [ ] **OpenAI usage limits** - Monthly spending cap configured
- [ ] **OpenAI key restricted** - Not shared publicly or in git
- [ ] **Google Places key restricted** - IP and API restrictions applied
- [ ] **Notion integration scoped** - Only shared with required pages
- [ ] **Telegram bot token secure** - Not exposed in logs or errors
- [ ] **GitHub token scoped** - Minimum required permissions
- [ ] **All API keys rotated** - Regular rotation schedule (90 days)

### Application Security

- [ ] **Allowed origins configured** - Only your domains whitelisted
- [ ] **Command owner set** - Privileged commands restricted to owner
- [ ] **No secrets in config** - API keys in env vars or secure storage
- [ ] **Config validation** - `openclaw doctor` passes without errors
- [ ] **Audit logs enabled** - Track all administrative actions
- [ ] **Rate limiting** - Prevent API abuse

### Monitoring & Alerts

- [ ] **Health checks** - Automated monitoring of gateway status
- [ ] **Log monitoring** - Alerts on errors or suspicious activity
- [ ] **Resource monitoring** - Alerts on high CPU/memory/disk usage
- [ ] **Cost monitoring** - Google Cloud and OpenAI budget alerts
- [ ] **Uptime monitoring** - External service checks availability
- [ ] **Backup verification** - Regular restore tests

### Compliance

- [ ] **Data retention policy** - Logs and backups retention defined
- [ ] **Access control** - Only authorized personnel have SSH access
- [ ] **Incident response plan** - Documented procedure for security incidents
- [ ] **Dependency updates** - Regular updates for security patches
- [ ] **Vulnerability scanning** - Regular security audits

### Security Testing

```bash
# Test firewall
nmap -p 1-65535 YOUR_VM_IP
# Should only show 22, 80, 443 open

# Test SSL
curl -I https://openclaw.yourdomain.com
# Should return 200 OK with valid certificate

# Test gateway token
curl -H "Authorization: Bearer wrong-token" https://openclaw.yourdomain.com
# Should return 401 Unauthorized

# Test health endpoint
curl https://openclaw.yourdomain.com/healthz
# Should return 200 OK

# Verify no secrets in logs
pm2 logs openclaw-gateway | grep -i "sk-"
# Should return nothing

# Docker: Check container security
docker scan ghcr.io/openclaw/openclaw:2026.5.7
```

### Security Incident Response

**If API key is compromised:**

1. **Immediately revoke** the compromised key
2. **Generate new key** in the provider dashboard
3. **Update configuration**:
   ```bash
   # PM2
   openclaw config set agents.main.auth.openai.apiKey NEW_KEY
   pm2 restart openclaw-gateway
   
   # Docker
   # Update in Portainer environment variables
   docker service update --force openclaw_openclaw_gateway
   ```
4. **Review logs** for unauthorized usage
5. **Monitor billing** for unexpected charges
6. **Document incident** for future reference

**If gateway token is compromised:**

1. **Generate new token**:
   ```bash
   openclaw doctor --generate-gateway-token
   ```
2. **Update configuration**:
   ```bash
   openclaw config set gateway.auth.token NEW_TOKEN
   ```
3. **Restart gateway**
4. **Revoke all devices**:
   ```bash
   openclaw devices clear
   ```
5. **Re-approve trusted devices** only

**If VM is compromised:**

1. **Isolate the VM** - Disconnect from network
2. **Create snapshot** - For forensic analysis
3. **Deploy new VM** - From clean image
4. **Restore from backup** - Use last known good backup
5. **Rotate all credentials** - API keys, tokens, passwords
6. **Review access logs** - Identify breach source
7. **Update security measures** - Prevent recurrence

---

## 11. Reference

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

## 11. Reference

### Quick Command Reference

#### PM2 Method Commands

```bash
# Status and Monitoring
pm2 status                              # Show all processes
pm2 monit                               # Real-time monitoring dashboard
pm2 logs openclaw-gateway               # View logs
pm2 logs openclaw-gateway --lines 100   # Last 100 lines

# Process Management
pm2 restart openclaw-gateway            # Restart gateway
pm2 stop openclaw-gateway               # Stop gateway
pm2 delete openclaw-gateway             # Remove from PM2
pm2 save                                # Save process list
pm2 startup                             # Enable auto-start on boot

# Logs
pm2 flush                               # Clear all logs
pm2 logs --err                          # Show only errors

# System
sudo systemctl status caddy             # Check Caddy status
sudo systemctl restart caddy            # Restart Caddy
sudo journalctl -u caddy -f             # Follow Caddy logs
```

#### Docker Method Commands

```bash
# Service Management
docker service ls                                           # List all services
docker service ps openclaw_openclaw_gateway                 # Service tasks
docker service logs -f openclaw_openclaw_gateway            # Follow logs
docker service logs openclaw_openclaw_gateway --tail 100    # Last 100 lines

# Updates and Rollbacks
docker service update --image ghcr.io/openclaw/openclaw:2026.5.8 openclaw_openclaw_gateway
docker service rollback openclaw_openclaw_gateway           # Rollback to previous
docker service update --force openclaw_openclaw_gateway     # Force restart

# Container Access
docker ps --filter name=openclaw                            # List containers
docker exec -it $(docker ps --filter name=openclaw -q) bash # Enter container
docker exec -it $(docker ps --filter name=openclaw -q) sh   # Enter (if bash unavailable)

# Monitoring
docker stats $(docker ps --filter name=openclaw -q)         # Resource usage
docker service inspect openclaw_openclaw_gateway            # Service details

# Stack Management
docker stack deploy -c docker-compose.yml openclaw          # Deploy stack
docker stack rm openclaw                                    # Remove stack
docker stack ps openclaw                                    # Stack status
```

#### OpenClaw CLI Commands (Both Methods)

```bash
# Configuration
openclaw config get                                         # View all config
openclaw config get gateway.auth.token                      # Get specific value
openclaw config set gateway.mode local                      # Set value
openclaw config set --batch-json '[{...}]'                  # Batch update

# Diagnostics
openclaw doctor                                             # Run diagnostics
openclaw doctor --fix                                       # Auto-fix issues
openclaw --version                                          # Show version

# Device Management
openclaw devices list                                       # List all devices
openclaw devices approve REQUEST_ID                         # Approve device
openclaw devices revoke DEVICE_ID                           # Revoke device
openclaw devices clear                                      # Clear all devices

# Gateway
openclaw gateway --allow-unconfigured                       # Start gateway manually
openclaw status                                             # Gateway status

# Skills
openclaw skills check --agent main                          # Check available skills
openclaw skills list                                        # List all skills

# Validation
node -e "JSON.parse(require('fs').readFileSync(process.env.HOME + '/.openclaw/openclaw.json','utf8')); console.log('✓ JSON is valid')"
```

### Configuration File Locations

#### PM2 Method

```text
~/.openclaw/openclaw.json              # Main configuration
~/.openclaw/workspace/                 # Agent workspaces
~/.openclaw/credentials/               # OAuth tokens
~/.openclaw/agents/                    # Agent configs
~/.openclaw/logs/                      # Application logs
/etc/caddy/Caddyfile                   # Caddy configuration
```

#### Docker Method

```text
/opt/infra/alobexpress/openclaw/openclaw.json              # Main configuration
/opt/infra/alobexpress/openclaw/workspace/                 # Agent workspaces
/opt/infra/alobexpress/openclaw/credentials/               # OAuth tokens
/opt/infra/alobexpress/openclaw/agents/                    # Agent configs
/opt/infra/alobexpress/openclaw/logs/                      # Application logs
/opt/infra/alobexpress/docker-compose.yml                  # Stack definition
```

### Port Reference

| Port | Service | Access | Purpose |
|------|---------|--------|---------|
| 22 | SSH | Public (restricted) | Server administration |
| 80 | HTTP | Public | Redirects to HTTPS |
| 443 | HTTPS | Public | Encrypted web traffic |
| 18789 | OpenClaw Gateway | **Local only** | Gateway API (never expose!) |
| 9615 | PM2 Web | Local only | PM2 dashboard (optional) |

### Environment Variables Reference

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `NODE_ENV` | No | `development` | Node environment (production/development) |
| `OPENCLAW_CONFIG` | No | `~/.openclaw/openclaw.json` | Config file path |
| `OPENCLAW_GATEWAY_BIND` | No | `localhost` | Gateway bind address (use `lan` for Docker) |
| `OPENCLAW_DISABLE_BONJOUR` | No | `0` | Disable mDNS (set `1` for Docker) |
| `OPENCLAW_GATEWAY_TOKEN` | Yes | - | Gateway authentication token |
| `OPENCLAW_HOOKS_TOKEN` | Yes | - | Webhook authentication token |
| `OPENAI_API_KEY` | Yes | - | OpenAI API key |
| `TELEGRAM_DEFAULT_BOT_TOKEN` | If using Telegram | - | Telegram bot token |
| `NOTION_API_KEY` | Optional | - | Notion integration token |
| `FIRECRAWL_API_KEY` | Optional | - | Firecrawl API key |
| `GOPLACES_API_KEY` | Optional | - | Google Places API key |
| `GITHUB_PERSONAL_ACCESS_TOKEN` | Optional | - | GitHub API token |

### Official Documentation Links

| Resource | URL |
|----------|-----|
| **OpenClaw Installation** | https://docs.openclaw.ai/install |
| **OpenClaw on GCP** | https://docs.openclaw.ai/install/gcp |
| **Control UI** | https://docs.openclaw.ai/web/control-ui |
| **Dashboard** | https://docs.openclaw.ai/web/dashboard |
| **Remote Access** | https://docs.openclaw.ai/gateway/remote |
| **Devices CLI** | https://docs.openclaw.ai/cli/devices |
| **Node.js Requirements** | https://docs.openclaw.ai/install/node |
| **Docker Image** | https://github.com/openclaw/openclaw/pkgs/container/openclaw |

### API Provider Links

| Provider | Dashboard | Billing | Documentation |
|----------|-----------|---------|---------------|
| **OpenAI** | https://platform.openai.com/ | https://platform.openai.com/settings/organization/billing/overview | https://platform.openai.com/docs |
| **OpenAI Usage** | https://platform.openai.com/usage | https://platform.openai.com/settings/organization/limits | - |
| **Google Cloud** | https://console.cloud.google.com/ | https://console.cloud.google.com/billing | https://cloud.google.com/docs |
| **Google Places API** | https://console.cloud.google.com/apis/library | https://console.cloud.google.com/apis/credentials | https://developers.google.com/maps/documentation/places |
| **Notion** | https://www.notion.so/my-integrations | - | https://developers.notion.com/ |
| **Telegram BotFather** | https://t.me/BotFather | - | https://core.telegram.org/bots |
| **Firecrawl** | https://www.firecrawl.dev/ | https://www.firecrawl.dev/pricing | https://docs.firecrawl.dev/ |
| **ElevenLabs** | https://elevenlabs.io/ | https://elevenlabs.io/pricing | https://docs.elevenlabs.io/ |
| **Cloudflare** | https://dash.cloudflare.com/ | - | https://developers.cloudflare.com/ |

### Architecture Summary

#### Traditional VM + PM2

```text
Internet → Cloudflare (CDN/DDoS) → Caddy (SSL/Proxy) → OpenClaw Gateway (PM2) → AI APIs
```

**Pros:**
- Simple setup
- Direct file access
- Lower resource overhead
- Easy debugging

**Cons:**
- Manual scaling
- Manual updates
- Single point of failure

#### Docker Swarm + Traefik

```text
Internet → Traefik (SSL/Proxy) → Docker Swarm → OpenClaw Container → AI APIs
```

**Pros:**
- Easy scaling
- Built-in rollback
- Visual management (Portainer)
- Isolated environment
- Quick updates

**Cons:**
- More complex setup
- Docker layer overhead
- Requires container knowledge

### Common Error Patterns

| Error Message | Likely Cause | Quick Fix |
|---------------|--------------|-----------|
| `origin not allowed` | CORS not configured | Add domain to `allowedOrigins` |
| `gateway token missing` | Token not entered | Enter token from config file |
| `device pairing required` | New device | Approve with `openclaw devices approve` |
| `No API key found` | API key not set | Configure with `openclaw config set` |
| `HTTP ERROR 502` | Gateway not running | Check logs, restart service |
| `bash: openclaw: command not found` | CLI not in PATH (Docker) | Create alias or use `docker exec` |
| `Proxy headers from untrusted` | Traefik IP not trusted | Add to `trustedProxies` |
| `JSON5: invalid character` | Manual config edit | Restore backup, use CLI only |
| `gateway.mode is unset` | Mode not configured | Set to `local` |
| `404 Not Found` | Volume/path issue | Check permissions and paths |

### Performance Benchmarks

| Metric | e2-standard-2 | e2-standard-4 | Notes |
|--------|---------------|---------------|-------|
| **Concurrent Agents** | 2-3 | 5-8 | Depends on complexity |
| **Response Time** | <1s | <500ms | Simple queries |
| **Memory Usage** | 2-4 GB | 4-8 GB | With active agents |
| **CPU Usage** | 40-60% | 20-40% | Under load |
| **Recommended Users** | 1-5 | 5-20 | Concurrent users |

### Cost Estimates (Monthly)

| Component | Cost Range | Notes |
|-----------|------------|-------|
| **GCP VM (e2-standard-2)** | $50-70 | us-central1 region |
| **GCP VM (e2-standard-4)** | $100-140 | us-central1 region |
| **Static IP** | $3-5 | If not attached to running VM |
| **Disk (50 GB SSD)** | $8-10 | Standard persistent disk |
| **OpenAI API** | $10-200+ | Highly variable by usage |
| **Cloudflare** | $0 | Free tier sufficient |
| **Domain** | $10-15/year | Varies by registrar |
| **Total (Small)** | $70-100 | e2-standard-2 + light API usage |
| **Total (Medium)** | $150-300 | e2-standard-4 + moderate API usage |

### Support and Community

| Resource | Link |
|----------|------|
| **GitHub Issues** | https://github.com/openclaw/openclaw/issues |
| **Discord Community** | Check OpenClaw website |
| **Documentation** | https://docs.openclaw.ai/ |
| **Release Notes** | https://github.com/openclaw/openclaw/releases |

---

## 📝 Final Notes

### Best Practices

1. **Always backup before changes** - Configuration, updates, or major changes
2. **Use `openclaw config set`** - Never manually edit `openclaw.json`
3. **Monitor API costs** - Set spending limits on all providers
4. **Keep secrets secure** - Never commit API keys to git
5. **Test in staging first** - If possible, test updates before production
6. **Document customizations** - Keep notes on your specific setup
7. **Regular security audits** - Review access logs and permissions monthly
8. **Automate backups** - Set up cron jobs for daily backups
9. **Monitor resource usage** - Set up alerts for high CPU/memory/disk
10. **Stay updated** - Follow OpenClaw releases for security patches

### Getting Help

If you encounter issues not covered in this guide:

1. **Check logs first** - Most issues are visible in logs
2. **Run diagnostics** - `openclaw doctor` catches common problems
3. **Search GitHub issues** - Someone may have had the same problem
4. **Check official docs** - Documentation is regularly updated
5. **Ask the community** - Discord or GitHub discussions
6. **Provide context** - Include logs, config (redact secrets), and steps to reproduce

### Contributing

Found an error in this guide or have improvements? Contributions are welcome!

---

**Last Updated:** 2026-05-15  
**OpenClaw Version:** 2026.5.7  
**Guide Version:** 2.0

---

Made with ❤️ for the OpenClaw community
