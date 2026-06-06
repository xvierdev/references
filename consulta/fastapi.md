# ⚡ Bíblia do FastAPI (Básico ao Profissional)

O FastAPI é um framework moderno, de alta performance, para construir APIs com Python 3.8+ baseado em Type Hints e Pydantic. Esta 'bíblia' cobre desde a criação básica até padrões de produção, segurança, testes e deploy.

---

## 📑 Sumário
1. [🚀 Iniciando](#-iniciando)
2. [🏗️ Estrutura de Projeto](#-estrutura-de-projeto)
3. [📦 FastAPI e Pydantic](#-fastapi-e-pydantic)
4. [📍 Parâmetros: Path, Query, Body](#-parâmetros-path-query-body)
5. [📦 Resposta e Models](#-resposta-e-models)
6. [🔁 Routers e Modularização](#-routers-e-modularização)
7. [💉 Dependências com `Depends`](#-dependências-com-depends)
8. [🔒 Segurança e Autenticação](#-segurança-e-autenticação)
9. [📤 Upload, Download e Arquivos](#-upload-download-e-arquivos)
10. [🧱 Middleware, CORS e Eventos](#-middleware-cors-e-eventos)
11. [🧠 Erros, Validação e Exceções](#-erros-validação-e-exceções)
12. [🧪 Testes](#-testes)
13. [🚀 Deploy e Produção](#-deploy-e-produção)
14. [💡 Boas Práticas](#-boas-práticas)
15. [✅ Checklist de Produção](#-checklist-de-produção)

---

## 🚀 Iniciando

### Instalar
```bash
pip install fastapi uvicorn[standard]
```

### app.py mínimo
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello World"}
```

### Executar
```bash
uvicorn app:app --reload
```

---

## 🏗️ Estrutura de Projeto

```
app.py
routes/
  users.py
  items.py
models/
  schemas.py
  database.py
services/
  auth.py
  users.py
tests/
  test_users.py
```

- `app.py`: entrada principal.
- `routes/`: APIRouters por domínio.
- `models/`: schemas e conexões.
- `services/`: lógica de negócio.
- `tests/`: casos de teste.

---

## 📦 FastAPI e Pydantic

### Schema básico
```python
from pydantic import BaseModel
from typing import Optional

class UserSchema(BaseModel):
    username: str
    email: str
    full_name: Optional[str] = None
    is_active: bool = True
```

### Configurações avançadas
```python
from pydantic import BaseModel, EmailStr, Field
from datetime import datetime

class UserSchema(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    created_at: datetime = Field(default_factory=datetime.utcnow)
```

---

## 📍 Parâmetros: Path, Query, Body

### Path Parameters
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

### Query Parameters
```python
@app.get("/search")
def search(q: str, limit: int = 10):
    return {"q": q, "limit": limit}
```

### Body Parameters
```python
@app.post("/users")
def create_user(user: UserSchema):
    return user
```

### Parâmetros mistos
```python
@app.post("/users/{user_id}")
def update_user(user_id: int, user: UserSchema, q: Optional[str] = None):
    return {"user_id": user_id, "user": user, "q": q}
```

### Request body direto
```python
from typing import Dict

@app.post("/data")
def receive_data(payload: Dict[str, int]):
    return payload
```

---

## 📦 Resposta e Models

### `response_model`
```python
@app.get("/me", response_model=UserSchema)
def read_user():
    return UserSchema(username="xvier", email="x@ex.com")
```

### `response_model_exclude_none`
```python
@app.get("/me", response_model=UserSchema, response_model_exclude_none=True)
def read_user():
    return {"username": "xvier", "email": "x@ex.com", "full_name": None}
```

### Status codes
```python
from fastapi import status

@app.post("/users", status_code=status.HTTP_201_CREATED)
def create_user(user: UserSchema):
    return user
```

### Headers na resposta
```python
from fastapi import Response

@app.get("/headers")
def headers(response: Response):
    response.headers["X-Frame-Options"] = "DENY"
    return {"message": "ok"}
```

---

## 🔁 Routers e Modularização

### `routes/users.py`
```python
from fastapi import APIRouter, HTTPException, status
from typing import Optional
from models.schemas import UserSchema

router = APIRouter(prefix="/users", tags=["users"])

@router.get("/{user_id}", response_model=UserSchema)
def get_user(user_id: int, q: Optional[str] = None):
    if user_id != 1:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Usuário não encontrado")
    return UserSchema(username="xvier", email="x@ex.com", full_name=q)
```

### `app.py`
```python
from fastapi import FastAPI
from routes.users import router as users_router

app = FastAPI()
app.include_router(users_router)
```

### Subrotas e tags
- `prefix` no router
- `tags` para agrupar documentação
- `dependencies` no router para dependências comuns

---

## 💉 Dependências com `Depends`

### Dependência simples
```python
from fastapi import Depends

def get_db():
    db = "db_connection"
    try:
        yield db
    finally:
        ...

@app.get("/items")
def read_items(db = Depends(get_db)):
    return {"db": db}
```

### Dependência com classes
```python
class CommonQueryParams:
    def __init__(self, q: Optional[str] = None, limit: int = 10):
        self.q = q
        self.limit = limit

@app.get("/items")
def read_items(commons: CommonQueryParams = Depends()):
    return {"q": commons.q, "limit": commons.limit}
```

### Dependências em routers
```python
router = APIRouter(prefix="/admin", dependencies=[Depends(get_current_user)])
```

### Reutilizar dependências
- autenticação
- conexão de banco de dados
- cache

---

## 🔒 Segurança e Autenticação

### HTTP Basic
```python
from fastapi.security import HTTPBasic, HTTPBasicCredentials
security = HTTPBasic()

@app.get("/basic")
def read_basic(credentials: HTTPBasicCredentials = Depends(security)):
    return {"username": credentials.username}
```

### Bearer Token
```python
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
bearer_scheme = HTTPBearer()

@app.get("/protected")
def read_protected(credentials: HTTPAuthorizationCredentials = Depends(bearer_scheme)):
    return {"token": credentials.credentials}
```

### OAuth2 Password Flow
```python
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.post("/token")
def login(form_data: OAuth2PasswordRequestForm = Depends()):
    return {"access_token": "token", "token_type": "bearer"}

@app.get("/users/me")
def read_current_user(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

### CORS
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Security utilities
- `HTTPException`
- `status` codes
- `Security` para múltiplas dependências

---

## 📤 Upload, Download e Arquivos

### Upload de arquivo
```python
from fastapi import File, UploadFile

@app.post("/upload")
async def upload_file(file: UploadFile = File(...)):
    content = await file.read()
    return {"filename": file.filename, "size": len(content)}
```

### Vários arquivos
```python
@app.post("/uploads")
async def upload_files(files: list[UploadFile] = File(...)):
    return {"filenames": [file.filename for file in files]}
```

### Download de arquivo
```python
from fastapi.responses import FileResponse

@app.get("/download")
def download_file():
    return FileResponse("/path/to/file.zip", filename="arquivo.zip")
```

### Static files
```python
from fastapi.staticfiles import StaticFiles
app.mount("/static", StaticFiles(directory="static"), name="static")
```

---

## 🧱 Middleware, CORS e Eventos

### Middleware
```python
from starlette.middleware.base import BaseHTTPMiddleware

@app.middleware("http")
async def log_requests(request, call_next):
    response = await call_next(request)
    return response
```

### Eventos de inicialização e finalização
```python
@app.on_event("startup")
async def startup_event():
    print("App iniciado")

@app.on_event("shutdown")
async def shutdown_event():
    print("App finalizado")
```

### Lifespan
```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    print("startup")
    yield
    print("shutdown")

app = FastAPI(lifespan=lifespan)
```

---

## 🧠 Erros, Validação e Exceções

### HTTPException
```python
from fastapi import HTTPException, status

@app.get("/error")
def error():
    raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail="Bad request")
```

### Exceptions handlers
```python
from fastapi.responses import JSONResponse

@app.exception_handler(ValueError)
async def value_error_handler(request, exc):
    return JSONResponse(status_code=400, content={"detail": str(exc)})
```

### Validação Pydantic
- campos obrigatórios
- `Field(..., min_length=3)`
- `EmailStr`, `conint`, `constr`

### Valores padrão e alias
```python
from pydantic import Field

class Item(BaseModel):
    item_id: int = Field(..., alias="itemId")
```

### Modelos aninhados
```python
class Address(BaseModel):
    city: str

class User(BaseModel):
    name: str
    address: Address
```

---

## 🧪 Testes

### Testar com TestClient
```python
from fastapi.testclient import TestClient
from app import app

client = TestClient(app)

def test_read_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello World"}
```

### Testar dependências
```python
from unittest.mock import MagicMock

app.dependency_overrides[get_db] = lambda: "fake_db"
```

---

## 🚀 Deploy e Produção

### Uvicorn + Gunicorn
```bash
gunicorn -k uvicorn.workers.UvicornWorker app:app
```

### Configurações recomendadas
- `workers`: 2x CPUs + 1
- `timeout`: 30
- `keepalive`: 5

### Segurança
- habilite `https`
- proteja cabeçalhos com `SecurityMiddleware`
- use `CORS` apenas para origens confiáveis

---

## 💡 Boas Práticas

- Use `async def` apenas em operações de I/O e mantenha rotas leves.
- Separe rotas em `APIRouter` organizados por domínio e contexto de negócio.
- Use `response_model` para documentação e validação de saída.
- Evite lógica de negócio nas rotas; mova para serviços e camadas de domínio.
- Valide dados com Pydantic e crie schemas separados para requests e responses.
- Documente endpoints com `summary`, `description` e `tags` para a OpenAPI.
- Use `dependency_overrides` em testes para mockar conexões de banco e serviços externos.
- Evite variáveis globais mutáveis no processo do servidor.
- Controle limites de request body e tempo de execução em produção.

## 🧪 Testes, Observabilidade e Qualidade

- Use `TestClient` para rotas síncronas e `AsyncClient` para testes assíncronos.
- Substitua dependências com `app.dependency_overrides` em testes de integração.
- Garanta cobertura de casos de erro 4xx e 5xx.
- Valide o schema OpenAPI gerado antes do deploy.
- Registre logs estruturados e use request IDs para correlação de chamadas.
- Monitore latência, taxa de erros e uso de recursos com APM ou métricas customizadas.

## ✅ Checklist de Produção

- [ ] Rode o app em produção com `gunicorn -k uvicorn.workers.UvicornWorker`.
- [ ] Defina workers compatíveis com CPU e memória (`2 * CPUs + 1`).
- [ ] Use reverse proxy para HTTPS, caching e healthchecks.
- [ ] Configure CORS apenas para origens confiáveis.
- [ ] Proteja cabeçalhos HTTP sensíveis e limite métodos quando possível.
- [ ] Implemente startup/shutdown para conexões de DB, cache e recursos externos.
- [ ] Teste rotas, validação de payload e tratamento de erros.
- [ ] Separe ambientes via variáveis de ambiente e não versionar segredos.
- [ ] Use documentação OpenAPI e valide as especificações antes do deploy.
- [ ] Monitore erros, latência e tráfego em produção.

---

## 📚 Referências Rápidas
- `FastAPI()`, `APIRouter`, `Depends`
- `BaseModel`, `Field`, `EmailStr`
- `HTTPException`, `status`
- `UploadFile`, `File`, `Form`
- `TestClient`
- `CORSMiddleware`, `StaticFiles`, `middleware`
