# 🤖 Guia Completo: pyTelegramBotAPI (telebot)

A biblioteca `pyTelegramBotAPI` (conhecida como `telebot`) é uma das formas mais simples e populares de criar bots para o Telegram usando Python.

---

## 📑 Sumário
1. [🛠️ Instalação e Token](#-instalação-e-token)
2. [🔄 Bot Síncrono (Básico)](#-bot-síncrrono-básico)
3. [⚡ Bot Assíncrono (Alta Performance)](#-bot-assíncrono-alta-performance)
4. [⌨️ Teclados e Interatividade](#-teclados-e-interatividade)
5. [📦 Tipos de Mensagens, Filtros e Handlers](#-tipos-de-mensagens-filtros-e-handlers)
6. [📌 Webhooks e Polling](#-webhooks-e-polling)
7. [🧠 Estado, Conversas e Fluxo](#-estado-conversas-e-fluxo)
8. [🛠️ Mensagens Avançadas](#-mensagens-avançadas)
9. [🧾 Erros, Logs e Retry](#-erros-logs-e-retry)
10. [⚙️ Boas Práticas de Produção](#-boas-práticas-de-produção)
11. [📚 Extras Úteis](#-extras-úteis)
12. [💡 Conclusão](#-conclusão)

---

## 🛠️ Instalação e Token

### Instalação
```bash
pip install pyTelegramBotAPI
```

### Obtendo o Token
1. Converse com o [@BotFather](https://t.me/botfather) no Telegram.
2. Use o comando `/newbot`.
3. Siga as instruções para nome e username.
4. Guarde o **Token API** fornecido.

> Nunca compartilhe o token publicamente.

---

## 🔄 Bot Síncrono (Básico)

O bot síncrono é simples e ideal para fluxos mais leves.

```python
import telebot

TOKEN = 'SEU_TOKEN_AQUI'
bot = telebot.TeleBot(TOKEN)

@bot.message_handler(commands=['start', 'help'])
def send_welcome(message):
    bot.reply_to(message, 'Olá! Eu sou um bot síncrono.')

@bot.message_handler(func=lambda m: m.text and 'oi' in m.text.lower())
def greet(message):
    bot.send_message(message.chat.id, 'Olá! Como posso ajudar?')

bot.infinity_polling(timeout=60, long_polling_timeout=90, non_stop=True)
```

**Dicas:**
- Use `timeout` e `long_polling_timeout` para evitar timeouts longos.
- `non_stop=True` ajuda a manter o bot ativo mesmo se ocorrerem erros transitórios.

---

## ⚡ Bot Assíncrono (Alta Performance)

Use o `AsyncTeleBot` quando precisar de I/O intensivo e alta concorrência.

```python
import asyncio
from telebot.async_telebot import AsyncTeleBot

TOKEN = 'SEU_TOKEN_AQUI'
bot = AsyncTeleBot(TOKEN)

@bot.message_handler(commands=['start'])
async def send_welcome(message):
    await bot.reply_to(message, 'Olá! Eu sou um bot assíncrono.')

@bot.message_handler(commands=['info'])
async def info(message):
    await asyncio.sleep(2)
    await bot.send_message(message.chat.id, 'Informação processada!')

async def main():
    await bot.infinity_polling()

if __name__ == '__main__':
    asyncio.run(main())
```

- O bot assíncrono evita bloqueios em handlers lentos.
- Ideal para chamadas externas, DB e I/O intensivo.

---

## ⌨️ Teclados e Interatividade

### Reply Keyboard
Botões fixos que substituem o teclado do usuário.

```python
from telebot import types

markup = types.ReplyKeyboardMarkup(row_width=2, resize_keyboard=True)
markup.add(types.KeyboardButton('Opção A'), types.KeyboardButton('Opção B'))
bot.send_message(chat_id, 'Escolha uma opção:', reply_markup=markup)
```

### Inline Keyboard
Botões anexados à mensagem.

```python
markup = types.InlineKeyboardMarkup()
btn = types.InlineKeyboardButton('Clique aqui', callback_data='btn_clicked')
markup.add(btn)
bot.send_message(chat_id, 'Botão inline:', reply_markup=markup)

@bot.callback_query_handler(func=lambda call: call.data == 'btn_clicked')
def callback_inline(call):
    bot.answer_callback_query(call.id, 'Você clicou no botão!')
    bot.send_message(call.message.chat.id, 'Ação confirmada.')
```

### URL Button e Login

```python
markup = types.InlineKeyboardMarkup()
markup.add(types.InlineKeyboardButton('Abrir site', url='https://example.com'))
bot.send_message(chat_id, 'Clique abaixo:', reply_markup=markup)
```

### Forçar envio de chat action

```python
bot.send_chat_action(chat_id, 'typing')
```

### Markdown e HTML

```python
bot.send_message(chat_id, '*Negrito* _itálico_', parse_mode='Markdown')
```

---

## 📦 Tipos de Mensagens, Filtros e Handlers

### Comandos

```python
@bot.message_handler(commands=['start'])
def start(message):
    bot.reply_to(message, 'Bem-vindo!')
```

### Texto e regex

```python
@bot.message_handler(regexp=r'^/echo\s+(.+)')
def echo(message):
    bot.reply_to(message, message.text)
```

### Filtros de conteúdo

```python
@bot.message_handler(content_types=['photo'])
def handle_photo(message):
    bot.reply_to(message, 'Recebi sua foto!')

@bot.message_handler(content_types=['document'])
def handle_document(message):
    bot.reply_to(message, 'Recebi o documento!')
```

### Mensagem editada

```python
@bot.edited_message_handler(func=lambda m: True)
def edited(message):
    bot.send_message(message.chat.id, 'Mensagem editada recebida.')
```

### Custom filters

```python
def is_admin(message):
    return message.from_user and message.from_user.id == ADMIN_ID

@bot.message_handler(func=is_admin)
def admin_handler(message):
    bot.reply_to(message, 'Olá admin!')
```

### Callback queries

```python
@bot.callback_query_handler(func=lambda call: True)
def callback_handler(call):
    bot.answer_callback_query(call.id, 'Processado!')
    bot.send_message(call.message.chat.id, f'Você escolheu {call.data}')
```

---

## 📌 Webhooks e Polling

### Polling

```python
bot.infinity_polling(timeout=60, long_polling_timeout=90, non_stop=True)
```

### Webhooks com Flask

```python
import telebot
from flask import Flask, request

app = Flask(__name__)
bot = telebot.TeleBot(TOKEN)

@app.route('/webhook', methods=['POST'])
def webhook():
    json_str = request.stream.read().decode('utf-8')
    update = telebot.types.Update.de_json(json_str)
    bot.process_new_updates([update])
    return '', 200

if __name__ == '__main__':
    bot.remove_webhook()
    bot.set_webhook(url='https://seudominio.com/webhook')
    app.run(host='0.0.0.0', port=5000)
```

### Webhooks com FastAPI

```python
from fastapi import FastAPI, Request
import telebot

app = FastAPI()
bot = telebot.TeleBot(TOKEN)

@app.post('/webhook')
async def webhook(request: Request):
    json_str = await request.body()
    update = telebot.types.Update.de_json(json_str.decode('utf-8'))
    bot.process_new_updates([update])
    return {'ok': True}
```

### Webhooks e certificado
- `bot.set_webhook(url, certificate=open('cert.pem', 'rb'))`
- Certifique-se de usar HTTPS válido.

---

## 🧠 Estado, Conversas e Fluxo

### Próximo handler

```python
@bot.message_handler(commands=['start'])
def start(message):
    msg = bot.reply_to(message, 'Qual seu nome?')
    bot.register_next_step_handler(msg, process_name)


def process_name(message):
    bot.reply_to(message, f'Prazer em conhecer você, {message.text}!')
```

### Conversas simples

```python
states = {}

@bot.message_handler(func=lambda message: True)
def all_messages(message):
    state = states.get(message.chat.id)
    if state == 'ask_email':
        bot.send_message(message.chat.id, f'Email registrado: {message.text}')
        states.pop(message.chat.id, None)
    else:
        bot.send_message(message.chat.id, 'Qual seu email?')
        states[message.chat.id] = 'ask_email'
```

---

## 🛠️ Mensagens Avançadas

### Fotos, áudio e documentos

```python
bot.send_photo(chat_id, open('imagem.jpg', 'rb'))
bot.send_audio(chat_id, open('audio.mp3', 'rb'))
bot.send_document(chat_id, open('arquivo.pdf', 'rb'))
```

### Material de mídia em grupo

```python
media = [
    telebot.types.InputMediaPhoto(open('img1.jpg', 'rb')),
    telebot.types.InputMediaPhoto(open('img2.jpg', 'rb')),
]
bot.send_media_group(chat_id, media)
```

### Enviar localização

```python
bot.send_location(chat_id, latitude=..., longitude=...)
```

### Editar mensagens

```python
bot.edit_message_text('Texto atualizado', chat_id=chat_id, message_id=message_id)
```

### Deletar mensagem

```python
bot.delete_message(chat_id, message_id)
```

---

## 🧾 Erros, Logs e Retry

### Tratamento de exceções

```python
try:
    bot.send_message(chat_id, 'Olá')
except telebot.apihelper.ApiException as e:
    print('Erro API Telegram:', e)
```

### Logging
- Use `logging` ou `loguru` para capturar detalhes e rastrear falhas.
- Registre `bot.polling()` e falhas em webhook.

### Retry automático
- Recrie o bot em caso de erro de conexão.
- Use `bot.infinity_polling()` com `non_stop=True`.

---

## ⚙️ Boas Práticas de Produção

1. Guarde `TOKEN` em variáveis de ambiente.
2. Use HTTPS para webhooks.
3. Prefira `webhook` em produção em vez de `polling`.
4. Mantenha os handlers rápidos e use async / background para I/O.
5. Use armazenamento persistente para estado de conversa.
6. Teste com logs e monitoramento.
7. Evite expor dados sensíveis em mensagens ou logs.
8. Use `bot.remove_webhook()` ao alternar entre webhook e polling.

---

## 📚 Extras Úteis

### Proxy

```python
telebot.apihelper.proxy = {'https': 'socks5://user:pass@host:port'}
```

### Informações do chat

```python
chat = bot.get_chat(chat_id)
print(chat.title, chat.type)
```

### Enviar botão de login

```python
login_button = types.InlineKeyboardButton('Login', login_url='https://example.com')
```

---

## 💡 Conclusão

Este guia agora aborda de forma ampla o `pyTelegramBotAPI`, incluindo instalação, bot síncrono e assíncrono, teclados, handlers, webhooks, fluxo de conversa, envio de mídia, tratamento de erros e melhores práticas.

> Com estes tópicos, a documentação aproxima-se de uma referência abrangente para bots Telegram em Python.
