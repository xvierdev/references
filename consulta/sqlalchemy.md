# 📊 Guia SQLAlchemy 2.0+ (Estilo Moderno)

O SQLAlchemy 2.0 manteve o poder do Core e do ORM, mas adicionou uma API mais clara, baseada em Type Hints, melhor suporte a async e maior compatibilidade com o Python moderno.

---

## 📑 Sumário
1. [🔧 Instalação e Engine](#-instalação-e-engine)
2. [🏗️ Definição de Modelos e Declarative Base](#-definição-de-modelos-e-declarative-base)
3. [📚 Metadata e Criação de Schema](#-metadata-e-criação-de-schema)
4. [🧠 Sessões e Ciclo de Vida](#-sessões-e-ciclo-de-vida)
5. [🛠️ CRUD com ORM](#-crud-com-orm)
6. [📌 Consultas e Expressões](#-consultas-e-expressões)
7. [🤝 Joins e Relacionamentos](#-joins-e-relacionamentos)
8. [⚡ Carregamento e Otimizações](#-carregamento-e-otimizações)
9. [🧪 Async com SQLAlchemy](#-async-com-sqlalchemy)
10. [🧾 Resultados: execute, scalars e mappings](#-resultados-execute-scalars-e-mappings)
11. [🧾 Mapped vs mapped_column](#-mapped-vs-mapped_column)
12. [💡 Dicas e Boas Práticas](#-dicas-e-boas-práticas)

---

## 🔧 Instalação e Engine

```bash
pip install sqlalchemy
```

### Engine básica
```python
from sqlalchemy import create_engine

engine = create_engine("sqlite:///meubanco.db", echo=True)
```

### Engine para PostgreSQL
```python
engine = create_engine(
    "postgresql+psycopg2://user:pass@localhost/dbname",
    echo=False,
    future=True,
    pool_size=10,
    max_overflow=20,
)
```

---

## 🏗️ Definição de Modelos e Declarative Base

### Base com `DeclarativeBase`
```python
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship
from sqlalchemy import String, ForeignKey
from typing import List, Optional

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(30), nullable=False)
    email: Mapped[Optional[str]] = mapped_column(String(120), unique=True)

    posts: Mapped[List["Post"]] = relationship(back_populates="author")

class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(200), nullable=False)
    author_id: Mapped[int] = mapped_column(ForeignKey("users.id"))

    author: Mapped["User"] = relationship(back_populates="posts")
```

### Colunas comuns
```python
from sqlalchemy import Integer, Boolean, DateTime, Text, Float

age: Mapped[int] = mapped_column(Integer)
is_active: Mapped[bool] = mapped_column(Boolean, default=True)
created_at: Mapped[datetime.datetime] = mapped_column(DateTime)
```

---

## 📚 Metadata e Criação de Schema

### Criar tabelas no banco
```python
Base.metadata.create_all(engine)
```

### Dropar tabelas
```python
Base.metadata.drop_all(engine)
```

### Declarar índices e constraints
```python
from sqlalchemy import Index, UniqueConstraint

class Product(Base):
    __tablename__ = "products"
    __table_args__ = (
        UniqueConstraint("sku", name="uq_product_sku"),
        Index("ix_product_name", "name"),
    )
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50))
    sku: Mapped[str] = mapped_column(String(20))
```

---

## 🧠 Sessões e Ciclo de Vida

### Sessão básica
```python
from sqlalchemy.orm import Session

with Session(engine) as session:
    ...
```

### `sessionmaker`
```python
from sqlalchemy.orm import sessionmaker

SessionLocal = sessionmaker(bind=engine, expire_on_commit=False, future=True)

with SessionLocal() as session:
    ...
```

### Flush, Commit e Rollback
- `session.flush()`: envia comandos pendentes ao banco, mas não finaliza a transação.
- `session.commit()`: confirma a transação.
- `session.rollback()`: desfaz a transação.

### Estados de objeto
- `transient`: criado, sem sessão
- `pending`: adicionado à sessão, mas não persistido
- `persistent`: salvo e associado à sessão
- `detached`: desanexado da sessão

### `expire_on_commit`
- `True`: objetos são expirados após commit e recarregam do banco na próxima leitura.
- `False`: os valores permanecem em cache na sessão.

---

## 🛠️ CRUD com ORM

### INSERT simples
```python
with Session(engine) as session:
    user = User(name="Xvier", email="x@example.com")
    session.add(user)
    session.commit()
```

### INSERT em lote
```python
with Session(engine) as session:
    users = [User(name="A"), User(name="B")]
    session.add_all(users)
    session.commit()
```

### SELECT de objetos
```python
from sqlalchemy import select

stmt = select(User).where(User.name == "Xvier")
with Session(engine) as session:
    user = session.execute(stmt).scalars().first()
```

### UPDATE via objeto
```python
with Session(engine) as session:
    user = session.get(User, 1)
    user.name = "Xvier Updated"
    session.commit()
```

### DELETE via objeto
```python
with Session(engine) as session:
    user = session.get(User, 1)
    session.delete(user)
    session.commit()
```

---

## 📌 Consultas e Expressões

### Seleção de colunas
```python
stmt = select(User.id, User.name)
```

### Filtros
```python
stmt = select(User).where(User.name.ilike("%xvier%"))
```

### Ordenação, limite e offset
```python
stmt = select(User).order_by(User.name.desc()).limit(10).offset(20)
```

### Agregação com `func`
```python
from sqlalchemy import func

stmt = (
    select(User.name, func.count(Post.id))
    .join(Post)
    .group_by(User.name)
)
```

### Joins explícitos
```python
stmt = select(Post, User).join(User, Post.author_id == User.id)
```

### Alias e subqueries
```python
from sqlalchemy import alias

user_alias = alias(User, name="u")
stmt = select(user_alias.c.id, user_alias.c.name)
```

### SQL textual
```python
from sqlalchemy import text
stmt = text("SELECT * FROM users WHERE name = :name")
with Session(engine) as session:
    result = session.execute(stmt, {"name": "Xvier"})
```

---

## 🤝 Joins e Relacionamentos

### Relacionamento 1:N
```python
class User(Base):
    posts: Mapped[List["Post"]] = relationship(back_populates="author")

class Post(Base):
    author: Mapped["User"] = relationship(back_populates="posts")
```

### Relacionamento N:N com tabela de associação
```python
from sqlalchemy import Table, Column, Integer, ForeignKey

post_tag = Table(
    "post_tag",
    Base.metadata,
    Column("post_id", ForeignKey("posts.id"), primary_key=True),
    Column("tag_id", ForeignKey("tags.id"), primary_key=True),
)

class Tag(Base):
    posts: Mapped[List["Post"]] = relationship(
        secondary=post_tag,
        back_populates="tags"
    )

class Post(Base):
    tags: Mapped[List["Tag"]] = relationship(
        secondary=post_tag,
        back_populates="posts"
    )
```

### Opções de relacionamento
- `back_populates`
- `lazy="select"` (padrão)
- `lazy="joined"` / `selectin` para eager loading
- `cascade="all, delete-orphan"`
- `uselist=False` para 1:1

### Acessando relacionamentos
```python
with Session(engine) as session:
    user = session.get(User, 1)
    for post in user.posts:
        print(post.title)
```

---

## ⚡ Carregamento e Otimizações

### Eager loading
```python
from sqlalchemy.orm import selectinload, joinedload

stmt = select(User).options(selectinload(User.posts))
```

### Subquery load
```python
from sqlalchemy.orm import subqueryload
stmt = select(User).options(subqueryload(User.posts))
```

### `with_loader_criteria`
```python
from sqlalchemy.orm import with_loader_criteria
stmt = select(User).options(
    with_loader_criteria(Post, Post.title != "draft")
)
```

### Evitando N+1
- Use `selectinload` ou `joinedload` para carregar relacionamentos em lote.

### Batch inserts e flush eficiente
```python
session.add_all([User(name=f"User{i}") for i in range(1000)])
session.commit()
```

---

## 🧪 Async com SQLAlchemy

### Engine assíncrona
```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession

ASYNC_DATABASE_URL = "sqlite+aiosqlite:///./test.db"
engine = create_async_engine(ASYNC_DATABASE_URL, echo=True)

AsyncSessionLocal = async_sessionmaker(
    bind=engine,
    class_=AsyncSession,
    expire_on_commit=False,
)
```

### Sessão assíncrona
```python
async with AsyncSessionLocal() as session:
    async with session.begin():
        user = User(name="Async", email="async@example.com")
        session.add(user)
```

### SELECT assíncrono
```python
stmt = select(User).where(User.name == "Async")
result = await session.execute(stmt)
users = result.scalars().all()
```

### INSERT/UPDATE/DELETE assíncrono
```python
from sqlalchemy import insert, update, delete

await session.execute(insert(User).values(name="A"))
await session.execute(
    update(User).where(User.id == 1).values(name="B")
)
await session.execute(delete(User).where(User.id == 1))
await session.commit()
```

---

## 🧾 Resultados: execute, scalars e mappings

- `session.execute(stmt)`: retorna `Result` com linhas e colunas.
- `scalars()`: retorna objetos ou valores individuais.
- `mappings()`: retorna dicionários por linha.

```python
result = session.execute(select(User.name, User.email))
for row in result.mappings():
    print(row["name"], row["email"])
```

---

## 🧾 Mapped vs mapped_column

- `Mapped[tipo]`: type hint para SQLAlchemy e IDE.
- `mapped_column(...)`: configura propriedades da coluna no banco.

```python
username: Mapped[str] = mapped_column(String(50), unique=True)
```

---

## 💡 Dicas e Boas Práticas

- Use `Session(engine)` ou `async with AsyncSessionLocal()` para controlar recursos.
- Prefira `select()` em vez de `session.query()` no estilo 2.0.
- Use `future=True` no `create_engine` para API moderna.
- Ajuste `expire_on_commit=False` se quiser evitar recarregamentos automáticos.
- Habilite `echo=True` apenas para depuração.
- Use `joinedload` / `selectinload` para evitar N+1.
- Prefira `mappings()` para retorno JSON direto.
- Evite `lazy="joined"` em coleções muito grandes se não precisar de todos os dados.
- Use transações explícitas com `session.begin()` quando precisar de atomicidade.
