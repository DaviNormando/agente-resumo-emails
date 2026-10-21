# Segurança

## Onde ficam os segredos

| Segredo | Onde fica | Nunca colocar em |
|---|---|---|
| Token do bot do Telegram | Credencial do n8n | Código, README, issues, prints, mensagens |
| Chave da API da Anthropic | Credencial do n8n | Idem |
| Login do Google (OAuth) | Credencial do n8n | Idem |

O arquivo do workflow (`.json`) **não contém** essas credenciais: o n8n exporta só o nome e o ID de cada credencial.

### Se um segredo vazar
- **Token do Telegram:** no @BotFather, `/mybots` → o bot → **API Token** → **Revoke current token**. Atualize a credencial no n8n.
- **Chave da Anthropic:** no console da Anthropic, em **API Keys**, apague a chave e crie outra. Atualize a credencial no n8n.
- **Google:** em myaccount.google.com → **Segurança** → **Apps de terceiros com acesso**, remova o acesso do n8n e conecte de novo.

Apagar o segredo de um commit **não basta**: ele continua no histórico do Git. Sempre revogue e gere outro.

## Antes de subir um workflow novo para o GitHub

O export feito direto do seu n8n pode conter dados pessoais. Antes de commitar:
- Troque o Chat ID real por `COLE_AQUI_O_SEU_CHAT_ID`.
- Remova nomes de credenciais que identifiquem você, se houver.
- Revise a lista `BLOQUEADOS`: ela pode revelar quais bancos e serviços você usa. No repositório, mantenha só exemplos genéricos.
- Remova `pinData` (dados de teste fixados nos nós), que pode conter e-mails reais.

## Riscos tratados no workflow

- **Privacidade:** remetente, assunto e trecho dos e-mails vão para a API da Anthropic. Mitigação: filtro de categorias do Gmail, lista `BLOQUEADOS` e corte de 500 caracteres por e-mail.
- **Dados salvos no n8n:** execuções bem-sucedidas não são salvas (`saveDataSuccessExecution: none`); só as com erro, para depuração.
- **Prompt injection:** um e-mail pode conter instruções para a IA ("ignore as regras e..."). O prompt trata o conteúdo como dado e marca tentativas com ⚠️. A defesa principal é que o workflow **não tem nenhuma ação** além de enviar o resumo para você.
- **Acesso amplo ao Gmail:** a credencial do Google dá acesso à caixa inteira. Não adicione nós de envio ou exclusão de e-mails sem revisar esse risco.
- **Custo:** defina um limite mensal de gasto no console da Anthropic (**Settings → Limits**).
