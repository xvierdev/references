# 🔌 Ultimate Guide de WebSockets em Python

WebSockets permitem comunicação bidirecional e em tempo real entre cliente e servidor. Diferente do HTTP, a conexão permanece aberta e troca mensagens em tempo real.

---

## 📑 Sumário
1. [🛠️ Instalação](#-instalação)
2. [🌐 O que são WebSockets?](#-o-que-são-websockets)
3. [🌐 Usando a Biblioteca `websockets` (Baixo Nível)](#-usando-a-biblioteca-websockets-baixo-nível)
4. [⚡ Integração com FastAPI (Profissional)](#-integração-com-fastapi-profissional)
5. [👥 Gerenciamento de Múltiplas Conexões (Broadcasting)](#-gerenciamento-de-múltiplas-conexões-broadcasting)
6. [📡 Handshake, Subprotocols e Headers](#-handshake-subprotocols-e-headers)
7. [💬 Text vs Binary](#-text-vs-binary)
8. [🫀 Heartbeats, Ping/Pong e Timeouts](#-heartbeats-pingpong-e-timeouts)
9. [🔐 Segurança e Autenticação](#-segurança-e-autenticação)
10. [🚀 Escalabilidade e Redis Pub/Sub](#-escalabilidade-e-redis-pubsub)
11. [🌐 WSS, Proxies e Reverse Proxy](#-wss-proxies-e-reverse-proxy)
12. [🔧 Debug e Testes](#-debug-e-testes)
13. [📌 Quando usar WebSockets](#-quando-usar-websockets)
14. [💡 Dicas de Ouro](#-dicas-de-ouro)

---

## 🛠️ Instalação
```bash
pip install websockets
pip install fastapi uvicorn[standard]
```

> Use `uvicorn[standard]` para suportar HTTP/2, melhor desempenho e recursos de produção.

---

## 🌐 O que são WebSockets?
WebSockets são um protocolo que permite comunicação bidirecional persistente entre cliente e servidor. Após o handshake inicial via HTTP, o canal permanece aberto, permitindo envio de mensagens em tempo real.

- `ws://` é o protocolo não criptografado.
- `wss://` é o protocolo WebSocket seguro sobre TLS.
- WebSockets são ideais para chats, notificações, jogos, dashboards e streaming de dados.

---

## 🌐 Usando a Biblioteca `websockets` (Baixo Nível)
### Servidor Simples
```python
import asyncio
import websockets

async def echo(websocket):
    try:
        async for message in websocket:
            print(f"Recebido: {message}")
            await websocket.send(f"Eco: {message}")
    except websockets.ConnectionClosedOK:
        print("Conexão fechada normalmente")
    except websockets.ConnectionClosedError as e:
        print(f"Conexão fechada com erro: {e}")

async def main():
    async with websockets.serve(echo, "localhost", 8765):
        await asyncio.Future()  # mantém o servidor rodando

asyncio.run(main())
```

### Cliente Simples
```python
import asyncio
import websockets

async def hello():
    uri = "ws://localhost:8765"
    async with websockets.connect(uri) as websocket:
        await websocket.send("Olá Mundo!")
        resposta = await websocket.recv()
        print(f"Resposta do servidor: {resposta}")

asyncio.run(hello())
```

---

## ⚡ Integração com FastAPI (Profissional)
O FastAPI facilita o uso de WebSockets junto às rotas e ao ciclo de vida da aplicação.

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"Mensagem recebida: {data}")
    except WebSocketDisconnect:
        print("Cliente desconectado")
```

---

## 👥 Gerenciamento de Múltiplas Conexões (Broadcasting)
Para criar chat, notificações ou feeds, gerencie um conjunto de conexões ativas.

```python
from typing import List
from fastapi import FastAPI, WebSocket, WebSocketDisconnect

class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            await connection.send_text(message)

manager = ConnectionManager()
app = FastAPI()

@app.websocket("/chat")
async def chat_endpoint(websocket: WebSocket):
    await manager.connect(websocket)
    try:
        while True:
            data = await websocket.receive_text()
            await manager.broadcast(f"Alguém disse: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
```

---

## 📡 Handshake, Subprotocols e Headers
### Subprotocols
Subprotocols permitem negociar um formato de mensagem com o cliente.

Servidor:
```python
async with websockets.serve(echo, "localhost", 8765, subprotocols=["chat", "superchat"]):
    ...
```

Cliente:
```python
async with websockets.connect("ws://localhost:8765", subprotocols=["chat"]) as websocket:
    print(websocket.subprotocol)
```

### Headers customizados
```python
async with websockets.connect("ws://localhost:8765", extra_headers={"X-Token": "abc"}) as websocket:
    ...
```

### Close Codes
Use códigos de fechamento para comunicar o motivo:
- `1000`: normal
- `1001`: going away
- `1008`: policy violation
- `1011`: internal error

```python
await websocket.close(code=1000, reason="Usuário desconectou")
```

---

## 💬 Text vs Binary
WebSockets suportam mensagens de texto e binárias.

### Texto
```python
await websocket.send("mensagem")
message = await websocket.recv()
```

### Binário
```python
await websocket.send(b"bytes aqui")
message = await websocket.recv()
```

Use binário para imagens, áudio, vídeos ou protocolos customizados.

---

## 🫀 Heartbeats, Ping/Pong e Timeouts
### Ping/Pong
WebSockets suportam ping/pong nativo para manter a conexão viva.

Servidor:
```python
await websocket.ping()
```

Cliente:
```python
await websocket.pong()
```

### Timeouts e keepalive
Configure timeouts para detectar conexões mortas rapidamente.

```python
async with websockets.connect(uri, ping_interval=20, ping_timeout=20) as websocket:
    ...
```

---

## 🔐 Segurança e Autenticação
### WSS obrigatório
Sempre use `wss://` em produção para proteger o tráfego.

### Autenticação no handshake
- Query string: `wss://site.com/ws?token=XYZ`
- Headers customizados: `Authorization: Bearer XYZ`

### Validação de origem
No servidor, verifique o `Origin` para evitar conexões de sites não autorizados.

### Proteja contra mensagem malformada
- valide JSON antes de processar
- limite o tamanho das mensagens
- defina um rate limit por conexão

---

## 🚀 Escalabilidade e Redis Pub/Sub
### Por que precisamos de Redis
Em clusters, cada conexão WebSocket fica “presa” no mesmo servidor. Para broadcast entre instâncias, use Redis Pub/Sub.

### Exemplo simplificado
```python
import aioredis

redis = await aioredis.create_redis_pool("redis://localhost")
await redis.publish_json("chat", {"msg": "bom dia"})
```

No servidor, toda instância assina o canal e distribui mensagens para sua lista de WebSockets ativas.

---

## 🌐 WSS, Proxies e Reverse Proxy
### nginx como reverse proxy
Use `proxy_pass` e cabeçalhos WebSocket:
```nginx
location /ws/ {
    proxy_pass http://127.0.0.1:8000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header Host $host;
}
```

### Headers importantes
- `Upgrade: websocket`
- `Connection: Upgrade`
- `Sec-WebSocket-Key`
- `Sec-WebSocket-Accept`

---

## 🔧 Debug e Testes
### Teste com o navegador
No console do browser:
```js
let ws = new WebSocket("ws://localhost:8000/ws");
ws.onmessage = (e) => console.log(e.data);
ws.onopen = () => ws.send("oi");
```

### Logs no servidor
Aumente logs para capturar handshake, ping/pong e erros de conexão.

### Testes automatizados
Use `pytest` com clientes WebSocket asíncronos para simular conexões e falhas.

---

## 📌 Quando usar WebSockets
Use WebSockets quando:
- precisar de comunicação bidirecional em tempo real
- for necessário enviar eventos do servidor sem polling
- quiser baixa latência e canal persistente

Evite quando:
- atualizações são raras ou não em tempo real
- a implementação pode ser mais simples com SSE ou long polling
- você precisa de suporte amplo a firewalls restritivos

---

## 💡 Dicas de Ouro
- Use `wss://` em produção.
- Mantenha o payload pequeno e simples.
- Faça validações de entrada no servidor.
- Em clusters, use Redis ou outro broker para sincronizar mensagens.
- Sempre lide com desconexões e reconexões do cliente.
- Evite enviar dados sensíveis sem criptografia adicional.
