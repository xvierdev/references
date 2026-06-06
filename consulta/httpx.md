# 🌐 Guia Completo: HTTPX (Próxima Geração de Clientes HTTP)

O `httpx` é um cliente HTTP moderno e completo para Python 3, compatível com a API da biblioteca `requests`, mas com suporte nativo a **Async**, **HTTP/2** e muito mais.

---

## 📑 Sumário
1. [🛠️ Instalação](#-instalação)
2. [🔄 Síncrono vs Assíncrono](#-síncrono-vs-assíncrono)
3. [📡 Requisições Básicas](#-requisições-básicas)
4. [🛠️ Parâmetros, Headers e Body](#️-parâmetros-headers-e-body)
5. [📦 O Objeto Response](#-o-objeto-response)
6. [🏢 Usando o Client (Melhor Prática)](#-usando-o-client-melhor-prática)
7. [⚡ Performance e Timeouts](#-performance-e-timeouts)
8. [🔐 Autenticação e Headers de Segurança](#-autenticação-e-headers-de-segurança)
9. [🧭 Redirecionamentos e Proxies](#-redirecionamentos-e-proxies)
10. [📡 Retry, Backoff e Resiliência](#-retry-backoff-e-resiliência)
11. [📦 Streaming e Upload/Download de Arquivos](#-streaming-e-uploaddownload-de-arquivos)
12. [🍪 Cookies e Sessões](#-cookies-e-sessões)
13. [🔒 SSL/TLS e Segurança de Conexão](#-ssltls-e-segurança-de-conexão)
14. [🐞 Debug, Logging e Diagnóstico](#-debug-logging-e-diagnóstico)
15. [✅ Checklist de HTTPX](#-checklist-de-httpx)

---

## 🛠️ Instalação
```bash
pip install httpx
# Para suporte a HTTP/2 (opcional)
pip install httpx[http2]
```

---

## 🔄 Síncrono vs Assíncrono

### **Síncrono (Estilo Requests)**
```python
import httpx

response = httpx.get("https://www.google.com")
print(response.status_code)
```

### **Assíncrono (Recomendado para FastAPI/Asyncio)**
```python
import httpx
import asyncio

async def main():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://www.google.com")
        print(response.status_code)

asyncio.run(main())
```

---

## 📡 Requisições Básicas
Suporta todos os verbos padrão:
```python
client.get(url)
client.post(url, data={"key": "value"})
client.put(url, json={"key": "value"})
client.patch(url, json={"key": "value"})
client.delete(url)
client.options(url)
client.head(url)
```

---

## 🛠️ Parâmetros, Headers e Body

### **Query Parameters**
```python
params = {"id": 123, "active": "true"}
response = httpx.get("https://api.exemplo.com/v1", params=params)
# URL final: https://api.exemplo.com/v1?id=123&active=true
```

### **Headers e Cookies**
```python
headers = {"X-Token": "meu-secret"}
cookies = {"session_id": "xyz"}
response = httpx.get(url, headers=headers, cookies=cookies)
```

### **Enviando Dados (Body)**
```python
# Formulário (application/x-www-form-urlencoded)
httpx.post(url, data={"nome": "João"})

# JSON (Automaticamente define Content-Type: application/json)
httpx.post(url, json={"id": 1, "status": "ok"})

# Arquivos (multipart/form-data)
files = {"upload-file": open("foto.jpg", "rb")}
httpx.post(url, files=files)
```

---

## 📦 O Objeto Response
```python
response = httpx.get(url)

print(response.status_code)    # 200
print(response.text)           # Conteúdo como string
print(response.content)        # Conteúdo como bytes (binário)
print(response.json())         # Decodifica JSON para dict/list
print(response.headers)        # Cabeçalhos de resposta
print(response.url)            # URL final (útil se houver redirect)
print(response.history)        # Lista de redirecionamentos (se houver)

# Lançar erro se o status for 4xx ou 5xx
response.raise_for_status()
```

---

## 🏢 Usando o Client (Melhor Prática)
O `Client` (ou `AsyncClient`) mantém conexões abertas (Connection Pooling), o que é **muito mais rápido** do que fazer chamadas avulsas.
Use um client único para a vida do aplicativo ou por sessão em vez de criar um novo por requisição.

```python
# Definindo configurações base no Client
with httpx.Client(base_url="https://api.site.com/v1", timeout=5.0) as client:
    r1 = client.get("/users")
    r2 = client.get("/posts")
```

```python
# AsyncClient em aplicações assíncronas
async with httpx.AsyncClient(base_url="https://api.site.com/v1", timeout=5.0) as client:
    r1 = await client.get("/users")
    r2 = await client.get("/posts")
```

---

## ⚡ Performance e Timeouts

### **Configurando Timeouts**
Diferente do `requests`, o `httpx` tem um timeout padrão de **5 segundos**.
```python
# Timeout total de 10 segundos
httpx.get(url, timeout=10.0)

# Sem timeout (não recomendado para produção)
httpx.get(url, timeout=None)

# Detalhado
timeout = httpx.Timeout(10.0, connect=2.0) # 10s total, mas 2s para conectar
httpx.get(url, timeout=timeout)
```

### **Limites de Conexão (Pool)**
```python
limits = httpx.Limits(max_keepalive_connections=5, max_connections=10)
client = httpx.Client(limits=limits)
```

---

## � Autenticação e Headers de Segurança
### Basic Auth
```python
response = httpx.get(url, auth=("user", "pass"))
```

### Bearer Token
```python
headers = {"Authorization": "Bearer SEU_TOKEN"}
response = httpx.get(url, headers=headers)
```

### Custom Auth
```python
class TokenAuth(httpx.Auth):
    def auth_flow(self, request):
        request.headers["Authorization"] = "Bearer SEU_TOKEN"
        yield request

with httpx.Client(auth=TokenAuth()) as client:
    client.get(url)
```

### Security Headers
- `Accept: application/json`
- `Content-Type: application/json`
- `User-Agent` customizado quando necessário

---

## 🧭 Redirecionamentos e Proxies
### Controlando redirecionamentos
```python
response = httpx.get(url, follow_redirects=True)
response = httpx.get(url, follow_redirects=False)
```

### Proxies
```python
proxies = {
    "http://": "http://localhost:8080",
    "https://": "http://localhost:8080",
}
response = httpx.get(url, proxies=proxies)
```

### Proxy com autenticação
```python
proxies = {
    "https://": "http://user:pass@proxy.local:3128",
}
```

---

## 📡 Retry, Backoff e Resiliência
O `httpx` não inclui retry automático no núcleo, mas você pode implementar facilmente lógica personalizada.

```python
import time
import httpx

for attempt in range(1, 4):
    try:
        response = httpx.get(url, timeout=5.0)
        response.raise_for_status()
        break
    except (httpx.RequestError, httpx.HTTPStatusError) as exc:
        if attempt == 3:
            raise
        time.sleep(2 ** attempt)
```

### Retry com AsyncClient
```python
import asyncio
import httpx

async def fetch(url):
    for attempt in range(1, 4):
        try:
            async with httpx.AsyncClient() as client:
                resp = await client.get(url, timeout=5.0)
                resp.raise_for_status()
                return resp
        except (httpx.RequestError, httpx.HTTPStatusError):
            await asyncio.sleep(2 ** attempt)
    raise RuntimeError("Falha após 3 tentativas")
```

---

## 📦 Streaming e Upload/Download de Arquivos
### Download em stream
```python
with httpx.stream("GET", url) as response:
    response.raise_for_status()
    with open("arquivo.bin", "wb") as f:
        for chunk in response.iter_bytes():
            f.write(chunk)
```

### Upload em streaming
```python
with open("grande.zip", "rb") as f:
    response = httpx.post(url, content=f)
```

### Async streaming
```python
async with httpx.AsyncClient() as client:
    async with client.stream("GET", url) as response:
        async for chunk in response.aiter_bytes():
            process(chunk)
```

---

## 🍪 Cookies e Sessões
### Cookies automáticos
```python
with httpx.Client() as client:
    client.get("https://example.com/login")
    client.get("https://example.com/profile")
```

### Cookie Jar
```python
client = httpx.Client(cookies={"session_id": "abc"})
```

### Limpando cookies
```python
client.cookies.clear()
```

---

## 🔒 SSL/TLS e Segurança de Conexão
### Verificação de certificados
```python
response = httpx.get(url, verify=True)
```

### Desabilitar verificação (não recomendado)
```python
response = httpx.get(url, verify=False)
```

### Usando CA customizada
```python
response = httpx.get(url, verify="/caminho/ca.pem")
```

### HTTP/2
```python
client = httpx.Client(http2=True)
```

### trust_env
Use `trust_env=True` para herdar proxies/variáveis do ambiente.
```python
client = httpx.Client(trust_env=True)
```

---

## 🐞 Debug, Logging e Diagnóstico
### Ativar logging do HTTPX
```python
import logging
logging.basicConfig(level=logging.DEBUG)
logging.getLogger("httpx").setLevel(logging.DEBUG)
```

### Exibir request/response
```python
print(response.request.headers)
print(response.text)
```

### Conexões e DNS
- Use `httpx.Client(http2=True, verify=True)` para debugar HTTP/2 e TLS.
- Use `trust_env=True` para ver se proxies de ambiente estão afetando a conexão.

---

## �💡 Dicas de Ouro
- Use **`response.raise_for_status()`** sempre que não quiser tratar erros manualmente.
- Se estiver usando **FastAPI**, o `httpx` é a escolha padrão para fazer requisições a outros serviços dentro de rotas assíncronas.
- O `httpx` detecta automaticamente a codificação de texto da resposta.
## ✅ Checklist de HTTPX
- [ ] Use `Client`/`AsyncClient` reutilizável em vez de instanciar clientes por requisição.
- [ ] Defina timeouts claros para conectar, ler e escrever.
- [ ] Configure `limits` quando houver alto volume de conexões.
- [ ] Use `verify` com certificados válidos em produção.
- [ ] Habilite `http2=True` apenas quando o servidor e proxy suportarem.
- [ ] Controle redirecionamentos e proxies explicitamente.
- [ ] Faça retry/backoff para chamadas instáveis e serviços externos.
- [ ] Registre requests e responses de forma segura para análise de falhas.
