# Bot do Clima - Telegram com n8n

Chatbot no Telegram que informa a temperatura atual de qualquer cidade do Brasil. O usuário envia o nome da cidade e recebe uma resposta amigável com a temperatura em graus Celsius, consumindo a API gratuita do OpenWeather.

## Exemplo de uso

```
Usuário:  São Paulo,SP,BR
Bot:      🌤️ A temperatura em São Paulo é de 24°C.

Usuário:  cidadeinvalida
Bot:      ❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).
```

---

## Pré-requisitos

- [Docker](https://www.docker.com/) instalado
- Bot criado no Telegram via [@BotFather](https://t.me/BotFather)
- Conta e API Key na [OpenWeather](https://home.openweathermap.org/api_keys)

---

## Variáveis necessárias

| Variável | Onde obter |
|---|---|
| `TELEGRAM_BOT_TOKEN` | [@BotFather](https://t.me/BotFather) → `/newbot` |
| `OPENWEATHER_API_KEY` | [openweathermap.org/api_keys](https://home.openweathermap.org/api_keys) |

> ⚠️ **Nunca suba essas chaves no repositório.** Configure apenas dentro do n8n e no `.env` local (já no `.gitignore`).

---

## Como executar o ambiente

### 1. Configure o `.env`

Crie um arquivo `.env` na raiz do projeto (ou edite o existente):

```env
POSTGRES_USER=n8n
POSTGRES_PASSWORD=SUA_SENHA_AQUI
POSTGRES_DB=n8n
N8N_ENCRYPTION_KEY=SUA_CHAVE_32_CHARS_AQUI
N8N_HOST=localhost
OPENWEATHER_API_KEY=SUA_CHAVE_OPENWEATHER_AQUI
TELEGRAM_BOT_TOKEN=SEU_TOKEN_TELEGRAM_AQUI
```

### 2. Suba os containers

```bash
docker compose up -d
```

Acesse o n8n em: **http://localhost:5678**

> **Para receber webhooks do Telegram localmente**, o n8n precisa ser acessível pela internet.
> Use [ngrok](https://ngrok.com/) para expor a porta:
> ```bash
> ngrok http 5678
> ```
> Depois atualize `N8N_HOST` e `WEBHOOK_URL` no `.env` com o endereço gerado pelo ngrok e reinicie os containers.

---

## Como importar o workflow no n8n

1. Acesse **http://localhost:5678** e faça login
2. No menu lateral, clique em **Workflows**
3. Clique em **Import from file**
4. Selecione o arquivo `workflow-chatbot-telegram.json`
5. O workflow será importado — configure as credenciais conforme abaixo

---

## Como configurar as credenciais no n8n

### Telegram Bot Token

1. No n8n, vá em **Settings → Credentials → New Credential**
2. Busque por **Telegram API**
3. No campo **Access Token**, insira seu `TELEGRAM_BOT_TOKEN`
4. Salve como **"Telegram Bot Token"**
5. No workflow importado, os nós **Telegram Trigger**, **Enviar Temperatura** e **Enviar Erro** já referenciam essa credential — apenas selecione a que você criou

### OpenWeather API Key

A chave é lida automaticamente via variável de ambiente `$env.OPENWEATHER_API_KEY`, que já é passada ao container n8n pelo `docker-compose.yml`. Basta ter o valor correto no `.env`.

---

## Como ativar e testar o chatbot

1. Com o workflow importado e as credenciais configuradas, clique em **Save** e depois no toggle **Inactive → Active**
2. O n8n irá registrar o webhook automaticamente no Telegram
3. Abra o Telegram e envie uma mensagem para o seu bot:
   - `São Paulo,SP,BR` → deve retornar a temperatura atual
   - `Belo Horizonte,MG,BR` → deve retornar a temperatura atual
   - `Florianópolis,SC,BR` → deve retornar a temperatura atual
   - `cidadexyz` → deve retornar a mensagem de erro

---

## Estrutura do workflow

```
[Telegram Trigger]
       ↓
[Formatar Entrada]  ← normaliza texto → variável `queue`
       ↓
[OpenWeather API]   ← GET /data/2.5/weather?q={queue}&units=metric&lang=pt_br
       ↓
[Resposta Válida?]  ← IF cod == 200
    ↓ SIM                     ↓ NÃO
[Formatar Mensagem]      [Enviar Erro]
    ↓                    ❌ Cidade não encontrada...
[Enviar Temperatura]
🌤️ A temperatura em X é de Y°C.
```

---

## Arquivos do repositório

| Arquivo | Descrição |
|---|---|
| `workflow-chatbot-telegram.json` | Workflow exportado do n8n (sem credenciais) |
| `docker-compose.yml` | Stack Docker com n8n, PostgreSQL e Redis |
| `README.md` | Este arquivo |
| `.env` | **NÃO commitado** — variáveis sensíveis locais |
