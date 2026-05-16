# Guia de Início Rápido do OpenClaw

Escolha seu método de implantação e siga o guia rápido apropriado.

## 🚀 Método 1: VM Tradicional + PM2 (Iniciantes)

**Tempo:** 30-45 minutos  
**Dificuldade:** Fácil  
**Melhor para:** Aprendizado, equipes pequenas, instância única

### Pré-requisitos
- Conta Google Cloud
- Domínio com Cloudflare
- Chave API da OpenAI

### Passos Rápidos

1. **Criar VM** (e2-standard-2, Ubuntu 22.04)
2. **Reservar IP estático** e configurar DNS
3. **Instalar Node.js 24**:
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
   sudo apt install -y nodejs
   ```
4. **Instalar OpenClaw**:
   ```bash
   curl -fsSL https://openclaw.ai/install.sh | bash
   openclaw setup --wizard
   ```
5. **Instalar Caddy**:
   ```bash
   sudo apt install -y caddy
   echo "openclaw.seudominio.com.br { reverse_proxy 127.0.0.1:18789 }" | sudo tee /etc/caddy/Caddyfile
   sudo systemctl restart caddy
   ```
6. **Instalar PM2**:
   ```bash
   sudo npm install -g pm2
   pm2 start "openclaw gateway --allow-unconfigured" --name openclaw-gateway
   pm2 save
   pm2 startup
   ```
7. **Configurar CORS**:
   ```bash
   openclaw config set --batch-json '[{"path":"gateway.controlUi.allowedOrigins","value":["https://openclaw.seudominio.com.br"]}]'
   pm2 restart openclaw-gateway
   ```
8. **Acessar dashboard** em `https://openclaw.seudominio.com.br`

[Documentação Completa →](README.md#método-1-vm-tradicional-com-pm2-recomendado-para-iniciantes)

---

## 🐳 Método 2: Docker Swarm + Traefik (Produção)

**Tempo:** 15-20 minutos (se Swarm já estiver pronto)  
**Dificuldade:** Média  
**Melhor para:** Produção, escalabilidade, equipes DevOps

### Pré-requisitos
- Docker Swarm inicializado
- Traefik rodando com Let's Encrypt
- Rede `network_swarm_public` criada
- Chave API da OpenAI

### Passos Rápidos

1. **Preparar diretório**:
   ```bash
   mkdir -p /opt/infra/alobexpress/openclaw
   chown -R 1000:1000 /opt/infra/alobexpress/openclaw
   ```

2. **Criar docker-compose.yml** (veja arquivo completo no README principal)

3. **Fazer deploy da stack**:
   ```bash
   docker stack deploy -c docker-compose.yml openclaw
   ```

4. **Criar alias do CLI**:
   ```bash
   echo 'alias openclaw="docker exec -it \$(docker ps --filter name=openclaw -q) node dist/index.js"' >> ~/.bashrc
   source ~/.bashrc
   ```

5. **Configurar CORS e proxies**:
   ```bash
   openclaw config set --batch-json '[
     {"path":"gateway.controlUi.allowedOrigins","value":["https://openclaw.seudominio.com.br"]},
     {"path":"gateway.trustedProxies","value":["IP_DO_TRAEFIK"]}
   ]'
   ```

6. **Definir modo do gateway**:
   ```bash
   openclaw config set gateway.mode local
   ```

7. **Acessar dashboard** em `https://openclaw.seudominio.com.br`

[Documentação Completa →](README.md#método-2-docker-swarm-com-traefik-recomendado-para-produção)

---

## 🆘 Problemas Comuns

| Problema | Solução Rápida |
|----------|----------------|
| `origin not allowed` | Adicione o domínio em `allowedOrigins` |
| `502 Bad Gateway` | Verifique se o gateway está rodando: `pm2 status` ou `docker service ps` |
| `device pairing required` | Execute `openclaw devices approve REQUEST_ID` |
| `No API key found` | Configure com `openclaw config set agents.main.auth.openai.apiKey sk-...` |

[Troubleshooting Completo →](README.md#-troubleshooting)

---

## 📚 Comandos Essenciais

### Método PM2
```bash
pm2 status                    # Verificar status
pm2 logs openclaw-gateway     # Ver logs
pm2 restart openclaw-gateway  # Reiniciar
openclaw doctor               # Diagnóstico
```

### Método Docker
```bash
docker service ps openclaw_openclaw_gateway              # Verificar status
docker service logs -f openclaw_openclaw_gateway         # Ver logs
docker service update --force openclaw_openclaw_gateway  # Reiniciar
openclaw doctor                                          # Diagnóstico
```

---

## 🔗 Links Rápidos

- [README Completo](README.md) - Documentação completa
- [Visão Geral da Arquitetura](README.md#-visão-geral-da-arquitetura)
- [Integrações de API](README.md#-integrações-de-api)
- [Checklist de Segurança](README.md#-checklist-de-segurança)
- [Documentação Oficial](https://docs.openclaw.ai/)

---

**Precisa de ajuda?** Consulte a [seção de Troubleshooting](README.md#-troubleshooting) ou [GitHub Issues](https://github.com/openclaw/openclaw/issues)
