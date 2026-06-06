# 🛡️ Bíblia do Pydantic v2

O Pydantic é a biblioteca de validação, parsing e modelagem de dados orientada a Type Hints para Python. Esta bíblia cobre Pydantic v2 com exemplos práticos, configurações avançadas e padrões de uso em produção.

---

## 📑 Sumário
1. [🛠️ Instalação](#️-instalação)
2. [🏗️ Primer Rápido: BaseModel](#️-primer-rápido-basemodel)
3. [🔧 Field e validações de campo](#-field-e-validações-de-campo)
4. [🧩 Tipos Pydantic e construtores](#-tipos-pydantic-e-construtores)
5. [🧠 Validadores e serializadores](#-validadores-e-serializadores)
6. [🏷️ Alias, campos opcionais e defaults](#-alias-campos-opcionais-e-defaults)
7. [🧬 Modelos aninhados, listas e genéricos](#-modelos-aninhados-listas-e-genéricos)
8. [🛠️ Configuração do modelo](#️-configuração-do-modelo)
9. [📦 Serialização e parsing](#️-serialização-e-parsing)
10. [🔐 ORM Mode e integração com SQLAlchemy](#-orm-mode-e-integração-com-sqlalchemy)
11. [⚙️ FastAPI e integração com SQLModel](#️-fastapi-e-integração-com-sqlmodel)
12. [⚙️ Settings e variáveis de ambiente](#️-settings-e-variáveis-de-ambiente)
13. [🧪 Erros e mensagens de validação](#️-erros-e-mensagens-de-validação)
14. [🧩 Custom Types e extensões](#-custom-types-e-extensões)
15. [📈 Performance e boas práticas](#️-performance-e-boas-práticas)
16. [✅ Checklist do Pydantic](#️-checklist-do-pydantic)

---

## 🛠️ Instalação
```bash
pip install pydantic
pip install pydantic-settings
pip install email-validator
```

> Em Pydantic v2, o pacote principal continua `pydantic`. Para leituras de configuração, use `pydantic-settings`.

---

## 🏗️ Primer Rápido: BaseModel
O `BaseModel` é a base de todos os modelos Pydantic.

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    email: str

user = User(id=1, name='Alice', email='alice@example.com')
print(user)
```

### Parsing automático
Pydantic converte tipos automaticamente quando possível.

```python
user = User(id='1', name=123, email='alice@example.com')
print(user.id, type(user.id))  # int
print(user.name, type(user.name))  # str
```

---

## 🔧 Field e validações de campo
Use `Field` para adicionar metadados e restrições a cada campo.

```python
from pydantic import BaseModel, Field, ValidationError
from typing import Optional

class UserSchema(BaseModel):
    username: str = Field(
        ...,
        min_length=3,
        max_length=20,
        pattern=r'^[a-zA-Z0-9]+$',
        description='Nome de usuário alfanumérico',
        example='jv123',
    )
    age: int = Field(gt=0, le=120, description='Idade em anos')
    email: str = Field(..., description='Email válido')
    bio: Optional[str] = Field(default=None, max_length=500)
    is_active: bool = True

try:
    UserSchema(username='jv', age=150, email='invalido')
except ValidationError as e:
    print(e)
```

### Parâmetros comuns do Field
- `default`, `default_factory`
- `alias`
- `title`, `description`, `example`
- `gt`, `ge`, `lt`, `le`
- `min_length`, `max_length`
- `pattern`
- `strict`
- `json_schema_extra`

---

## 🧩 Tipos Pydantic e construtores
### Tipos úteis
```python
from pydantic import BaseModel, EmailStr, HttpUrl, AnyUrl, SecretStr, SecretBytes
from pydantic import PositiveInt, NegativeInt, NonNegativeInt, ConstrainedStr

class Product(BaseModel):
    email: EmailStr
    website: HttpUrl
    age: PositiveInt
    secret: SecretStr
```

### Tipos JSON e Raw
```python
from pydantic import Json

class Item(BaseModel):
    metadata: Json[dict]
```

### Tipos de caminho e sistema de arquivos
```python
from pathlib import Path
from pydantic import FilePath, DirectoryPath

class PathModel(BaseModel):
    file_path: FilePath
    dir_path: DirectoryPath
```

### Tipos estritos
```python
from pydantic import StrictInt, StrictBool, StrictStr

class StrictModel(BaseModel):
    value: StrictInt
```

### Tipos constritos com `Annotated`
```python
from typing import Annotated
from pydantic import Field, constr, conint

Username = Annotated[str, Field(min_length=3, max_length=20)]
Age = Annotated[int, Field(gt=0, le=150)]
```

---

## 🧠 Validadores e serializadores
### Field validators
```python
from pydantic import field_validator

class User(BaseModel):
    name: str
    email: EmailStr

    @field_validator('name')
    def validate_name(cls, v):
        if ' ' in v:
            raise ValueError('Nome não pode conter espaços')
        return v.title()
```

### Model validators
```python
from pydantic import model_validator

class Coordinates(BaseModel):
    x: float
    y: float

    @model_validator(mode='before')
    def pre_validate(cls, values):
        if isinstance(values, str):
            return dict(zip(['x', 'y'], map(float, values.split(','))))
        return values

    @model_validator(mode='after')
    def post_validate(self):
        if self.x == self.y:
            raise ValueError('x e y não podem ser iguais')
        return self
```

### Root validators
Em v2, use `@model_validator(mode='before')` para validações de todo o modelo.

---

## 🏷️ Alias, campos opcionais e defaults
### Alias de campos
```python
class User(BaseModel):
    user_id: int = Field(..., alias='id')

user = User(id=123)
print(user.user_id)
```

### `populate_by_name`
Use quando quiser criar modelos com nome do Python e alias.

```python
class User(BaseModel):
    user_id: int = Field(..., alias='id')
    model_config = ConfigDict(populate_by_name=True)
```

### Campos opcionais e default
- `Optional[str] = None`
- `Field(default_factory=list)`
- `Field(default_factory=datetime.utcnow)`

### Negar valores vazios
```python
from pydantic import Field

class Payload(BaseModel):
    value: str = Field(..., min_length=1)
```

---

## 🧬 Modelos aninhados, listas e genéricos
### Modelos aninhados
```python
class Address(BaseModel):
    city: str
    zip_code: str

class User(BaseModel):
    name: str
    address: Address

user = User(name='Alice', address={'city': 'SP', 'zip_code': '12345'})
```

### Listas e dicionários
```python
class Group(BaseModel):
    members: list[User]
    scores: dict[str, int]
```

### Genéricos
```python
from typing import Generic, TypeVar
from pydantic import BaseModel

T = TypeVar('T')

class ResponseModel(BaseModel, Generic[T]):
    data: T
    status: str
```

---

## 🛠️ Configuração do modelo
### `ConfigDict`
```python
from pydantic import BaseModel, ConfigDict

class User(BaseModel):
    name: str
    model_config = ConfigDict(
        extra='forbid',
        validate_default=True,
        arbitrary_types_allowed=False,
        str_strip_whitespace=True,
        populate_by_name=True,
    )
```

### `extra`
- `ignore`: descarta campos extras
- `allow`: mantém campos extras
- `forbid`: dispara erro em campos extras

### `frozen`
```python
class ImmutableModel(BaseModel):
    name: str
    model_config = ConfigDict(frozen=True)
```

### `json_schema_extra`
```python
class Product(BaseModel):
    name: str
    price: float
    model_config = ConfigDict(
        json_schema_extra={
            'example': {'name': 'Caneca', 'price': 19.90}
        }
    )
```

---

## 📦 Serialização e parsing
### `model_dump()`
```python
user = User(id=1, name='Alice')
print(user.model_dump())
```

### `model_dump_json()`
```python
print(user.model_dump_json())
```

### `model_validate()`
```python
user = User.model_validate({'id': '1', 'name': 'Alice'})
```

### `construct()`
Cria modelos sem validação, útil para performance quando os dados já são confiáveis.
```python
user = User.model_construct(id=1, name='Alice')
```

---

## 🔐 ORM Mode e integração com SQLAlchemy
### `from_attributes`
```python
class User(BaseModel):
    id: int
    name: str
    model_config = ConfigDict(from_attributes=True)
```

### Exemplo com SQLAlchemy
```python
from sqlmodel import SQLModel, Field

class UserTable(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str

user_obj = UserTable(id=1, name='Alice')
user = User.model_validate(user_obj)
```

---

## ⚙️ FastAPI e integração com SQLModel
### Exemplo de endpoint com request/response model
```python
from fastapi import FastAPI, HTTPException
from sqlmodel import SQLModel, Field, Session, create_engine, select
from pydantic import BaseModel

class UserBase(BaseModel):
    name: str

class UserCreate(UserBase):
    email: str

class UserRead(UserBase):
    id: int
    email: str

class UserTable(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str
    email: str

engine = create_engine("sqlite:///database.db", echo=True)
SQLModel.metadata.create_all(engine)

app = FastAPI()

@app.post("/users", response_model=UserRead)
def create_user(user: UserCreate):
    with Session(engine) as session:
        db_user = UserTable.from_orm(user)
        session.add(db_user)
        session.commit()
        session.refresh(db_user)
        return db_user

@app.get("/users/{user_id}", response_model=UserRead)
def read_user(user_id: int):
    with Session(engine) as session:
        user = session.get(UserTable, user_id)
        if not user:
            raise HTTPException(status_code=404, detail="User not found")
        return user
```

### Integração com Pydantic e SQLModel
- `response_model` serializa modelos Pydantic automaticamente.
- `UserCreate` valida payloads de entrada.
- `UserRead` define a forma segura de resposta.
- `from_orm` permite criar instâncias Pydantic a partir de objetos SQLAlchemy/SQLModel.

### Atualização parcial
```python
from fastapi import Body

@app.patch("/users/{user_id}", response_model=UserRead)
def update_user(user_id: int, user_update: UserCreate = Body(...)):
    with Session(engine) as session:
        user = session.get(UserTable, user_id)
        if not user:
            raise HTTPException(status_code=404, detail="User not found")
        user_data = user_update.model_dump(exclude_unset=True)
        for key, value in user_data.items():
            setattr(user, key, value)
        session.add(user)
        session.commit()
        session.refresh(user)
        return user
```

### Usando `model_validate` com payloads externos
```python
external_data = {"name": "Bob", "email": "bob@example.com"}
user = UserCreate.model_validate(external_data)
```

### Exemplo async com `Depends`
```python
from typing import Generator
from fastapi import Depends
from sqlmodel import Session

engine = create_engine("sqlite:///database.db", echo=True)

def get_session() -> Generator[Session, None, None]:
    with Session(engine) as session:
        yield session

@app.post("/users-async", response_model=UserRead)
async def create_user_async(user: UserCreate, session: Session = Depends(get_session)):
    db_user = UserTable.from_orm(user)
    session.add(db_user)
    session.commit()
    session.refresh(db_user)
    return db_user

@app.get("/users-async/{user_id}", response_model=UserRead)
async def read_user_async(user_id: int, session: Session = Depends(get_session)):
    user = session.get(UserTable, user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

### Observações sobre `Depends` e sessions
- `Depends(get_session)` cria uma sessão de banco por requisição.
- Use `yield` no dependency para garantir que a sessão seja fechada corretamente.
- Em produção, prefira `AsyncEngine` e `AsyncSession` quando suportado pelo driver.

---

## ⚙️ Settings e variáveis de ambiente
### Configuração básica
```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    DATABASE_URL: str
    SECRET_KEY: str
    DEBUG: bool = False
    model_config = SettingsConfigDict(env_file='.env', env_file_encoding='utf-8')
```

### `.env`
```env
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
SECRET_KEY=super-secret-key-123
DEBUG=True
```

### Configurações avançadas
- `env_prefix='APP_'`
- `env_file = '.env.local'`
- `extra='ignore'`
- `case_sensitive=True`

---

## 🧪 Erros e mensagens de validação
### Capturando erros
```python
try:
    UserSchema(username='jv', age=150, email='invalido')
except ValidationError as e:
    print(e.errors())
```

### Exemplo de erro
```json
[
  {"loc": ["username"], "msg": "string does not match regex", "type": "value_error.str.regex"},
  {"loc": ["age"], "msg": "ensure this value is less than or equal to 120", "type": "value_error.number.not_le"},
  {"loc": ["email"], "msg": "value is not a valid email address", "type": "model_type=..."}
]
```

---

## 🧩 Custom Types e extensões
### Tipos personalizados
```python
from pydantic import BaseModel
from typing import NewType

PositiveInt = NewType('PositiveInt', int)

class Model(BaseModel):
    value: PositiveInt
```

### Campos privados
```python
from pydantic import PrivateAttr

class Counter(BaseModel):
    _count: int = PrivateAttr(default=0)
```

### Computed fields
```python
from pydantic import computed_field

class Product(BaseModel):
    price: float
    qty: int

    @computed_field
    @property
    def total(self) -> float:
        return self.price * self.qty
```

---

## 📈 Performance e boas práticas
- Use `model_construct()` quando a validação não é necessária.
- Prefira `extra='ignore'` em APIs públicas para evitar validação desnecessária.
- Evite validações pesadas dentro de loops.
- Use `field_validator` em vez de `root_validator` quando possível.
- Normalize strings com `str_strip_whitespace=True` no `ConfigDict`.

---

## ✅ Checklist do Pydantic
- [ ] Use `BaseModel` para schemas de dados.
- [ ] Defina campos com `Field(...)` para regras e documentação.
- [ ] Preferências de configuração em `ConfigDict`.
- [ ] Use `model_validate()` para parse de dados externos.
- [ ] Use `model_dump()` / `model_dump_json()` para serialização.
- [ ] Trate e logue `ValidationError` corretamente.
- [ ] Use `BaseSettings` para configurações de ambiente.
- [ ] Configure `extra='forbid'` para segurança quando adequado.
- [ ] Use `from_attributes=True` para ORM Mode.
- [ ] Prefira tipos nativos como `EmailStr`, `HttpUrl`, `SecretStr`.

---

## 💡 Observações finais
Pydantic v2 é poderoso e expressivo. Este guia cobre desde o uso básico até padrões avançados, ajudando a tornar aplicações Python mais seguras e robustas.
