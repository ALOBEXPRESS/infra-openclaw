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

## 📋 Pré-requisitos

### Requisitos Comuns (Ambos os Métodos)

- [ ] **Conta Google Cloud** com Compute Engine habilitado
- [ ] **Domínio ou subdomínio** (ex: `openclaw.alobexpress.com.br`)
- [ ] **Chave API OpenAI** para acesso aos modelos de IA
- [ ] **Acesso SSH** ao seu terminal local

### Requisitos do Método 1: VM Tradicional

- [ ] **Conta Cloudflare** gerenciando o DNS do seu domínio
- [ ] Conhecimento básico de linha de comando Linux

### Requisitos do Método 2: Docker Swarm

- [ ] **Docker Engine** 24.x ou superior
- [ ] **Docker Swarm** inicializado
- [ ] **Traefik** reverse proxy configurado
- [ ] **Portainer** (opcional, para gerenciamento visual)
- [ ] Rede externa `network_swarm_public` criada

### Opcional (para recursos estendidos)

- [ ] Chave API Google Places (para buscas baseadas em localização)
- [ ] Token de integração Notion (para operações de banco de dados)
- [ ] Token de Bot Telegram (para integrações de chat)
- [ ] Chave API Firecrawl (para web scraping)
- [ ] Chave API ElevenLabs (para síntese de voz)

---

## 🏗 Visão Geral da Arquitetura

### Arquitetura do Sistema

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

### Por Que Esta Arquitetura?

| Benefício | Explicação |
|---------|-------------|
| **Segurança** | Porta 18789 nunca exposta publicamente; todo tráfego através de Cloudflare + Caddy |
| **Confiabilidade** | PM2 garante que o gateway reinicie automaticamente em caso de falhas |
| **Performance** | CDN Cloudflare faz cache de assets estáticos globalmente |
| **Escalabilidade** | Fácil atualizar o tamanho da VM conforme o uso cresce |
| **Manutenibilidade** | Separação de responsabilidades: Cloudflare (edge), Caddy (proxy), OpenClaw (app) |
| **SSL/TLS** | Gerenciamento automático de certificados via Caddy |

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

## 🚀 Métodos de Implantação

Escolha o método de implantação que melhor se adequa às suas necessidades:

### Comparação Rápida

| Aspecto | VM Tradicional + PM2 | Docker Swarm + Traefik |
|--------|---------------------|------------------------|
| **Tempo de Configuração** | 30-45 minutos | 15-20 minutos (se Swarm pronto) |
| **Complexidade** | Baixa | Média |
| **Melhor Para** | Iniciantes, pequenas equipes, aprendizado | Produção, equipes DevOps, escalabilidade |
| **Escalabilidade** | Manual (upgrade de VM) | Automática (adicionar nós) |
| **Gerenciamento** | Linha de comando (SSH) | Interface Portainer + CLI |
| **Atualizações** | `npm install -g openclaw@latest` | `docker service update --image` |
| **Rollback** | Restauração manual de backup | Integrado (`docker service rollback`) |
| **Uso de Recursos** | Menor overhead | Ligeiramente maior (camada Docker) |
| **Monitoramento** | PM2 monit | Docker stats + Portainer |
| **Multi-instância** | Requer configuração manual | Suporte nativo |
| **Gerenciamento SSL** | Caddy (automático) | Traefik (automático) |
| **Custo** | Apenas VM | VM + potencial overhead de orquestração |

### Guia de Decisão

**Escolha VM Tradicional + PM2 se:**
- ✅ Você é novo no OpenClaw ou Docker
- ✅ Você quer configuração simples e direta
- ✅ Você está executando uma única instância
- ✅ Você prefere acesso direto ao sistema de arquivos
- ✅ Você quer menor overhead de recursos

**Escolha Docker Swarm + Traefik se:**
- ✅ Você já tem infraestrutura Docker
- ✅ Você precisa de escalabilidade fácil e alta disponibilidade
- ✅ Você quer gerenciamento visual (Portainer)
- ✅ Você está confortável com contêineres
- ✅ Você precisa de rollbacks e atualizações rápidas
- ✅ Você está executando múltiplos serviços com Traefik

---

## Método 1: VM Tradicional com PM2 (Recomendado para Iniciantes)

Este método usa uma configuração tradicional de VM com PM2 para gerenciamento de processos e Caddy para proxy reverso.

### 1. Configuração da Infraestrutura

#### Passo 1.1: Criar VM no Google Cloud

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

**Regras de Firewall:**

Certifique-se de que estas portas estão abertas no firewall da sua VPC:

| Porta | Protocolo | Propósito | Público? |
|------|----------|---------|---------|
| 22 | TCP | Acesso SSH | Sim (restrito ao seu IP recomendado) |
| 80 | TCP | HTTP (redireciona para HTTPS) | Sim |
| 443 | TCP | HTTPS | Sim |
| 18789 | TCP | OpenClaw Gateway | **NÃO** (apenas interno) |

**Nota de Segurança**: Nunca exponha a porta 18789 publicamente. Ela deve ser acessível apenas via `127.0.0.1` (localhost).

#### Passo 1.2: Reservar IP Estático

Um IP estático evita que seu DNS quebre quando a VM reiniciar.

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

**Obtenha seu IP estático:**
```bash
gcloud compute addresses describe openclaw-static-ip \
  --region=us-central1 \
  --format="get(address)"
```

#### Passo 1.3: Configurar DNS no Cloudflare

1. Log in to [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Select your domain (e.g., `alobexpress.com.br`)
3. Go to **DNS** → **Records**

**Adicionar Registros DNS:**

| Type | Name | Content | Proxy Status | TTL |
|------|------|---------|--------------|-----|
| A | advanced | `YOUR_VM_STATIC_IP` | Proxied (🟠) | Auto |
| CNAME | openclaw | advanced.alobexpress.com.br | Proxied (🟠) | Auto |

**Example:**
```text
A     advanced    34.123.45.67    Proxied    Auto
CNAME openclaw    advanced.alobexpress.com.br    Proxied    Auto
```

**Status do Proxy**: A nuvem laranja (🟠 Proxied) significa que o tráfego passa pela CDN da Cloudflare.

**Configuração SSL/TLS:**

1. Go to **SSL/TLS** → **Overview**
2. Set encryption mode to **Full**
   - ❌ Not `Flexible` (insecure)
   - ✅ Use `Full` or `Full (strict)`

**Por que Full?**
- `Flexible`: Cloudflare ↔ Usuário é HTTPS, mas Cloudflare ↔ Servidor é HTTP (inseguro)
- `Full`: HTTPS ponta a ponta (Caddy gerencia SSL no lado do servidor)

**Propagação DNS:**

Após adicionar os registros, aguarde 5-10 minutos para propagação do DNS. Verifique com:

```bash
# Check DNS resolution
nslookup openclaw.alobexpress.com.br

# Or use dig
dig openclaw.alobexpress.com.br
```

**Nota**: Com o proxy Cloudflare habilitado, ferramentas DNS mostrarão IPs da Cloudflare, não o IP da sua VM. Isso é esperado.

---

### 2. Instalação do OpenClaw

#### Passo 2.1: Acessar Sua VM

**Via Console do Google Cloud:**
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

#### Passo 2.2: Preparar o Ambiente

Atualize os pacotes do sistema e instale dependências:

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

**O que cada pacote faz:**
- `curl`: Baixar arquivos e fazer requisições HTTP
- `ca-certificates`: Validação de certificados SSL
- `gnupg`: Gerenciamento de chaves GPG
- `git`: Controle de versão (útil para atualizações futuras)
- `build-essential`: Compiladores para módulos nativos do Node.js
- `unzip`: Extrair arquivos compactados
- `nano`: Editor de texto
- `htop`: Monitoramento de processos

#### Passo 2.3: Instalar Node.js

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

#### Passo 2.4: Instalar OpenClaw

**Método 1: Instalador Oficial (recomendado)**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Este script:
- Detecta seu SO e arquitetura
- Instala OpenClaw globalmente
- Configura a configuração inicial
- Adiciona OpenClaw ao seu PATH

**Método 2: Instalação Global via npm**

```bash
sudo npm install -g openclaw@latest
```

**Verificar Instalação:**

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

#### Passo 2.5: Configuração Inicial

Execute o assistente de configuração:

```bash
openclaw setup --wizard
```

**Ou use o comando de configuração interativo:**

```bash
openclaw configure
```

**Prompts de Configuração:**

| Prompt | Valor Recomendado | Notas |
|--------|-------------------|-------|
| **Modo Gateway** | Local | Para implantação em servidor único |
| **Provedor IA** | OpenAI | Ou Claude, Gemini, etc. |
| **Modelo** | gpt-4o-mini | Custo-efetivo para testes |
| **Canais** | Telegram | Adicione outros conforme necessário |
| **Plugins** | Padrão | Personalize depois |

**Localização do Arquivo de Configuração:**

```text
~/.openclaw/openclaw.json          # If running as regular user
/root/.openclaw/openclaw.json      # If running as root
```

**Importante**: Sempre use o mesmo usuário para configuração e execução do OpenClaw. Misturar usuários causa problemas de permissão.

**Visualizar Configuração:**

```bash
cat ~/.openclaw/openclaw.json
```

---

### 3. Configuração do Gateway

#### Passo 3.1: Configurar Token do Gateway

O token do gateway autentica o acesso ao dashboard.

**Abrir arquivo de configuração:**

```bash
nano ~/.openclaw/openclaw.json
```

**Ou se executando como root:**

```bash
sudo nano /root/.openclaw/openclaw.json
```

**Encontre a seção gateway:**

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

#### Passo 3.2: Permitir Acesso de Domínio Público

Por padrão, OpenClaw bloqueia requisições de origens desconhecidas. Configure domínios permitidos:

**Editar configuração:**

```bash
nano ~/.openclaw/openclaw.json
```

**Adicionar seção `controlUi`:**

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

**Opções de Configuração:**

| Opção | Tipo | Propósito |
|--------|------|---------|
| `allowInsecureAuth` | boolean | Permitir autenticação baseada em token (defina `true` para produção com HTTPS) |
| `allowedOrigins` | array | Lista de domínios permitidos que podem acessar o gateway |

**Múltiplos Domínios:**

```json
"allowedOrigins": [
  "https://openclaw.alobexpress.com.br",
  "https://openclaw-staging.alobexpress.com.br",
  "http://localhost:3000"
]
```

**Validar Sintaxe JSON:**

```bash
# Check for syntax errors
node -e "JSON.parse(require('fs').readFileSync(process.env.HOME + '/.openclaw/openclaw.json','utf8')); console.log('✓ JSON is valid')"
```

**If running as root:**

```bash
node -e "JSON.parse(require('fs').readFileSync('/root/.openclaw/openclaw.json','utf8')); console.log('✓ JSON is valid')"
```

**Erros JSON Comuns:**
- Vírgula faltando entre propriedades
- Vírgula sobrando após última propriedade
- Chaves `{}` não correspondentes
- Colchetes `[]` não correspondentes

#### Passo 3.3: Instalar e Configurar Caddy

Caddy é um servidor web moderno com HTTPS automático.

**Instalar Caddy:**

```bash
# Update package list
sudo apt update

# Install Caddy
sudo apt install -y caddy

# Verify installation
caddy version
```

**Configurar Caddyfile:**

```bash
sudo nano /etc/caddy/Caddyfile
```

**Adicionar esta configuração:**

```caddy
openclaw.alobexpress.com.br {
    reverse_proxy 127.0.0.1:18789
}
```

**O que isso faz:**
- Escuta nas portas 80 e 443
- Obtém automaticamente certificado SSL do Let's Encrypt
- Faz proxy de todas as requisições para o gateway OpenClaw em localhost:18789
- Gerencia redirecionamentos HTTP → HTTPS

**Caddyfile Avançado (com logging e headers):**

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

**Reiniciar Caddy:**

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

**Visualizar logs do Caddy:**

```bash
sudo journalctl -u caddy -f
```

#### Passo 3.4: Testar o Gateway

Inicie o gateway OpenClaw manualmente para verificar a configuração:

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

**Testar acesso local:**

```bash
# In another terminal
curl http://127.0.0.1:18789/health
```

**Testar acesso público:**

Abra seu navegador e navegue para:

```text
https://openclaw.alobexpress.com.br
```

**Você deve ver:**
- Tela de login do dashboard OpenClaw
- Prompt para token do gateway

**Digite o token de `~/.openclaw/openclaw.json`**

**Parar o gateway manual:**

Pressione `Ctrl+C` no terminal executando `openclaw gateway`

#### Passo 3.5: Pareamento de Dispositivos

OpenClaw usa pareamento de dispositivos para segurança.

**No primeiro acesso ao dashboard, você verá:**

```text
Device pairing required
Request ID: 1e0ea8bf-2b7d-41ac-aca4-42e64f78ec70
```

**No terminal da sua VM, aprove o dispositivo:**

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

**Notas Importantes:**
- IDs de requisição expiram após 5 minutos
- Se expirado, atualize o dashboard para gerar uma nova requisição
- Aprove imediatamente após ver o ID da requisição
- Cada navegador/dispositivo precisa de aprovação separada

**Gerenciar dispositivos:**

```bash
# List all approved devices
openclaw devices list --approved

# Revoke a device
openclaw devices revoke DEVICE_ID

# Clear all devices
openclaw devices clear
```

---

### 4. Gerenciamento de Processos com PM2

PM2 mantém o OpenClaw rodando 24/7 com reinicializações automáticas.

#### Passo 4.1: Instalar PM2

```bash
sudo npm install -g pm2
```

#### Passo 4.2: Iniciar OpenClaw com PM2

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

#### Passo 4.3: Salvar Configuração do PM2

```bash
# Save current process list
pm2 save

# Generate startup script
pm2 startup
```

**PM2 irá gerar um comando como:**

```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u username --hp /home/username
```

**Copie e execute esse comando exato.**

Isso garante que o OpenClaw inicie automaticamente após reinicializações do servidor.

#### Passo 4.4: Referência de Comandos PM2

| Comando | Propósito |
|---------|---------|
| `pm2 status` | Mostrar todos os processos |
| `pm2 logs openclaw-gateway` | Ver logs em tempo real |
| `pm2 logs openclaw-gateway --lines 100` | Ver últimas 100 linhas de log |
| `pm2 restart openclaw-gateway` | Reiniciar o gateway |
| `pm2 stop openclaw-gateway` | Parar o gateway |
| `pm2 delete openclaw-gateway` | Remover do PM2 |
| `pm2 monit` | Dashboard de monitoramento em tempo real |
| `pm2 flush` | Limpar todos os logs |

**Após mudanças de configuração:**

```bash
# Restart to apply new config
pm2 restart openclaw-gateway

# View logs to verify
pm2 logs openclaw-gateway --lines 50
```

---

## Método 2: Docker Swarm com Traefik (Recomendado para Produção)

Este método usa Docker Swarm para orquestração e Traefik para SSL automático e roteamento.

### Pré-requisitos para o Método Docker

Certifique-se de ter:

| Requisito | Versão Mínima |
|-------------|-----------------|
| Docker Engine | 24.x |
| Docker Compose / Swarm | v2.x |
| Traefik (reverse proxy) | v2.x ou v3.x |
| Portainer (opcional) | Mais recente |

**Antes de começar:**

```bash
# Initialize Docker Swarm (if not already done)
docker swarm init

# Create external network
docker network create --driver overlay --attachable network_swarm_public

# Verify Traefik is running
docker service ls | grep traefik
```

### Passo 1: Preparar Diretório de Dados

```bash
# Create directory for OpenClaw data
mkdir -p /opt/infra/alobexpress/openclaw

# Set correct permissions (uid 1000 = node user in container)
chown -R 1000:1000 /opt/infra/alobexpress/openclaw
```

**Por que uid 1000?** A imagem Docker oficial do OpenClaw executa como usuário `node` (uid 1000). Sem as permissões corretas, o contêiner não pode escrever arquivos de configuração.

### Passo 2: Criar docker-compose.yml

Crie `/opt/infra/alobexpress/docker-compose.yml`:

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

Esta seção cobre problemas comuns para ambos os métodos de implantação.

### Problemas Comuns (Ambos os Métodos)

#### ❌ `origin not allowed` no WebSocket

**Causa:** Gateway bloqueia conexões de origens desconhecidas.

**Solução para PM2:**
```bash
nano ~/.openclaw/openclaw.json
```

Adicione em `gateway.controlUi`:
```json
"allowedOrigins": ["https://openclaw.seudominio.com.br"]
```

Reinicie:
```bash
pm2 restart openclaw-gateway
```

**Solução para Docker:**
```bash
openclaw config set \
  --batch-json '[{"path":"gateway.controlUi.allowedOrigins","value":["https://openclaw.seudominio.com.br"]}]'

docker service update --force openclaw_openclaw_gateway
```

---

#### ❌ `No API key found for provider "openai"`

**Causa:** Chave API OpenAI não configurada.

**Solução para PM2:**
```bash
openclaw configure
# Ou
export OPENAI_API_KEY="sk-..."
```

**Solução para Docker:**
```bash
# Verificar se variável de ambiente está definida
docker exec -it $(docker ps --filter name=openclaw -q) printenv OPENAI_API_KEY

# Se vazio, definir na configuração
openclaw config set agents.main.auth.openai.apiKey sk-...

# Ou atualizar variáveis de ambiente no Portainer e reimplantar
```

---

#### ❌ `device pairing required (requestId: ...)`

**Causa:** Novo navegador/dispositivo precisa de aprovação.

**Solução (Ambos os Métodos):**
```bash
# Listar requisições pendentes
openclaw devices list

# Aprovar dispositivo
openclaw devices approve REQUEST_ID_HERE
```

**Nota:** IDs de requisição expiram após 5 minutos. Se expirado, atualize o dashboard para gerar um novo.

---

#### ❌ `gateway.mode is unset; gateway start will be blocked`

**Causa:** Modo do gateway não configurado.

**Solução (Ambos os Métodos):**
```bash
openclaw config set gateway.mode local
```

Depois reinicie o serviço.

---

### Problemas Específicos do Método 1 (PM2)

#### ❌ `Unable to locate package npm`

**Causa:** Node.js não instalado ou repositório desatualizado.

**Solução:**
```bash
# Instalar via NodeSource
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt install -y nodejs

# Verificar
node -v
npm -v
```

---

#### ❌ `HTTP ERROR 502`

**Causa:** Caddy está rodando mas o gateway OpenClaw não está respondendo.

**Solução:**
```bash
# Verificar status do PM2
pm2 status

# Ver logs
pm2 logs openclaw-gateway

# Verificar se a porta está escutando
ss -lntp | grep 18789

# Reiniciar se necessário
pm2 restart openclaw-gateway
```

---

#### ❌ `gateway token missing`

**Causa:** Token não inserido no dashboard.

**Solução:**
```bash
# Ver token
cat ~/.openclaw/openclaw.json | grep token

# Ou
openclaw config get gateway.auth.token
```

Digite este token no campo "Token do Gateway" do dashboard.

---

#### ❌ JSON inválido

**Causa:** Edição manual introduziu erros de sintaxe.

**Solução:**
```bash
# Validar JSON
node -e "JSON.parse(require('fs').readFileSync(process.env.HOME + '/.openclaw/openclaw.json','utf8')); console.log('✓ JSON is valid')"

# Se inválido, restaurar backup
cp ~/.openclaw/openclaw.json.bak ~/.openclaw/openclaw.json

# Reiniciar
pm2 restart openclaw-gateway
```

---

#### ❌ gcloud: `project is not currently set`

**Causa:** Projeto do Google Cloud não configurado.

**Solução:**
```bash
# Listar projetos
gcloud projects list

# Definir projeto
gcloud config set project YOUR_PROJECT_ID
```

---

#### ❌ gcloud: `insufficient authentication scopes`

**Causa:** Autenticação expirada ou permissões insuficientes.

**Solução:**
```bash
gcloud auth login
gcloud auth application-default login
```

---

### Problemas Específicos do Método 2 (Docker)

#### ❌ `bash: openclaw: command not found` (exit 127)

**Causa:** Alias CLI não configurado ou usando imagem errada.

**Solução:**
```bash
# Verificar se está usando imagem oficial
docker service inspect openclaw_openclaw_gateway | grep Image
# Deve mostrar: ghcr.io/openclaw/openclaw:2026.5.7

# Criar alias
echo 'alias openclaw="docker exec -it \$(docker ps --filter name=openclaw -q) node dist/index.js"' >> ~/.bashrc
source ~/.bashrc

# Testar
openclaw --version
```

---

#### ❌ `Proxy headers detected from untrusted address`

**Causa:** IP do Traefik não está na lista de proxies confiáveis.

**Solução:**
```bash
# Encontrar IP do Traefik nos logs
docker service logs openclaw_openclaw_gateway | grep "peer="
# Exemplo: peer=10.0.1.3:54321

# Adicionar aos proxies confiáveis
openclaw config set \
  --batch-json '[{"path":"gateway.trustedProxies","value":["10.0.1.3"]}]'

# Reiniciar
docker service update --force openclaw_openclaw_gateway
```

---

#### ❌ `JSON5: invalid character '"' at 11:2`

**Causa:** Edição manual introduziu aspas tipográficas ou JSON inválido.

**Solução:**
```bash
# Restaurar último backup bom
cp /opt/infra/alobexpress/openclaw/openclaw.json.last-good \
   /opt/infra/alobexpress/openclaw/openclaw.json

# Ou executar doctor
openclaw doctor --fix

# Reiniciar serviço
docker service update --force openclaw_openclaw_gateway
```

**Importante:** Nunca edite `openclaw.json` manualmente. Sempre use `openclaw config set`.

---

#### ❌ `404 Not Found` no domínio

**Causa:** Incompatibilidade de caminho de volume ou problemas de permissão.

**Solução:**
```bash
# Verificar se o caminho existe
ls -la /opt/infra/alobexpress/openclaw

# Corrigir permissões (uid 1000 = usuário node)
chown -R 1000:1000 /opt/infra/alobexpress/openclaw

# Verificar volume no serviço
docker service inspect openclaw_openclaw_gateway | grep -A5 Mounts

# Reiniciar serviço
docker service update --force openclaw_openclaw_gateway
```

---

#### ❌ Contêiner continua reiniciando

**Causa:** Erro de configuração ou variáveis de ambiente obrigatórias faltando.

**Solução:**
```bash
# Ver logs
docker service logs openclaw_openclaw_gateway --tail 100

# Verificar variáveis de ambiente
docker exec -it $(docker ps --filter name=openclaw -q) env | grep OPENCLAW

# Verificar se variáveis obrigatórias estão definidas
docker exec -it $(docker ps --filter name=openclaw -q) printenv OPENAI_API_KEY
docker exec -it $(docker ps --filter name=openclaw -q) printenv OPENCLAW_GATEWAY_TOKEN

# Atualizar variáveis de ambiente no Portainer e reimplantar
```

---

#### ❌ Healthcheck falhando

**Causa:** Gateway não está respondendo na porta 18789.

**Solução:**
```bash
# Verificar se a porta está escutando dentro do contêiner
docker exec -it $(docker ps --filter name=openclaw -q) curl http://127.0.0.1:18789/healthz

# Ver logs detalhados
docker service logs openclaw_openclaw_gateway --tail 200

# Verificar saúde do serviço
docker service ps openclaw_openclaw_gateway

# Se necessário, aumentar start_period no healthcheck
# Editar docker-compose.yml:
# start_period: 180s  # Dar mais tempo para inicialização
```

---

### Comandos de Diagnóstico

**Para Método PM2:**
```bash
# Verificação completa do sistema
openclaw doctor

# Ver toda a configuração
openclaw config get

# Verificar status do PM2
pm2 status
pm2 monit

# Ver logs
pm2 logs openclaw-gateway --lines 100

# Verificar porta
ss -lntp | grep 18789

# Testar acesso local
curl http://127.0.0.1:18789/healthz

# Verificar Caddy
sudo systemctl status caddy
sudo journalctl -u caddy -f
```

**Para Método Docker:**
```bash
# Verificação completa do sistema
openclaw doctor

# Ver toda a configuração
openclaw config get

# Verificar status do serviço
docker service ls | grep openclaw
docker service ps openclaw_openclaw_gateway

# Ver logs
docker service logs -f openclaw_openclaw_gateway

# Verificar saúde do contêiner
docker ps --filter name=openclaw

# Entrar no contêiner
docker exec -it $(docker ps --filter name=openclaw -q) bash

# Testar dentro do contêiner
docker exec -it $(docker ps --filter name=openclaw -q) curl http://127.0.0.1:18789/healthz

# Verificar Traefik
docker service logs traefik | grep openclaw

# Ver detalhes do serviço
docker service inspect openclaw_openclaw_gateway
```

---

## 9. Manutenção

### 9.1. Backup

**O Que Fazer Backup:**

| Caminho | Contém | Crítico |
|------|----------|----------|
| `~/.openclaw/openclaw.json` | Main configuration | Yes |
| `~/.openclaw/workspace` | Agent workspaces | Yes |
| `~/.openclaw/credentials` | OAuth tokens | Yes |
| `~/.openclaw/agents` | Agent configurations | Yes |
| `~/.openclaw/logs` | Application logs | No |

**Para Método PM2:**

```bash
# Criar backup
tar -czf openclaw-backup-$(date +%F).tar.gz ~/.openclaw

# Se executando como root
tar -czf openclaw-backup-$(date +%F).tar.gz /root/.openclaw

# Armazenar backup com segurança (contém segredos!)
mv openclaw-backup-*.tar.gz /local/seguro/backup/
```

**Para Método Docker:**

```bash
# Backup do host (dados estão no volume)
tar -czf openclaw-backup-$(date +%F).tar.gz /opt/infra/alobexpress/openclaw

# Ou backup do contêiner
docker exec $(docker ps --filter name=openclaw -q) tar -czf /tmp/backup.tar.gz /home/node/.openclaw
docker cp $(docker ps --filter name=openclaw -q):/tmp/backup.tar.gz ./openclaw-backup-$(date +%F).tar.gz

# Armazenar backup com segurança
mv openclaw-backup-*.tar.gz /local/seguro/backup/
```

**Script de Backup Automatizado:**

```bash
#!/bin/bash
# /usr/local/bin/backup-openclaw.sh

BACKUP_DIR="/secure/backups/openclaw"
DATE=$(date +%F)
RETENTION_DAYS=30

# Criar backup
tar -czf "$BACKUP_DIR/openclaw-$DATE.tar.gz" ~/.openclaw

# Remover backups antigos
find "$BACKUP_DIR" -name "openclaw-*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup concluído: openclaw-$DATE.tar.gz"
```

**Agendar com cron:**

```bash
# Editar crontab
crontab -e

# Adicionar backup diário às 2h da manhã
0 2 * * * /usr/local/bin/backup-openclaw.sh
```

**⚠️ Aviso de Segurança:** Backups contêm chaves API, tokens e credenciais. Nunca faça commit em repositórios públicos ou armazene em locais não seguros.

### 9.2. Atualizações

#### Atualizando Método PM2

```bash
# Fazer backup primeiro!
tar -czf openclaw-backup-before-update-$(date +%F).tar.gz ~/.openclaw

# Atualizar OpenClaw
sudo npm install -g openclaw@latest

# Reiniciar gateway
pm2 restart openclaw-gateway

# Verificar
openclaw --version
openclaw doctor

# Verificar logs
pm2 logs openclaw-gateway --lines 50
```

#### Atualizando Método Docker

```bash
# Fazer backup primeiro!
tar -czf openclaw-backup-before-update-$(date +%F).tar.gz /opt/infra/alobexpress/openclaw

# Atualizar para versão específica
docker service update --image ghcr.io/openclaw/openclaw:2026.5.8 openclaw_openclaw_gateway

# Ou atualizar para a mais recente
docker service update --image ghcr.io/openclaw/openclaw:latest openclaw_openclaw_gateway

# Monitorar atualização
docker service ps openclaw_openclaw_gateway

# Verificar logs
docker service logs -f openclaw_openclaw_gateway

# Verificar
openclaw --version
openclaw doctor
```

**Reverter se necessário:**

```bash
# Docker tem rollback integrado
docker service rollback openclaw_openclaw_gateway

# Ou especificar versão anterior
docker service update --image ghcr.io/openclaw/openclaw:2026.5.7 openclaw_openclaw_gateway
```

### 9.3. Monitoramento

#### Monitoramento PM2

```bash
# Monitoramento em tempo real
pm2 monit

# Status dos processos
pm2 status

# Ver logs
pm2 logs openclaw-gateway

# Uso de CPU e memória
pm2 describe openclaw-gateway

# Dashboard web (opcional)
pm2 web
# Acessar em http://localhost:9615
```

#### Monitoramento Docker

```bash
# Status do serviço
docker service ps openclaw_openclaw_gateway

# Uso de recursos
docker stats $(docker ps --filter name=openclaw -q)

# Logs
docker service logs -f openclaw_openclaw_gateway

# Status de saúde
docker ps --filter name=openclaw --format "table {{.Names}}\t{{.Status}}"

# Dashboard Portainer
# Acessar em https://portainer.seudominio.com
```

### 9.4. Gerenciamento de Logs

#### Logs PM2

```bash
# Ver logs
pm2 logs openclaw-gateway

# Limpar logs
pm2 flush

# Rotacionar logs (configurar em ecosystem.config.js)
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
```

#### Logs Docker

```bash
# Ver logs
docker service logs openclaw_openclaw_gateway --tail 100

# Seguir logs
docker service logs -f openclaw_openclaw_gateway

# Logs são automaticamente rotacionados pelo Docker
# Configurar em /etc/docker/daemon.json:
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

### 9.5. Ajuste de Performance

#### Recursos da VM (Ambos os Métodos)

**Monitorar uso de recursos:**

```bash
# CPU e memória
htop

# Uso de disco
df -h
du -sh ~/.openclaw/*

# Rede
netstat -tulpn | grep 18789
```

**Quando escalar:**

| Métrica | Limite | Ação |
|--------|-----------|--------|
| Uso de CPU | >80% sustentado | Atualizar para e2-standard-4 |
| Uso de memória | >90% | Adicionar mais RAM |
| Uso de disco | >80% | Aumentar tamanho do disco |
| Tempo de resposta | >2s | Verificar logs, otimizar agentes |

#### Limites de Recursos Docker

Editar `docker-compose.yml`:

```yaml
resources:
  reservations:
    cpus: "0.5"      # Aumentar se necessário
    memory: 1024M    # Aumentar se necessário
  limits:
    cpus: "4.0"      # CPUs máximas
    memory: 8192M    # Memória máxima
```

Reimplantar:

```bash
docker stack deploy -c docker-compose.yml openclaw
```

### 9.6. Verificações de Saúde

#### Verificação Manual de Saúde

```bash
# Verificar saúde do gateway
curl http://localhost:18789/healthz

# Resposta esperada: 200 OK
```

#### Monitoramento Automatizado

**Para PM2 (usando cron):**

```bash
#!/bin/bash
# /usr/local/bin/check-openclaw-health.sh

if ! curl -sf http://localhost:18789/healthz > /dev/null; then
    echo "Verificação de saúde do OpenClaw falhou, reiniciando..."
    pm2 restart openclaw-gateway
    # Enviar alerta (email, Slack, etc.)
fi
```

**Para Docker (integrado):**

Healthcheck já está configurado no docker-compose.yml. Docker reinicia automaticamente contêineres não saudáveis.

### 9.7. Atualizações de Segurança

```bash
# Atualizar pacotes do sistema (ambos os métodos)
sudo apt update
sudo apt upgrade -y

# Atualizar Node.js (método PM2)
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt install -y nodejs

# Atualizar Docker (método Docker)
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Reiniciar serviços após atualizações
# PM2:
pm2 restart openclaw-gateway

# Docker:
docker service update --force openclaw_openclaw_gateway
```

---

## 10. Checklist de Segurança

Antes de ir para produção, verifique se todas as medidas de segurança estão implementadas:

### Segurança da Infraestrutura

#### Comum (Ambos os Métodos)

- [ ] **Firewall configurado** - Apenas portas 22, 80, 443 abertas publicamente
- [ ] **Porta 18789 NÃO exposta** - Gateway acessível apenas via proxy reverso
- [ ] **IP estático reservado** - Previne quebra de DNS ao reiniciar VM
- [ ] **Autenticação por chave SSH** - Autenticação por senha desabilitada
- [ ] **Token de gateway forte** - Mínimo 32 caracteres, gerado aleatoriamente
- [ ] **Device pairing enabled** - Only approved devices can connect
- [ ] **HTTPS enforced** - All traffic encrypted (Cloudflare/Traefik)
- [ ] **Regular backups** - Automated daily backups to secure location
- [ ] **Criptografia de backup** - Backups contêm segredos, devem ser criptografados
- [ ] **Atualizações do sistema** - SO e pacotes atualizados regularmente

#### Específico do Método PM2

- [ ] **Proxy Cloudflare ativo** - Nuvem laranja habilitada para proteção DDoS
- [ ] **Modo SSL/TLS: Full** - Não Flexible (inseguro)
- [ ] **Auto-atualizações Caddy** - Certificados SSL renovam automaticamente
- [ ] **Startup PM2 configurado** - Gateway inicia na inicialização
- [ ] **Rotação de logs habilitada** - Previne problemas de espaço em disco

#### Específico do Método Docker

- [ ] **SSL Traefik configurado** - Certificados Let's Encrypt funcionando
- [ ] **Proxies confiáveis definidos** - IP do Traefik na configuração do gateway
- [ ] **Usuário do contêiner** - Executando como não-root (uid 1000)
- [ ] **Permissões de volume** - Propriedade correta (1000:1000)
- [ ] **Limites de recursos** - Limites de CPU e memória configurados
- [ ] **Verificações de saúde** - Monitoramento de saúde do contêiner ativo
- [ ] **Gerenciamento de segredos** - Chaves API em variáveis de ambiente, não na imagem
- [ ] **Imagem fixada** - Usando tag de versão específica, não `latest`

### Segurança de API

- [ ] **Limites de uso OpenAI** - Limite de gastos mensais configurado
- [ ] **Chave OpenAI restrita** - Não compartilhada publicamente ou no git
- [ ] **Chave Google Places restrita** - Restrições de IP e API aplicadas
- [ ] **Integração Notion com escopo** - Compartilhada apenas com páginas necessárias
- [ ] **Token bot Telegram seguro** - Não exposto em logs ou erros
- [ ] **Token GitHub com escopo** - Permissões mínimas necessárias
- [ ] **Todas as chaves API rotacionadas** - Cronograma de rotação regular (90 dias)

### Segurança da Aplicação

- [ ] **Origens permitidas configuradas** - Apenas seus domínios na lista branca
- [ ] **Proprietário de comando definido** - Comandos privilegiados restritos ao proprietário
- [ ] **Sem segredos na configuração** - Chaves API em variáveis de ambiente ou armazenamento seguro
- [ ] **Validação de configuração** - `openclaw doctor` passa sem erros
- [ ] **Logs de auditoria habilitados** - Rastrear todas as ações administrativas
- [ ] **Limitação de taxa** - Prevenir abuso de API

### Monitoramento e Alertas

- [ ] **Verificações de saúde** - Monitoramento automatizado do status do gateway
- [ ] **Monitoramento de logs** - Alertas sobre erros ou atividade suspeita
- [ ] **Monitoramento de recursos** - Alertas sobre alto uso de CPU/memória/disco
- [ ] **Monitoramento de custos** - Alertas de orçamento Google Cloud e OpenAI
- [ ] **Monitoramento de uptime** - Serviço externo verifica disponibilidade
- [ ] **Verificação de backup** - Testes regulares de restauração

### Conformidade

- [ ] **Política de retenção de dados** - Retenção de logs e backups definida
- [ ] **Controle de acesso** - Apenas pessoal autorizado tem acesso SSH
- [ ] **Plano de resposta a incidentes** - Procedimento documentado para incidentes de segurança
- [ ] **Atualizações de dependências** - Atualizações regulares para patches de segurança
- [ ] **Varredura de vulnerabilidades** - Auditorias de segurança regulares

### Testes de Segurança

```bash
# Testar firewall
nmap -p 1-65535 SEU_IP_VM
# Deve mostrar apenas 22, 80, 443 abertas

# Testar SSL
curl -I https://openclaw.seudominio.com
# Deve retornar 200 OK com certificado válido

# Testar token do gateway
curl -H "Authorization: Bearer token-errado" https://openclaw.seudominio.com
# Deve retornar 401 Unauthorized

# Testar endpoint de saúde
curl https://openclaw.seudominio.com/healthz
# Deve retornar 200 OK

# Verificar se não há segredos nos logs
pm2 logs openclaw-gateway | grep -i "sk-"
# Não deve retornar nada

# Docker: Verificar segurança do contêiner
docker scan ghcr.io/openclaw/openclaw:2026.5.7
```

### Resposta a Incidentes de Segurança

**Se chave API for comprometida:**

1. **Revogar imediatamente** a chave comprometida
2. **Gerar nova chave** no dashboard do provedor
3. **Atualizar configuração**:
   ```bash
   # PM2
   openclaw config set agents.main.auth.openai.apiKey NOVA_CHAVE
   pm2 restart openclaw-gateway
   
   # Docker
   # Atualizar nas variáveis de ambiente do Portainer
   docker service update --force openclaw_openclaw_gateway
   ```
4. **Revisar logs** para uso não autorizado
5. **Monitorar faturamento** para cobranças inesperadas
6. **Documentar incidente** para referência futura

**Se token do gateway for comprometido:**

1. **Gerar novo token**:
   ```bash
   openclaw doctor --generate-gateway-token
   ```
2. **Atualizar configuração**:
   ```bash
   openclaw config set gateway.auth.token NOVO_TOKEN
   ```
3. **Reiniciar gateway**
4. **Revogar todos os dispositivos**:
   ```bash
   openclaw devices clear
   ```
5. **Re-aprovar apenas dispositivos confiáveis**

**Se VM for comprometida:**

1. **Isolar a VM** - Desconectar da rede
2. **Criar snapshot** - Para análise forense
3. **Implantar nova VM** - A partir de imagem limpa
4. **Restaurar do backup** - Usar último backup conhecido bom
5. **Rotacionar todas as credenciais** - Chaves API, tokens, senhas
6. **Revisar logs de acesso** - Identificar fonte da violação
7. **Atualizar medidas de segurança** - Prevenir recorrência

---

## 11. Referência

### Referência Rápida de Comandos

#### Comandos do Método PM2

```bash
# Status e Monitoramento
pm2 status                              # Mostrar todos os processos
pm2 monit                               # Dashboard de monitoramento em tempo real
pm2 logs openclaw-gateway               # Ver logs
pm2 logs openclaw-gateway --lines 100   # Últimas 100 linhas

# Gerenciamento de Processos
pm2 restart openclaw-gateway            # Reiniciar gateway
pm2 stop openclaw-gateway               # Parar gateway
pm2 delete openclaw-gateway             # Remover do PM2
pm2 save                                # Salvar lista de processos
pm2 startup                             # Habilitar auto-início na inicialização

# Logs
pm2 flush                               # Limpar todos os logs
pm2 logs --err                          # Mostrar apenas erros

# Sistema
sudo systemctl status caddy             # Verificar status do Caddy
sudo systemctl restart caddy            # Reiniciar Caddy
sudo journalctl -u caddy -f             # Seguir logs do Caddy
```

#### Comandos do Método Docker

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
- Rollback integrado
- Gerenciamento visual (Portainer)
- Ambiente isolado
- Atualizações rápidas

**Contras:**
- Configuração mais complexa
- Overhead da camada Docker
- Requer conhecimento de contêineres

### Padrões Comuns de Erros

| Mensagem de Erro | Causa Provável | Correção Rápida |
|---------------|--------------|-----------|
| `origin not allowed` | CORS não configurado | Adicionar domínio a `allowedOrigins` |
| `gateway token missing` | Token não inserido | Inserir token do arquivo de configuração |
| `device pairing required` | Novo dispositivo | Aprovar com `openclaw devices approve` |
| `No API key found` | Chave API não definida | Configurar com `openclaw config set` |
| `HTTP ERROR 502` | Gateway não está rodando | Verificar logs, reiniciar serviço |
| `bash: openclaw: command not found` | CLI não está no PATH (Docker) | Criar alias ou usar `docker exec` |
| `Proxy headers from untrusted` | IP do Traefik não confiável | Adicionar a `trustedProxies` |
| `JSON5: invalid character` | Edição manual de config | Restaurar backup, usar apenas CLI |
| `gateway.mode is unset` | Modo não configurado | Definir como `local` |
| `404 Not Found` | Problema de volume/caminho | Verificar permissões e caminhos |

### Benchmarks de Performance

| Métrica | e2-standard-2 | e2-standard-4 | Notas |
|--------|---------------|---------------|-------|
| **Agentes Concorrentes** | 2-3 | 5-8 | Depende da complexidade |
| **Tempo de Resposta** | <1s | <500ms | Consultas simples |
| **Uso de Memória** | 2-4 GB | 4-8 GB | Com agentes ativos |
| **Uso de CPU** | 40-60% | 20-40% | Sob carga |
| **Usuários Recomendados** | 1-5 | 5-20 | Usuários concorrentes |

### Estimativas de Custo (Mensal)

| Componente | Faixa de Custo | Notas |
|-----------|------------|-------|
| **VM GCP (e2-standard-2)** | $50-70 | Região us-central1 |
| **VM GCP (e2-standard-4)** | $100-140 | Região us-central1 |
| **IP Estático** | $3-5 | Se não anexado a VM em execução |
| **Disco (50 GB SSD)** | $8-10 | Disco persistente padrão |
| **API OpenAI** | $10-200+ | Altamente variável por uso |
| **Cloudflare** | $0 | Plano gratuito suficiente |
| **Domínio** | $10-15/ano | Varia por registrador |
| **Total (Pequeno)** | $70-100 | e2-standard-2 + uso leve de API |
| **Total (Médio)** | $150-300 | e2-standard-4 + uso moderado de API |

### Suporte e Comunidade

| Recurso | Link |
|----------|------|
| **GitHub Issues** | https://github.com/openclaw/openclaw/issues |
| **Comunidade Discord** | Verificar site do OpenClaw |
| **Documentação** | https://docs.openclaw.ai/ |
| **Notas de Lançamento** | https://github.com/openclaw/openclaw/releases |

---

## 📝 Notas Finais

### Melhores Práticas

1. **Sempre faça backup antes de mudanças** - Configuração, atualizações ou mudanças importantes
2. **Use `openclaw config set`** - Nunca edite `openclaw.json` manualmente
3. **Monitore custos de API** - Defina limites de gastos em todos os provedores
4. **Mantenha segredos seguros** - Nunca faça commit de chaves API no git
5. **Teste em staging primeiro** - Se possível, teste atualizações antes da produção
6. **Documente personalizações** - Mantenha notas sobre sua configuração específica
7. **Auditorias de segurança regulares** - Revise logs de acesso e permissões mensalmente
8. **Automatize backups** - Configure cron jobs para backups diários
9. **Monitore uso de recursos** - Configure alertas para alto uso de CPU/memória/disco
10. **Mantenha-se atualizado** - Acompanhe lançamentos do OpenClaw para patches de segurança

### Obtendo Ajuda

Se você encontrar problemas não cobertos neste guia:

1. **Verifique os logs primeiro** - A maioria dos problemas é visível nos logs
2. **Execute diagnósticos** - `openclaw doctor` detecta problemas comuns
3. **Pesquise issues no GitHub** - Alguém pode ter tido o mesmo problema
4. **Consulte a documentação oficial** - Documentação é atualizada regularmente
5. **Pergunte à comunidade** - Discord ou discussões no GitHub
6. **Forneça contexto** - Inclua logs, config (oculte segredos) e passos para reproduzir

### Contribuindo

Encontrou um erro neste guia ou tem melhorias? Contribuições são bem-vindas!

---

**Última Atualização:** 2026-05-15  
**Versão OpenClaw:** 2026.5.7  
**Versão do Guia:** 2.0

---

Feito com ❤️ para a comunidade OpenClaw
