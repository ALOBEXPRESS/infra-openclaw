# Instalação do OpenClaw na Google Cloud com subdomínio, Cloudflare e PM2

Guia prático para instalar o **OpenClaw** em uma VM Google Cloud, publicar no subdomínio `openclaw.alobexpress.com.br`, conectar APIs como OpenAI/Whisper, Google Places, Notion, Telegram, Firecrawl e deixar tudo rodando 24/7.

> Este README foi escrito para leigos. Copie e cole os comandos na ordem. Sempre troque valores como `SEU_TOKEN`, `SEU_DOMINIO` e `SUA_CHAVE` pelos seus dados reais.

---

## 1. Visão geral da arquitetura

A arquitetura usada aqui é:

```text
Usuário / navegador
        ↓
https://openclaw.alobexpress.com.br
        ↓
Cloudflare, com nuvem laranja/proxy ativo
        ↓
Google Cloud VM
        ↓
Caddy, portas 80/443
        ↓
OpenClaw Gateway em 127.0.0.1:18789
        ↓
Agentes, Telegram, APIs, browser automation e integrações
```

Por que essa arquitetura é boa:

- O OpenClaw fica em uma VM separada do n8n.
- O dashboard não precisa expor a porta `18789` diretamente.
- A Cloudflare fornece HTTPS e proteção básica.
- O Caddy faz o proxy reverso para o OpenClaw.
- O PM2 mantém o gateway rodando mesmo depois de fechar o terminal.

---

## 2. Custos e cuidados

### Google Cloud

Para começar, uma VM `e2-standard-2` é suficiente para testes e uso inicial:

```text
e2-standard-2
2 vCPU
8 GB RAM
50 GB disco
Ubuntu 22.04 ou 24.04 LTS
```

Se começar a usar muitos agentes ao mesmo tempo, browser automation pesado ou muitas abas/headless browsers, pode subir depois para:

```text
e2-standard-4
4 vCPU
16 GB RAM
100 GB disco
```

No Google Cloud é simples alterar o tipo da VM: pare a VM, edite o tipo de máquina e ligue novamente.

### OpenAI API

A assinatura do ChatGPT Business/Plus/Team não é a mesma coisa que a cobrança da API. A API tem billing separado. Se você recebeu US$5 de crédito, o OpenClaw pode usar esse crédito até acabar. Depois, será necessário adicionar billing/prepaid na plataforma da OpenAI.

Links úteis:

- Usage: https://platform.openai.com/usage
- Billing: https://platform.openai.com/settings/organization/billing/overview
- Limits: https://platform.openai.com/settings/organization/limits

Recomendação: coloque limite baixo no começo, tipo US$5 ou US$10.

---

## 3. Pré-requisitos

Você vai precisar de:

- Conta Google Cloud com Compute Engine habilitado.
- VM Ubuntu na Google Cloud.
- Domínio ou subdomínio configurável, exemplo: `openclaw.alobexpress.com.br`.
- Conta Cloudflare controlando o DNS do domínio.
- Chave API da OpenAI, se for usar modelos OpenAI/Codex/Whisper.
- Opcional: Google Places API, Notion API, Telegram Bot Token, Firecrawl API, ElevenLabs API.

---

## 4. Criando a VM na Google Cloud

Configuração inicial recomendada:

```text
Nome: openclaw-alobexpress
Tipo: e2-standard-2
vCPU: 2
RAM: 8 GB
Disco: 50 GB
Sistema: Ubuntu LTS
Região: us-central1, se quiser custo menor
```

### Firewall da VM

No Google Cloud, permita pelo menos:

```text
HTTP: porta 80
HTTPS: porta 443
SSH: porta 22
```

Evite abrir a porta `18789` publicamente. Ela deve ficar apenas local na VM.

---

## 5. Reservar IP estático

É recomendado reservar um IP externo estático para a VM.

No Google Cloud Console:

```text
VPC Network
→ IP addresses
→ Reserve static address
→ associe à VM openclaw-alobexpress
```

Isso evita que o IP mude e quebre o DNS/Cloudflare.

---

## 6. Configurando DNS na Cloudflare

Exemplo usado:

```text
advanced.alobexpress.com.br → A → IP_DA_VM
openclaw.alobexpress.com.br → CNAME → advanced.alobexpress.com.br
```

Na Cloudflare, deixe a nuvem laranja ativada:

```text
Proxy status: Proxied
```

Isso significa que o tráfego passa pela Cloudflare antes de chegar na VM.

### SSL/TLS na Cloudflare

Vá em:

```text
Cloudflare
→ SSL/TLS
→ Overview
→ Full
```

Use **Full**, não `Flexible`.

Observação: quando o proxy laranja está ativo, ferramentas como DNS Checker podem mostrar IPs da Cloudflare em vez do IP real da VM. Isso é normal.

---

## 7. Acessando a VM

Pelo Google Cloud Console, clique em:

```text
Compute Engine
→ VM instances
→ SSH
```

Ou pelo terminal local com gcloud:

```bash
gcloud config set project SEU_PROJECT_ID

gcloud compute ssh openclaw-alobexpress --zone=us-central1-c
```

Troque a zona se sua VM estiver em outra.

---

## 8. Atualizar a VM e instalar ferramentas básicas

Dentro da VM:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl ca-certificates gnupg git build-essential unzip nano
```

Se o `nano` não existir, instale:

```bash
sudo apt install -y nano
```

---

## 9. Instalar Node.js

O OpenClaw recomenda Node moderno. Use Node 24:

```bash
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt install -y nodejs
```

Confira:

```bash
node -v
npm -v
```

Se aparecer Node 24 ou Node 22.14+, está ok.

---

## 10. Instalar o OpenClaw

### Método recomendado pelo instalador

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Alternativa com npm

Se você já tem Node funcionando:

```bash
sudo npm install -g openclaw@latest
```

Confira:

```bash
openclaw --version
openclaw doctor
```

---

## 11. Setup inicial do OpenClaw

Rode:

```bash
openclaw setup
```

Depois rode o wizard/configuração:

```bash
openclaw setup --wizard
```

ou:

```bash
openclaw configure
```

Durante o setup, configure:

```text
Gateway: local
Modelo/Provider: OpenAI, Claude, Gemini etc.
Canais: Telegram, WhatsApp etc., se quiser
Plugins/skills: conforme necessidade
```

O arquivo principal de configuração fica em:

```text
~/.openclaw/openclaw.json
```

Se você instalou/rodou como root, fica em:

```text
/root/.openclaw/openclaw.json
```

Atenção: use sempre o mesmo usuário. Se configurou como `root`, rode como `root`. Se configurou como `jonat`, rode como `jonat`.

---

## 12. Configurar token do Gateway

Abra o arquivo:

```bash
nano ~/.openclaw/openclaw.json
```

Ou, se estiver usando root:

```bash
sudo nano /root/.openclaw/openclaw.json
```

Procure algo parecido com:

```json
"gateway": {
  "auth": {
    "token": "SEU_TOKEN_AQUI"
  }
}
```

Esse token é o que você coloca no dashboard do OpenClaw no campo:

```text
Token do Gateway
```

Não use uma senha aleatória diferente se o OpenClaw já gerou token no `openclaw.json`. Use o token que está no arquivo.

---

## 13. Liberar o domínio público no OpenClaw

Quando você acessa por domínio, o OpenClaw pode bloquear com erro:

```text
origin not allowed
```

Para resolver, dentro de `~/.openclaw/openclaw.json`, deixe a parte `controlUi` assim:

```json
"controlUi": {
  "allowInsecureAuth": true,
  "allowedOrigins": [
    "https://openclaw.alobexpress.com.br"
  ]
}
```

Exemplo dentro de `gateway`:

```json
"gateway": {
  "auth": {
    "token": "SEU_TOKEN_AQUI"
  },
  "controlUi": {
    "allowInsecureAuth": true,
    "allowedOrigins": [
      "https://openclaw.alobexpress.com.br"
    ]
  }
}
```

Atenção ao JSON:

- Precisa de vírgula entre blocos.
- Não pode ter vírgula sobrando no último item.
- Se der erro de JSON, valide com:

```bash
node -e "JSON.parse(require('fs').readFileSync(process.env.HOME + '/.openclaw/openclaw.json','utf8')); console.log('JSON OK')"
```

Se estiver como root:

```bash
node -e "JSON.parse(require('fs').readFileSync('/root/.openclaw/openclaw.json','utf8')); console.log('JSON OK')"
```

---

## 14. Instalar e configurar Caddy

O Caddy será o proxy reverso. Ele recebe `https://openclaw.alobexpress.com.br` e encaminha para `127.0.0.1:18789`.

Instale:

```bash
sudo apt update
sudo apt install -y caddy
```

Edite o Caddyfile:

```bash
sudo nano /etc/caddy/Caddyfile
```

Coloque:

```caddy
openclaw.alobexpress.com.br {
    reverse_proxy 127.0.0.1:18789
}
```

Salve e reinicie:

```bash
sudo systemctl restart caddy
sudo systemctl status caddy
```

Se não quiser usar nano, pode criar o arquivo assim:

```bash
sudo tee /etc/caddy/Caddyfile > /dev/null <<'EOF'
openclaw.alobexpress.com.br {
    reverse_proxy 127.0.0.1:18789
}
EOF

sudo systemctl restart caddy
sudo systemctl status caddy
```

---

## 15. Iniciar o Gateway manualmente para testar

Rode:

```bash
openclaw gateway --allow-unconfigured
```

Se estiver tudo certo, você verá mensagens parecidas com:

```text
gateway ready
http server listening
browser control listening
heartbeat started
```

Agora abra no navegador:

```text
https://openclaw.alobexpress.com.br
```

Se pedir token, coloque o token que está em:

```text
~/.openclaw/openclaw.json
```

---

## 16. Aprovar o dispositivo no primeiro acesso

No primeiro acesso, pode aparecer:

```text
device pairing required
```

Isso é normal. É uma proteção do OpenClaw.

Em outro terminal da VM, rode:

```bash
openclaw devices list
```

Copie o `Request ID` pendente e aprove:

```bash
openclaw devices approve REQUEST_ID_AQUI
```

Exemplo:

```bash
openclaw devices approve 1e0ea8bf-2b7d-41ac-aca4-42e64f78ec70
```

Importante: o request expira/troca rápido. Se der `unknown requestId`, volte no navegador, clique conectar de novo, rode `openclaw devices list` de novo e aprove o ID novo imediatamente.

---

## 17. Deixar o OpenClaw rodando para sempre com PM2

Instale o PM2:

```bash
sudo npm install -g pm2
```

Inicie o gateway com PM2:

```bash
pm2 start "openclaw gateway --allow-unconfigured" --name openclaw-gateway
```

Salve o estado:

```bash
pm2 save
```

Ative no boot:

```bash
pm2 startup
```

O PM2 vai imprimir um comando com `sudo env ...`. Copie e execute exatamente o comando que ele mostrar.

Depois confira:

```bash
pm2 status
pm2 logs openclaw-gateway
```

Comandos úteis:

```bash
pm2 restart openclaw-gateway
pm2 stop openclaw-gateway
pm2 delete openclaw-gateway
pm2 logs openclaw-gateway
pm2 monit
```

Se alterar `openclaw.json`, reinicie:

```bash
pm2 restart openclaw-gateway
```

---

## 18. Configurar OpenAI API

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

### Custos da OpenAI

- ChatGPT Business e OpenAI API têm billing separado.
- API é cobrada por uso.
- Monitore em: https://platform.openai.com/usage
- Configure limites em: https://platform.openai.com/settings/organization/limits

---

## 19. Configurar Whisper/transcrição

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

---

## 20. Configurar Google Places API

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

---

## 21. Configurar Notion API

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

---

## 22. Configurar Telegram

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

---

## 23. Configurar Firecrawl / Fireclaw Search

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

---

## 24. Configurar ElevenLabs

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

## 25. Testes finais

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

## 26. Troubleshooting — erros comuns

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

## 27. Backup

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

---

## 28. Atualizar OpenClaw

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

---

## 29. Checklist de segurança

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

## 30. Comandos finais mais usados

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

---

## 31. Fontes oficiais úteis

- Instalação OpenClaw: https://docs.openclaw.ai/install
- OpenClaw no GCP: https://docs.openclaw.ai/install/gcp
- Control UI: https://docs.openclaw.ai/web/control-ui
- Dashboard: https://docs.openclaw.ai/web/dashboard
- Remote access: https://docs.openclaw.ai/gateway/remote
- Devices CLI: https://docs.openclaw.ai/cli/devices
- Node.js para OpenClaw: https://docs.openclaw.ai/install/node
- OpenAI API usage: https://platform.openai.com/usage
- OpenAI billing: https://platform.openai.com/settings/organization/billing/overview

---

## 32. Resumo para lembrar

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
