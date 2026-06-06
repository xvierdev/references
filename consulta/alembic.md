# ⚗️ Bíblia do Alembic: Gestão de Banco em Produção

O Alembic é a ferramenta oficial de migração de banco de dados para SQLAlchemy. Ele mantém o histórico de mudanças do schema e permite aplicar ou desfazer versões em bancos de dados diferentes com segurança, permitindo o versionamento de infraestrutura como código.

---

## 📑 Sumário
1. [🔧 Instalação e Configuração Detalhada](#-instalação-e-configuração-detalhada)
2. [🚀 Inicialização do Ambiente](#-inicialização-do-ambiente)
3. [📂 Estrutura Completa do Diretório](#-estrutura-completa-do-diretório)
4. [⚙️ Configurações do env.py (Múltiplos Cenários)](#️-configurações-do-envpy-múltiplos-cenários)
5. [📋 Referência de Comandos CLI](#-referência-de-comandos-cli)
6. [🛠️ Uso do Autogenerate: Poder e Limites](#️-uso-do-autogenerate-poder-e-limites)
7. [🧾 Revisões e Edição Manual (API op)](#-revisões-e-edição-manual-api-op)
8. [🧩 Batch Mode para SQLite](#-batch-mode-para-sqlite)
9. [🔄 Aplicando e Revertendo Migrações](#-aplicando-e-revertendo-migrações)
10. [👥 Workflow de Time (Merge Heads)](#-workflow-de-time-merge-heads)
11. [📊 Data Migrations (Transformação de Dados)](#-data-migrations-transformação-de-dados)
12. [🛡️ Migrações Seguras em Produção](#️-migrações-seguras-em-produção)
13. [🌐 Modo Offline vs Online](#-modo-offline-vs-online)
14. [🧠 Bootstrapping e Scripts Customizados](#-bootstrapping-e-scripts-customizados)
15. [🚀 Automação em CI/CD](#-automação-em-cicd)
16. [💡 Dicas e Boas Práticas de Engenharia](#-dicas-e-boas-práticas-de-engenharia)
17. [✅ Checklist de Migração Profissional](#-checklist-de-migração-profissional)

---

## 🔧 Instalação e Configuração Detalhada

Para começar, instale o Alembic e o SQLAlchemy. É recomendável usar um ambiente virtual.

```bash
pip install alembic sqlalchemy
```

Se você estiver utilizando bancos de dados específicos, instale os drivers correspondentes:
- **PostgreSQL**: `pip install psycopg2-binary` (ou `asyncpg` para async)
- **MySQL**: `pip install mysqlclient` ou `pymysql`
- **SQLite**: Já incluído no Python padrão.

---

## 🚀 Inicialização do Ambiente

Para criar a estrutura inicial do Alembic no seu projeto, execute:

```bash
alembic init alembic
```

Se você estiver em um ambiente assíncrono (como FastAPI com SQLAlchemy Async), pode inicializar com o template async:
```bash
alembic init -t async alembic
```

---

## 📂 Estrutura Completa do Diretório

Ao inicializar, o Alembic cria os seguintes arquivos:

- **`alembic.ini`**: O arquivo de configuração principal. Contém a URL do banco de dados (pode ser sobrescrita) e configurações de logging.
- **`alembic/env.py`**: O script mais importante. Ele é carregado toda vez que o Alembic é executado. Ele configura o motor de conexão e o contexto de migração.
- **`alembic/versions/`**: O diretório que armazena seus scripts de migração. Cada mudança no banco gera um novo arquivo aqui.
- **`alembic/script.py.mako`**: O template usado para gerar novos scripts de migração. Você pode editá-lo para adicionar imports automáticos (como GeoAlchemy ou UUIDs).

---

## ⚙️ Configurações do env.py (Múltiplos Cenários)

O `env.py` precisa saber qual é o `metadata` dos seus modelos para comparar com o banco.

### 1. Exemplo Mínimo (SQLAlchemy ORM clássico)
```python
from my_app.models import Base
target_metadata = Base.metadata
```

### 2. Exemplo Moderno (SQLAlchemy 2.0+ com DeclarativeBase)
```python
from my_app.database import Base
import my_app.models # Garante que todos os modelos sejam carregados
target_metadata = Base.metadata
```

### 3. Usando Variáveis de Ambiente (Segurança em Produção)
No `env.py`, evite deixar senhas expostas. Use:
```python
import os
from alembic import context

config = context.config
section = config.get_section(config.config_ini_section)
section["sqlalchemy.url"] = os.getenv("DATABASE_URL")

# Proceda com a criação da engine usando a seção atualizada
```

---

## 📋 Referência de Comandos CLI

| Comando | Função |
|---|---|
| `alembic revision -m "msg"` | Cria uma revisão manual vazia. |
| `alembic revision --autogenerate -m "msg"` | Gera a migração comparando código vs banco. |
| `alembic upgrade head` | Atualiza o banco para a versão mais recente. |
| `alembic upgrade <revision_id>` | Vai para uma versão específica. |
| `alembic downgrade -1` | Desfaz a última migração aplicada. |
| `alembic downgrade base` | Reverte todas as migrações até o estado inicial. |
| `alembic current` | Exibe a revisão atualmente aplicada no banco. |
| `alembic history --verbose` | Mostra a árvore cronológica de todas as revisões. |
| `alembic show <rev>` | Detalha as mudanças de uma revisão específica. |
| `alembic stamp head` | Marca o banco como "atualizado" sem rodar nenhum SQL. |
| `alembic check` | Verifica se existem mudanças no código não migradas. |

---

## 🛠️ Uso do Autogenerate: Poder e Limitas

O comando `--autogenerate` economiza muito tempo, mas não é infalível.

### ✅ O que ele detecta automaticamente:
- Criação e exclusão de tabelas.
- Adição e remoção de colunas.
- Mudanças de nulidade (`nullable`).
- Alterações básicas em índices e constraints de chave estrangeira.
- Mudanças de tipo de coluna (se `compare_type=True` estiver ativado no `context.configure`).

### ❌ O que ele NÃO detecta (Você deve escrever manualmente):
- Mudanças de nome de tabelas ou colunas (ele verá como um DROP e um ADD).
- Alterações em constraints de CHECK customizadas.
- Mudanças em Triggers ou Views.
- Alterações em tipos de dados muito específicos de cada provedor (ex: ENUMs customizados no Postgres).

---

## 🧾 Revisões e Edição Manual (API op)

A biblioteca `alembic.op` fornece comandos para manipular o schema.

### Exemplos comuns:
```python
from alembic import op
import sqlalchemy as sa

def upgrade():
    # Criar tabela
    op.create_table('tasks',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('title', sa.String(100), nullable=False)
    )
    # Adicionar coluna
    op.add_column('users', sa.Column('is_verified', sa.Boolean(), server_default='false'))
    # Criar Índice
    op.create_index('ix_user_name', 'users', ['name'])
    # SQL Puro
    op.execute("UPDATE users SET is_verified = true WHERE email LIKE '%@google.com'")

def downgrade():
    op.drop_table('tasks')
    op.drop_column('users', 'is_verified')
```

---

## 🧩 Batch Mode para SQLite

O SQLite tem suporte limitado para `ALTER TABLE`. O Alembic resolve isso recriando a tabela e copiando os dados.

```python
def upgrade():
    with op.batch_alter_table("users") as batch_op:
        batch_op.add_column(sa.Column("middle_name", sa.String(50)))
        batch_op.drop_column("temp_field")
```

---

## 🔄 Aplicando e Revertendo Migrações

### Fluxo Recomendado
1. Faça a alteração no seu modelo SQLAlchemy.
2. Rode `alembic revision --autogenerate -m "descrição"`.
3. Abra o arquivo gerado em `versions/` e verifique se o código condiz com o esperado.
4. Rode `alembic upgrade head`.

### Revertendo
Se algo der errado, você pode voltar rapidamente:
```bash
alembic downgrade -1
```

---

## 👥 Workflow de Time (Merge Heads)

Quando dois desenvolvedores criam migrações em paralelo, o Alembic detecta um conflito de "duas cabeças".

**Como identificar:**
`alembic heads` -> Mostrará dois IDs de revisão.

**Como resolver:**
```bash
alembic merge heads -m "merge de branches paralelos"
```
Isso criará um novo arquivo de revisão que aponta para os dois conflituosos como pais, unificando a história.

---

## 📊 Data Migrations (Transformação de Dados)

Use o Alembic para popular tabelas ou converter dados durante o deploy.

```python
def upgrade():
    # Exemplo: concatenar nome e sobrenome
    op.execute("UPDATE users SET full_name = first_name || ' ' || last_name")
```

---

## 🛡️ Migrações Seguras em Produção

### Colunas NOT NULL
Adicionar uma coluna `NOT NULL` em uma tabela que já tem dados causará erro.
1. Adicione a coluna como `nullable=True`.
2. Popule a coluna com valores padrão via `op.execute`.
3. Mude a coluna para `nullable=False` usando `op.alter_column`.

### Locks no Postgres
Operações como `CREATE INDEX` podem travar a tabela para leitura/escrita. Use o modo `CONCURRENTLY` se o banco for muito grande.

---

## 🌐 Modo Offline vs Online

### Modo Online (Padrão)
O Alembic se conecta ao banco via engine e executa o SQL imediatamente.

### Modo Offline
Gera o script SQL sem tocar no banco de dados. Útil para auditoria ou quando o deploy é feito por um DBA.
```bash
alembic upgrade head --sql > migration_v2.sql
```

---

## 🧠 Bootstrapping e Scripts Customizados

### script.py.mako
Você pode customizar o template para que todas as migrações incluam o logo da sua empresa ou imports de tipos customizados como `CITEXT` ou `GEOMETRY`.

### env.py customizado
Você pode adicionar lógica no `env.py` para ignorar tabelas específicas (como as do WordPress ou outras ferramentas que compartilham o mesmo banco).

---

## 🚀 Automação em CI/CD

- **Check de Sincronismo**: No seu CI, rode `alembic check`. Se ele retornar erro, significa que o desenvolvedor mudou o código mas esqueceu de gerar a migração.
- **Auto-Upgrade**: Em containers, rode `alembic upgrade head` no script de entrypoint antes de iniciar o Gunicorn/Uvicorn.

---

## 💡 Dicas e Boas Práticas de Engenharia

1. **Nunca altere uma migração já enviada para o repositório principal.** Se errou, crie uma nova migração corrigindo o erro.
2. **Teste o Downgrade.** Muitas vezes o `upgrade` funciona mas o `downgrade` falha por causa de constraints.
3. **Use Stamps.** Se você já criou as tabelas manualmente, use `alembic stamp head` para dizer ao Alembic que o banco já está na última versão.
4. **Mantenha o banco limpo.** Delete migrações de "rascunho" antes de fazer o merge na branch `main`.

---

## ✅ Checklist de Migração Profissional

- [ ] A migração foi revisada linha a linha?
- [ ] O `downgrade` foi testado e reverte o banco ao estado original exato?
- [ ] Existe tratamento para dados existentes (campos NOT NULL)?
- [ ] O conflito de `heads` foi resolvido antes do Merge Request?
- [ ] As variáveis de ambiente estão configuradas no `env.py` para evitar exposição de credenciais?
- [ ] Foi feito um backup do banco antes de rodar migrações críticas em produção?
- [ ] O tempo de execução da migração foi estimado para evitar downtime excessivo?

---

## 📚 Referências Rápidas
- Documentação Oficial: [alembic.sqlalchemy.org](https://alembic.sqlalchemy.org/)
- Repositório SQLAlchemy: [github.com/sqlalchemy/alembic](https://github.com/sqlalchemy/alembic)
