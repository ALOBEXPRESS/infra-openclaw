# Checklist Pré-Instalação do OpenClaw

Complete este checklist antes de iniciar sua instalação do OpenClaw para garantir uma implantação tranquila.

## 📋 Requisitos Gerais

### Configuração de Contas

- [ ] **Conta Google Cloud Platform** criada
  - [ ] Faturamento habilitado
  - [ ] API Compute Engine habilitada
  - [ ] Alertas de orçamento configurados (recomendado: $100-200/mês)
  - [ ] Projeto criado e selecionado

- [ ] **Domínio registrado** e acessível
  - [ ] Domínio: `_________________`
  - [ ] Subdomínio planejado: `openclaw.seudominio.com.br`
  - [ ] Acesso ao gerenciamento DNS confirmado

- [ ] **Conta OpenAI** criada
  - [ ] Chave API gerada: `sk-...`
  - [ ] Método de pagamento adicionado
  - [ ] Limites de uso definidos (recomendado: $10-50/mês inicialmente)
  - [ ] Saldo de créditos verificado

### Ferramentas Locais

- [ ] **Cliente SSH** instalado
  - [ ] Windows: PuTTY ou Windows Terminal
  - [ ] Mac/Linux: Terminal integrado
  - [ ] Chave SSH gerada (opcional mas recomendado)

- [ ] **gcloud CLI** instalado (opcional)
  - [ ] Download: https://cloud.google.com/sdk/docs/install
  - [ ] Autenticado: `gcloud auth login`
  - [ ] Projeto definido: `gcloud config set project PROJECT_ID`

- [ ] **Editor de texto** pronto
  - [ ] VS Code, Sublime ou similar
  - [ ] Para editar arquivos de configuração localmente

## 🔀 Requisitos Específicos por Método

### Para Método VM Tradicional + PM2

- [ ] **Conta Cloudflare** criada
  - [ ] Domínio adicionado ao Cloudflare
  - [ ] Nameservers atualizados no registrador
  - [ ] Propagação DNS confirmada (24-48 horas)
  - [ ] Modo SSL/TLS definido como "Full"

- [ ] **Conhecimento básico de Linux**
  - [ ] Confortável com linha de comando
  - [ ] Sabe usar nano ou vim
  - [ ] Entende permissões de arquivo
  - [ ] Sabe usar SSH

### Para Método Docker Swarm

- [ ] **Conhecimento de Docker**
  - [ ] Entende contêineres
  - [ ] Familiarizado com Docker Compose
  - [ ] Conhece comandos básicos Docker
  - [ ] Entende volumes e redes

- [ ] **Configuração Traefik** (se ainda não estiver rodando)
  - [ ] Traefik instalado e rodando
  - [ ] Let's Encrypt configurado
  - [ ] Rede externa criada: `network_swarm_public`
  - [ ] Domínio de teste funcionando com Traefik

- [ ] **Docker Swarm inicializado**
  - [ ] Modo Swarm habilitado: `docker swarm init`
  - [ ] Nó manager confirmado
  - [ ] Nós worker adicionados (se multi-nó)

- [ ] **Portainer instalado** (opcional mas recomendado)
  - [ ] Portainer rodando
  - [ ] Pode acessar UI do Portainer
  - [ ] Familiarizado com interface Portainer

## 🔑 Chaves API e Tokens

Reúna estes antes de começar:

### Obrigatórios

- [ ] **Chave API OpenAI**
  - Obter em: https://platform.openai.com/api-keys
  - Formato: `sk-...`
  - Armazenado com segurança: `_________________`

- [ ] **Token do Gateway** (será gerado durante a configuração)
  - Será auto-gerado
  - Manter seguro após geração

### Opcionais (baseado em suas necessidades)

- [ ] **Chave API Google Places**
  - Obter em: https://console.cloud.google.com/apis/credentials
  - API habilitada: Places API
  - Restrições configuradas
  - Armazenado com segurança: `_________________`

- [ ] **Token de Integração Notion**
  - Obter em: https://www.notion.so/my-integrations
  - Formato: `secret_...`
  - Páginas compartilhadas com integração
  - Armazenado com segurança: `_________________`

- [ ] **Token do Bot Telegram**
  - Obter de: @BotFather no Telegram
  - Formato: `123456:ABC...`
  - Bot criado e configurado
  - Armazenado com segurança: `_________________`

- [ ] **Chave API Firecrawl**
  - Obter em: https://www.firecrawl.dev/
  - Conta criada
  - Armazenado com segurança: `_________________`

- [ ] **Chave API ElevenLabs**
  - Obter em: https://elevenlabs.io/
  - Conta criada
  - Armazenado com segurança: `_________________`

- [ ] **Token de Acesso Pessoal GitHub**
  - Obter em: https://github.com/settings/tokens
  - Escopos: repo, read:org (mínimo)
  - Armazenado com segurança: `_________________`

## 💻 Especificações da VM

### Configuração Inicial Recomendada

- [ ] **Tipo de Máquina**: e2-standard-2
  - vCPUs: 2
  - RAM: 8 GB
  - Adequado para: 1-5 usuários simultâneos

- [ ] **Sistema Operacional**: Ubuntu 22.04 LTS ou 24.04 LTS
  - [ ] 64-bit
  - [ ] Edição servidor (sem GUI necessária)

- [ ] **Disco**: 50 GB SSD
  - [ ] Disco persistente padrão
  - [ ] Pode ser expandido depois

- [ ] **Região**: Escolha a mais próxima dos seus usuários
  - [ ] us-central1 (Iowa) - Menor custo
  - [ ] us-east1 (South Carolina)
  - [ ] europe-west1 (Bélgica)
  - [ ] asia-southeast1 (Singapura)
  - Selecionado: `_________________`

### Regras de Firewall

- [ ] **Porta 22** (SSH) - Aberta
  - [ ] Restrita ao seu IP (recomendado)
  - [ ] Ou aberta para 0.0.0.0/0 (menos seguro)

- [ ] **Porta 80** (HTTP) - Aberta
  - [ ] Para redirecionamento HTTP para HTTPS

- [ ] **Porta 443** (HTTPS) - Aberta
  - [ ] Para tráfego web criptografado

- [ ] **Porta 18789** (OpenClaw Gateway) - **FECHADA**
  - [ ] NÃO deve ser exposta publicamente
  - [ ] Acessível apenas via localhost

## 🌐 Configuração de Rede

### Registros DNS a Criar

- [ ] **Registro A** para subdomínio base
  - Nome: `advanced` (ou sua escolha)
  - Tipo: A
  - Valor: `IP_ESTATICO_DA_VM`
  - TTL: Auto ou 300

- [ ] **Registro CNAME** para OpenClaw
  - Nome: `openclaw`
  - Tipo: CNAME
  - Valor: `advanced.seudominio.com.br`
  - TTL: Auto ou 300

### Configurações Cloudflare (Método PM2)

- [ ] **Status do Proxy**: Proxied (🟠 nuvem laranja)
- [ ] **Modo SSL/TLS**: Full (não Flexible)
- [ ] **Always Use HTTPS**: Habilitado
- [ ] **Automatic HTTPS Rewrites**: Habilitado
- [ ] **Versão TLS Mínima**: 1.2 ou superior

## 📊 Configuração de Monitoramento

### Google Cloud

- [ ] **Alertas de Orçamento** configurados
  - [ ] Alerta em 50% do orçamento
  - [ ] Alerta em 90% do orçamento
  - [ ] Alerta em 100% do orçamento

- [ ] **Monitoramento** habilitado
  - [ ] Alertas de uso de CPU
  - [ ] Alertas de uso de memória
  - [ ] Alertas de uso de disco

### OpenAI

- [ ] **Limites de Uso** definidos
  - [ ] Limite rígido: $_____ por mês
  - [ ] Limite flexível: $_____ por mês
  - [ ] Notificações por email habilitadas

- [ ] **Monitoramento de Uso** configurado
  - [ ] Verificar dashboard de uso semanalmente
  - [ ] Revisar faturamento mensalmente

## 🔒 Preparação de Segurança

### Senhas e Tokens

- [ ] **Senhas fortes** preparadas
  - [ ] Mínimo 16 caracteres
  - [ ] Mix de letras, números, símbolos
  - [ ] Armazenadas em gerenciador de senhas

- [ ] **Chave SSH** gerada (recomendado)
  - [ ] Chave pública: `~/.ssh/id_rsa.pub`
  - [ ] Chave privada: `~/.ssh/id_rsa`
  - [ ] Protegida com passphrase

### Estratégia de Backup

- [ ] **Local de backup** decidido
  - [ ] Local: `_________________`
  - [ ] Nuvem: `_________________`
  - [ ] Criptografado: Sim / Não

- [ ] **Cronograma de backup** planejado
  - [ ] Backups automáticos diários
  - [ ] Verificação manual semanal
  - [ ] Testes de restauração mensais

## 📚 Documentação

- [ ] **Guia de instalação** revisado
  - [ ] README.md principal lido
  - [ ] Método escolhido: PM2 / Docker
  - [ ] Passos compreendidos

- [ ] **Guia de troubleshooting** marcado
  - [ ] Problemas comuns revisados
  - [ ] Canais de suporte identificados

- [ ] **Documento de notas** criado
  - [ ] Para registrar configurações personalizadas
  - [ ] Para documentar decisões
  - [ ] Para rastrear problemas

## ⏱️ Alocação de Tempo

### Tempo Estimado Necessário

**Método PM2:**
- [ ] Configuração da VM: 15 minutos
- [ ] Instalação de software: 15 minutos
- [ ] Configuração OpenClaw: 30 minutos
- [ ] Testes e verificação: 15 minutos
- **Total: 1-1.5 horas**

**Método Docker:**
- [ ] Configuração da VM: 15 minutos
- [ ] Deploy da stack Docker: 10 minutos
- [ ] Configuração OpenClaw: 20 minutos
- [ ] Testes e verificação: 15 minutos
- **Total: 1 hora**

### Agende Sua Instalação

- [ ] **Data**: `_________________`
- [ ] **Hora**: `_________________`
- [ ] **Pessoa de backup** (se equipe): `_________________`
- [ ] **Plano de rollback** preparado: Sim / Não

## ✅ Verificações Finais

Antes de iniciar a instalação:

- [ ] Todas as contas necessárias criadas
- [ ] Todas as chaves API geradas e armazenadas com segurança
- [ ] Especificações da VM decididas
- [ ] Registros DNS planejados
- [ ] Estratégia de backup implementada
- [ ] Tempo alocado
- [ ] Documentação revisada
- [ ] Canais de suporte identificados
- [ ] Equipe notificada (se aplicável)
- [ ] Plano de rollback preparado

## 🚀 Pronto para Instalar?

Se todos os itens estão marcados, você está pronto para prosseguir!

**Próximos Passos:**

1. **Método PM2**: Vá para [README.md - Método 1](README.md#método-1-vm-tradicional-com-pm2-recomendado-para-iniciantes)
2. **Método Docker**: Vá para [README.md - Método 2](README.md#método-2-docker-swarm-com-traefik-recomendado-para-produção)
3. **Ainda decidindo?**: Confira [COMPARISON.md](COMPARISON.md)

---

**Dúvidas?** Revise a [seção de FAQ](README.md#-troubleshooting) ou confira [GitHub Issues](https://github.com/openclaw/openclaw/issues)

**Precisa de ajuda?** Junte-se à comunidade OpenClaw no Discord (link na documentação oficial)
