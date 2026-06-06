# 🔐 Ultimate Guide de JWT (JSON Web Token) em Python

Este guia cobre JWT em profundidade: estrutura, criação, verificação, segurança, fluxo de refresh, armazenamento e armadilhas comuns.

---

## 📑 Sumário
1. [🛠️ Instalação](#️-instalação)
2. [🧩 O que é JWT?](#-o-que-é-jwt)
3. [📦 Estrutura do Token](#-estrutura-do-token)
4. [🔐 Algoritmos e Segurança](#-algoritmos-e-segurança)
5. [🏗️ Criação de Tokens](#-criação-de-tokens)
6. [🔎 Validação e Decodificação](#-validação-e-decodificação)
7. [🔁 Refresh Tokens e Rotação](#-refresh-tokens-e-rotação)
8. [🧾 Revogação e Blacklist](#-revogação-e-blacklist)
9. [🛡️ Armazenamento Seguro](#-armazenamento-seguro)
10. [🔐 JWK e Rotação de Chaves](#-jwk-e-rotação-de-chaves)
11. [🧠 JWT, OAuth2 e OIDC](#-jwt-oauth2-e-oidc)
12. [⚠️ Armadilhas e Ataques Comuns](#-armadilhas-e-ataques-comuns)
13. [📌 Quando Usar JWT (e Quando Evitar)](#-quando-usar-jwt-e-quando-evitar)
14. [📘 Exemplos com Flask/FastAPI](#-exemplos-com-flaskfastapi)
15. [💡 Boas Práticas](#-boas-práticas)
16. [✅ Checklist de Produção](#-checklist-de-produção)

---

## 🛠️ Instalação
```bash
pip install PyJWT
```

> Em ambientes mais avançados, considere bibliotecas como `python-jose` ou `authlib` se precisar de suporte a mais algoritmos ou JWKs.

---

## 🧩 O que é JWT?
JWT é um padrão RFC 7519 para transmitir informações de forma compacta e segura entre duas partes. Ele é composto por três partes:
- Header: algoritmo e tipo
- Payload: claims com informações do token
- Signature: assinatura para garantir integridade

JWTs são frequentemente usados para autenticação stateless e autorização em APIs.

---

## 📦 Estrutura do Token
Um JWT tem o formato:
```
header.payload.signature
```
Exemplo de header:
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
Claims comuns no payload:
- `iss`: emissor
- `sub`: assunto (usuário, cliente)
- `aud`: audiência
- `exp`: expiração
- `nbf`: não antes
- `iat`: emitido em
- `jti`: identificador único do token
- `type`: tipo customizado (`access`, `refresh`)

---

## 🔐 Algoritmos e Segurança
### HS256 (simétrico)
- Bom para APIs simples e serviços que compartilham segredo.
- Requer manter o segredo seguro em ambos os lados.

### RS256 / ES256 (assimétrico)
- Recomendado para produção quando você quer separar assinatura e verificação.
- A chave privada assina; a chave pública valida.
- Útil para microserviços e autenticação federada.

### Atenção ao `alg`
Nunca confie apenas no header do token para escolher o algoritmo. Sempre defina explicitamente os algoritmos aceitos.

---

## 🏗️ Criação de Tokens
### Access Token com claims padrão
```python
import jwt
import datetime
from typing import Optional

SECRET_KEY = "sua_chave_secreta_super_segura"
ALGORITHM = "HS256"

def create_access_token(data: dict, expires_delta: Optional[datetime.timedelta] = None):
    to_encode = data.copy()
    now = datetime.datetime.now(datetime.timezone.utc)
    expire = now + (expires_delta or datetime.timedelta(minutes=15))
    to_encode.update({
        "iss": "seu_servico",
        "sub": str(data.get("user_id")),
        "aud": "seu_cliente",
        "iat": now,
        "nbf": now,
        "exp": expire,
        "jti": "token-único-123",
        "type": "access",
    })
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
```

### Refresh Token
```python
REFRESH_SECRET_KEY = "outra_chave_ainda_mais_segura"

def create_refresh_token(data: dict):
    now = datetime.datetime.now(datetime.timezone.utc)
    expire = now + datetime.timedelta(days=7)
    to_encode = data.copy()
    to_encode.update({
        "iss": "seu_servico",
        "sub": str(data.get("user_id")),
        "aud": "seu_cliente",
        "iat": now,
        "nbf": now,
        "exp": expire,
        "jti": "refresh-único-456",
        "type": "refresh",
    })
    return jwt.encode(to_encode, REFRESH_SECRET_KEY, algorithm=ALGORITHM)
```

### RS256 com chaves PEM
```python
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa

private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
public_key = private_key.public_key()

pem_priv = private_key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.PKCS8,
    encryption_algorithm=serialization.NoEncryption(),
)

pem_pub = public_key.public_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PublicFormat.SubjectPublicKeyInfo,
)

encoded = jwt.encode(payload, pem_priv, algorithm="RS256")
```

---

## 🔎 Validação e Decodificação
### Decodificando com validação forte
```python
def decode_token(token: str, audience: str, is_refresh: bool = False):
    key = REFRESH_SECRET_KEY if is_refresh else SECRET_KEY
    try:
        payload = jwt.decode(
            token,
            key,
            algorithms=[ALGORITHM],
            audience=audience,
            issuer="seu_servico",
            options={"require": ["exp", "iat", "nbf", "iss", "aud", "jti"]},
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise ValueError("Token expirado")
    except jwt.InvalidAudienceError:
        raise ValueError("Audience inválida")
    except jwt.InvalidIssuerError:
        raise ValueError("Issuer inválido")
    except jwt.InvalidIssuedAtError:
        raise ValueError("Token emitido em horário inválido")
    except jwt.InvalidTokenError:
        raise ValueError("Token inválido")
```

### Verifique o tipo do token
Sempre confirme se o token é do tipo correto.
```python
if payload.get("type") != "access":
    raise ValueError("Token de tipo incorreto")
```

### Usar opções de verificação adicionais
- `verify_signature`: assine sempre os tokens.
- `verify_exp`: não aceite tokens expirados.
- `verify_aud`: valide a audiência.
- `verify_iss`: valide o emissor.

---

## 🔁 Refresh Tokens e Rotação
### Fluxo típico
1. Cliente usa Access Token em chamadas de API.
2. Quando o Access Token expira, o cliente envia o Refresh Token.
3. O servidor valida o Refresh Token.
4. Se válido, emite novo Access Token e, opcionalmente, novo Refresh Token.

### Rotação de refresh token
Sempre que um refresh token for usado, emita um novo refresh token e invalide o antigo para reduzir risco de roubo.

```python
def rotate_refresh_token(old_refresh_token: str):
    payload = decode_token(old_refresh_token, audience="seu_cliente", is_refresh=True)
    # marcar old_refresh_token como revogado
    return create_refresh_token({"user_id": payload["sub"]})
```

### Rejeitar refresh tokens usados
Armazene o `jti` do refresh token usado e bloqueie reuso. Isso é crítico para evitar replay attacks.

---

## 🧾 Revogação e Blacklist
### Logout e revogação imediata
Para invalidar tokens imediatamente, mantenha uma blacklist de `jti` em um cache rápido como Redis.

- Access Token: guarda lista de `jti` revogados enquanto o token ainda não expira.
- Refresh Token: se o refresh token for revogado, impeça a emissão de novos tokens.

### Alternativa: allowlist com data de expiração
Em vez de blacklist, mantenha uma allowlist de tokens válidos e remova tokens antigos automaticamente.

---

## 🛡️ Armazenamento Seguro
### Access Tokens
- **Cookies `HttpOnly` e `SameSite=Lax/Strict`** são recomendados para reduzir XSS.
- `localStorage` é vulnerável a XSS e deve ser evitado se possível.

### Refresh Tokens
- Devem ser armazenados em cookies `HttpOnly`, preferencialmente com `Secure` e `SameSite`.
- Podem ser mantidos em um armazenamento separado do access token.

### CSRF versus XSS
- Se usar cookies, proteja contra CSRF com tokens CSRF ou SameSite forte.
- Se usar cabeçalhos personalizados (`Authorization: Bearer`), proteja contra XSS.

### Não armazene segredos no payload
O JWT não deve conter dados sensíveis como senha, token de API ou PII que não devem ser expostos.

---

## 🔐 JWK e Rotação de Chaves
### O que são JWK e JWKS
- **JWK** (JSON Web Key) é um formato JSON para representar chaves criptográficas.
- **JWKS** (JSON Web Key Set) é um conjunto de JWKs, geralmente exposto por um provedor de identidade.

### Uso em verificação assimétrica
Em vez de codificar a chave pública no seu código, consuma um JWKS endpoint e selecione a chave correta pelo `kid`.

```python
import requests
from jwt import PyJWKClient

jwks_url = "https://example.com/.well-known/jwks.json"
jwk_client = PyJWKClient(jwks_url)
signing_key = jwk_client.get_signing_key_from_jwt(token)
payload = jwt.decode(token, signing_key.key, algorithms=["RS256"], audience="seu_cliente")
```

### Rotação de chaves
- pública e privada devem ser rotacionadas periodicamente.
- mantenha múltiplas chaves válidas durante a transição.
- use `kid` no header para indicar qual chave assinou o token.

### JWE (JSON Web Encryption)
JWTs normalmente são assinados, não criptografados. Se precisar de confidencialidade, use JWE em vez de JWT padrão.

---

## 🧠 JWT, OAuth2 e OIDC
### JWT no OAuth2
- OAuth2 usa tokens de acesso e refresh, que podem ser JWTs ou simples tokens opacos.
- JWTs permitem validação descentralizada sem consulta ao servidor de autenticação.

### OIDC (OpenID Connect)
- OIDC é uma camada de identidade sobre OAuth2.
- OIDCs usam JWTs chamados `ID Tokens` para transmitir informações de autenticação sobre o usuário.

### ID Token vs Access Token
- **Access Token**: usado para autorização de API.
- **ID Token**: usado para autenticação e contém informações sobre o usuário.

### Exemplo de fluxo simplificado
1. Cliente obtém código de autorização.
2. Troca código por `access_token`, `refresh_token` e `id_token`.
3. Valida `id_token` com `iss`, `aud`, `exp`, `nonce`.

---

## ⚠️ Armadilhas e Ataques Comuns
- `alg: none`: proteja-se definindo explicitamente `algorithms=[...]` no decode.
- Tokens em `localStorage` podem ser roubados via XSS.
- Payload não é criptografado: o conteúdo é legível sem chave.
- Não use JWT como sessão única se precisar de logout instantâneo sem blacklist.
- Não confie apenas em `exp` para logout; use revogação/blacklist quando necessário.
- Não implemente sua própria geração de `jti` insegura; use UUIDs ou identificadores únicos fortes.

---

## 📌 Quando Usar JWT (e Quando Evitar)
### Use JWT quando:
- você precisa de autenticação stateless.
- múltiplos serviços precisam validar um token sem acessar um banco de dados central.
- você precisa de tokens compactos e independentes.

### Não use JWT quando:
- você precisa de controle de sessão centralizado estrito.
- logout imediato e revogação são obrigatórios sem complexidade adicional.
- seu token carrega grande volume de dados ou informações muito sensíveis.

Alternativas:
- sessões server-side (cookies de sessão)
- OAuth 2.0 com tokens de acesso curtos e refresh tokens
- API keys para autorização simples

---

## 📘 Exemplos com Flask/FastAPI
### Flask
```python
from flask import Flask, request, jsonify
app = Flask(__name__)

@app.route('/secure')
def secure():
    auth = request.headers.get('Authorization', '')
    token = auth.replace('Bearer ', '')
    payload = decode_token(token, audience='seu_cliente')
    return jsonify(payload)
```

### FastAPI
```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import HTTPBearer

app = FastAPI()
security = HTTPBearer()

async def get_current_user(credentials=Depends(security)):
    token = credentials.credentials
    return decode_token(token, audience='seu_cliente')
```

---

## 💡 Boas Práticas
- Use `iat`, `nbf`, `exp`, `iss`, `aud`, `sub` e `jti` sempre que possível.
- Assine tokens com chaves fortes e rotacione-as periodicamente.
- Mantenha durações curtas para access tokens (minutos) e durações maiores para refresh tokens (dias/semana).
- Documente o fluxo de autenticação e como tokens são emitidos e revogados.
- Teste o comportamento de tokens expirados, inválidos e revogados.
- Monitore tentativas de uso de tokens inválidos e respostas 401/403.
- Evite payloads grandes ou frágeis; mantenha apenas o mínimo necessário.

---

## ✅ Checklist de Produção
- [ ] Usar `exp`, `iat`, `nbf`, `iss`, `aud` e `jti`.
- [ ] Verificar explicitamente o algoritmo permitido.
- [ ] Separar chaves de acesso e refresh.
- [ ] Armazenar refresh tokens em `HttpOnly` + `SameSite`.
- [ ] Implementar rotação de refresh token.
- [ ] Implementar revogação/blacklist ou allowlist.
- [ ] Proteger contra XSS e CSRF.
- [ ] Não incluir dados sensíveis no payload.
- [ ] Rotacionar chaves e usar `kid` para JWK.
- [ ] Usar TLS em todas as transmissões de token.

---

## 🧠 Observações Finais
Este guia agora cobre JWT desde a estrutura até a implementação segura em produção. A verdadeira produção JWT exige design cuidadoso, revisão de segurança e testes contínuos.
