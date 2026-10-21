# Agente de resumo de e-mails

Workflow do [n8n](https://n8n.io) que lê os e-mails do Gmail das últimas 24 horas, usa o Claude (Anthropic) para classificar e resumir, e entrega o resumo todo dia às 7h no Telegram.

```
Todo dia às 7h → Gmail (só leitura) → Filtro de remetentes → Claude Haiku → Telegram
```

Exemplo de mensagem:

```
☀️ Resumo dos e-mails (21/10)

🔴 URGENTE
Banco X — fatura vence amanhã; pagar hoje.

🟡 RESPONDER
Ana — pergunta se a reunião pode ser às 10h.

🔵 INFORMATIVO
GitHub — novo login detectado na sua conta (foi você).

⚪ PODE IGNORAR
6 e-mails: Amazon, LinkedIn e 4 newsletters.

👉 Prioridade do dia: pagar a fatura do Banco X.

📊 9 e-mails lidos · 1 filtrado por segurança
```
*(exemplo ilustrativo)*

## Como funciona

| Nó | O que faz |
|---|---|
| **Todo dia às 7h** | Gatilho de horário (fuso `America/Sao_Paulo`). |
| **Buscar e-mails (24h)** | Busca no Gmail com `newer_than:1d -category:promotions -category:social`. Só leitura. |
| **Preparar e-mails** | Código JavaScript: remove remetentes da lista `BLOQUEADOS`, corta cada trecho em 500 caracteres e junta tudo num texto só. |
| **IA: classificar e resumir** | Basic LLM Chain com o Claude Haiku. Classifica cada e-mail em urgente, responder, informativo ou pode ignorar. |
| **Enviar no Telegram** | Envia o resumo (até 3.800 caracteres) por um bot. |

É um *workflow com IA*, não um agente autônomo: o caminho é fixo e a IA não executa ações. Isso é intencional (ver [segurança](docs/seguranca.md)).

## Requisitos

- n8n (Cloud ou auto-hospedado)
- Conta Google com Gmail
- Chave da API da Anthropic, com créditos
- Bot do Telegram (criado com o [@BotFather](https://t.me/BotFather)) e o seu Chat ID

## Instalação

1. **Importe o workflow:** no n8n, crie um workflow novo → menu **⋯** → **Import from File** → `workflows/resumo-emails-v1.json`.
2. **Conecte as credenciais** (elas ficam no cofre do n8n, nunca no código):
   - **Buscar e-mails (24h):** credencial Gmail OAuth2 (no n8n Cloud, **Sign in with Google**).
   - **Claude:** credencial da Anthropic. Escolha o Claude Haiku mais recente no campo *Model*.
   - **Enviar no Telegram:** credencial com o token do bot.
3. **Chat ID:** no nó **Enviar no Telegram**, troque `COLE_AQUI_O_SEU_CHAT_ID` pelo seu Chat ID.
   - Para descobrir: envie uma mensagem ao seu bot e abra `https://api.telegram.org/bot<TOKEN>/getUpdates`. O número fica em `"chat":{"id": ...}`.
4. **Remetentes sensíveis:** edite a lista `BLOQUEADOS` no nó **Preparar e-mails**.
5. **Teste:** clique em **Execute workflow** e confira a saída de cada nó.
6. **Ative:** ligue a chave **Active**.

## Personalização

| O quê | Onde |
|---|---|
| Horário | Nó *Todo dia às 7h* → *Trigger at Hour* |
| Quais e-mails entram | Nó *Buscar e-mails* → *Filters → Search* (sintaxe da busca do Gmail) |
| Remetentes que não vão para a IA | Lista `BLOQUEADOS` no nó *Preparar e-mails* |
| Formato do resumo | Mensagem do sistema no nó *IA: classificar e resumir* |

## Segurança

Leia [docs/seguranca.md](docs/seguranca.md). Resumo:
- Nenhuma senha, token ou chave fica no repositório; tudo fica nas credenciais do n8n.
- O workflow só lê e-mails. Ele não envia, não apaga e não responde nada.
- O prompt trata o conteúdo dos e-mails como dado, não como instrução (proteção contra *prompt injection*).

## Roadmap

- [x] v1: resumo diário, só leitura
- [ ] v2: agente com ferramentas (criar rascunho de resposta, aplicar etiquetas, criar nota no Obsidian)
- [ ] Ler o e-mail completo só dos marcados como urgentes ou para responder
- [ ] Alerta no Telegram quando o workflow falhar

## Stack

n8n · Claude Haiku (API da Anthropic) · Gmail API · Telegram Bot API
