# 🏆 O Guia Supremo do Desenvolvedor (The Ultimate Cheat Sheet)

Este documento é a referência definitiva para o ecossistema de desenvolvimento. Ele cobre desde os fundamentos da web até a administração de sistemas, servindo como uma "cola" exaustiva para o dia a dia.

---

## 📑 Sumário
1. [💻 Linguagens e Tecnologias](#-linguagens-e-tecnologias)
2. [🌐 Web: Verbos HTTP e Status Codes](#-web-verbos-http-e-status-codes)
3. [☁️ Cloudflare e Rede](#-cloudflare-e-rede)
4. [🐍 Ecossistema Python (pip vs uv)](#-ecossistema-python-pip-vs-uv)
5. [🚀 Streamlit](#-streamlit)
6. [🛠️ Controle de Versão (Git)](#-controle-de-versão-git)
7. [🗄️ Bancos de Dados](#-bancos-de-dados)
8. [🐳 Containerização (Docker & Compose)](#-containerização-docker--compose)
9. [📋 Templates de Configuração (Boilerplates)](#-templates-de-configuração-boilerplates)
10. [🐚 Terminais: PowerShell](#-terminais-powershell)
11. [🐧 Terminais: Bash e Linux](#-terminais-bash-e-linux)
12. [🩺 Diagnóstico de Sistema e Rede](#-diagnóstico-de-sistema-e-rede)
13. [📚 Bibliotecas Recomendadas](#-bibliotecas-recomendadas)
14. [📖 Glossário Técnico](#-glossário-técnico)

---

## 💻 Linguagens e Tecnologias

### 🌍 Tecnologias Web (Front-end e Estilização)
- **HTML**: Marcação estrutural da web. Frameworks/Ferramentas: (Não se aplica diretamente, mas usa-se Templates como Jinja2).
- **CSS**: Estilização visual. Frameworks: **Bootstrap** (componentes prontos), **Tailwind CSS** (utilitário focado em produtividade).
- **JavaScript**: Lógica de cliente. Frameworks: **React**, **Vue**, **Angular**. Ambiente: **Node.js**.
- **TypeScript**: Superset de JavaScript que adiciona tipagem estática, tornando o código mais seguro e escalável. Frameworks: **Next.js**, **NestJS**, **Express**.

### ⚙️ Linguagens Interpretadas
- **Python**: Focada em legibilidade e produtividade. Frameworks: **Django** (robusto), **FastAPI** (performance), **Flask** (minimalista).
- **Bash**: Linguagem de script nativa para automação em sistemas Unix/Linux.

### 🚀 Linguagens Compiladas (Performance Nativa)
- **Go (Golang)**: Compila para binários estáticos nativos. Focada em concorrência. Frameworks: **Gin**, **Echo**.
- **Rust**: Focada em segurança de memória sem garbage collector. Frameworks: **Actix**, **Axum**.
- **C / C++**: Controle total de hardware. Frameworks: **Qt**, **Boost**.

### ☕ Linguagens Híbridas (Máquina Virtual)
- **Java**: Compila para *Bytecode* e roda na **JVM (Java Virtual Machine)**. Frameworks: **Spring Boot**, **Quarkus**.
- **C#**: Compila para *IL (Intermediate Language)* e roda no **CLR (Common Language Runtime)** via ecossistema .NET. Frameworks: **ASP.NET Core**, **Entity Framework**.

### 📝 Linguagens de Configuração e Dados
- **SQL**: Linguagem padrão para consulta e gerenciamento de bancos de dados relacionais.
- **Markdown**: Linguagem de marcação leve usada para documentação.
- **YAML / TOML / JSON**: Formatos de serialização para configurações e troca de dados.
- **Dockerfile**: Script para automação de infraestrutura imutável.

👉 **[Guia Markdown](./consulta/markdown.md)** | **[Guia SQL](./consulta/sql.md)**

---

## 🟢 O que é Node.js?
O **Node.js** não é uma linguagem, mas um **ambiente de execução (runtime)** que permite rodar JavaScript no lado do servidor (backend), fora do navegador. Ele utiliza o motor V8 do Google Chrome e é altamente eficiente para aplicações assíncronas e em tempo real.

---

## 🌐 Web: Verbos HTTP e Status Codes

### Verbos e Sucesso Esperado
- **GET**: Recupera dados. Sucesso: `200 OK`.
- **POST**: Envia dados para criar um novo recurso. Sucesso: `201 Created`.
- **PUT**: Substitui um recurso inteiro. Sucesso: `200 OK` ou `204 No Content`.
- **PATCH**: Modifica parcialmente um recurso. Sucesso: `200 OK`.
- **DELETE**: Remove um recurso. Sucesso: `204 No Content` ou `200 OK`.
- **OPTIONS**: Consulta métodos permitidos (CORS). Sucesso: `200 OK` ou `204 No Content`.

### Principais Códigos de Status
- `1xx`: Informativo (Processando).
- `2xx`: Sucesso (Tudo certo).
- `3xx`: Redirecionamento (O recurso mudou).
- `4xx`: Erro do Cliente (`400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`).
- `5xx`: Erro do Servidor (`500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`).

---

## ☁️ Cloudflare e Rede

### Cloudflare Tunnel (cloudflared)
- **Função**: Cria um túnel seguro entre sua máquina local e a rede Cloudflare.
- **Uso**: Expor `localhost:8501` para um domínio como `app.meudominio.com` sem abrir portas no router ou lidar com IP dinâmico.
- **Segurança**: Criptografia ponta a ponta e ocultação total do seu IP de origem.

### Comandos de Rede
- `ping google.com`: Testa a conectividade e latência com um servidor remoto.
- `ip addr` (Linux): Lista todas as interfaces de rede e seus respectivos IPs.
- `ipconfig` (Windows): Versão do Windows para listar configurações de IP.
- `nslookup google.com`: Consulta o DNS para traduzir domínios em IPs.
- `curl -I http://site.com`: Verifica apenas o cabeçalho HTTP de uma URL.

---

## 🐍 Ecossistema Python (pip vs uv)

### 📦 pip (O Padrão Clássico)
- `pip install <package>`: Instala um pacote específico do PyPI.
- `pip install -r requirements.txt`: Instala todas as dependências listadas no arquivo.
- `pip uninstall <package>`: Remove um pacote do ambiente.
- `pip list`: Exibe todos os pacotes instalados e suas versões.
- `pip show <package>`: Mostra detalhes de um pacote (localização, autor, dependências).
- `pip freeze > requirements.txt`: Exporta as dependências atuais para um arquivo de texto.

### ⚡ uv (O Gerenciador Ultra-Rápido)
- `uv init`: Cria a estrutura inicial de um projeto Python moderno.
- `uv add <package>`: Instala e adiciona automaticamente ao arquivo de projeto.
- `uv remove <package>`: Remove e limpa o arquivo de projeto.
- `uv sync`: Garante que o ambiente virtual está exatamente igual ao `uv.lock`.
- `uv run script.py`: Executa um script Python usando o ambiente gerenciado pelo uv.
- `uv venv`: Cria um ambiente virtual (.venv) otimizado.
- `uv pip install -e .`: Instala o projeto atual em modo editável.

👉 **[Python Types](./consulta/python_types.md)** | **[Pathlib](./consulta/pathlib.md)** | **[Módulo sys](./consulta/sys.md)** | **[Docstrings](./consulta/docstring.md)** | **[Tempo (Datetime)](./consulta/tempo.md)** | **[Dicas Avançadas](./consulta/python_tips.md)** | **[contextlib](./consulta/contextlib.md)**

---

## 🚀 Streamlit

Este guia foi movido para a bíblia dedicada de Streamlit. Veja a referência completa em:

👉 **[Streamlit (Bíblia)](./consulta/streamlit.md)**

---

## 🛠️ Controle de Versão (Git)

Esta seção foi extraída para a bíblia completa de Git e agora vive em:

👉 **[Git (Bíblia)](./consulta/git.md)**
---

## 🗄️ Bancos de Dados
- **SQLite**: Arquivo local. Simples e rápido. `sqlite:///banco.db`
- **PostgreSQL**: Robusto e completo. `postgresql://user:pass@host:port/db`
- **MySQL/MariaDB**: Popular e veloz. `mysql+pymysql://user:pass@host:port/db`

👉 **[Guia SQL](./consulta/sql.md)** | **[SQLAlchemy 2.0+](./consulta/sqlalchemy.md)** | **[Alembic (Migrações)](./consulta/alembic.md)**

---

## 🐳 Containerização (Docker & Compose)

As referências e exemplos de Docker e Docker Compose foram movidos para a bíblia dedicada de Docker.

👉 **[Docker & Docker Compose (Bíblia)](./consulta/docker.md)**
---

## 📋 Templates de Configuração (Boilerplates)

### 🐳 Exemplo de Dockerfile (Python/Streamlit)
```dockerfile
# 1. Imagem base leve com Python
FROM python:3.11-slim

# 2. Define o diretório de trabalho no container
WORKDIR /app

# 3. Evita que o Python gere arquivos .pyc e permite logs em tempo real
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# 4. Instala dependências do sistema necessárias (opcional)
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# 5. Copia e instala dependências Python (cache otimizado)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 6. Copia o restante do código fonte
COPY . .

# 7. Expõe a porta padrão do Streamlit
EXPOSE 8501

# 8. Comando para rodar a aplicação
CMD ["streamlit", "run", "main.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

### 🐙 Exemplo de docker-compose.yml
```yaml
services:
  web:
    # Constrói a imagem usando o Dockerfile na pasta atual
    build: .
    # Nome da imagem gerada
    image: checklist-app:latest
    # Mapeamento de portas: <host>:<container>
    ports:
      - "8501:8501"
    # Variáveis de ambiente
    environment:
      - DATABASE_URL=sqlite:///./app.db
    # Mapeamento de volumes (Hot Reload no desenvolvimento)
    volumes:
      - .:/app
    # Política de reinicialização
    restart: always

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: app_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 🔥 Ultra Complete Boilerplate (Produção)

#### Dockerfile (Multi-stage Build)
```dockerfile
# ESTÁGIO 1: Build (Instalação de dependências pesadas)
FROM python:3.11-slim as builder
WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
RUN apt-get update && apt-get install -y build-essential gcc
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# ESTÁGIO 2: Final (Imagem leve e segura)
FROM python:3.11-slim
WORKDIR /app
# Copia apenas as dependências instaladas do builder
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
EXPOSE 8000
# Usuário não-root por segurança
RUN useradd -m myuser
USER myuser
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### docker-compose.prod.yml
```yaml
version: '3.8'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend

  db:
    image: postgres:15-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

networks:
  backend:
    driver: bridge

volumes:
  pgdata:

secrets:
  db_password:
    file: ./db_password.txt
```

---

## 🐚 Terminais: PowerShell

- `ls` ou `dir`: Lista arquivos e pastas do diretório atual.
- `cd <caminho>`: Entra em uma pasta específica.
- `mkdir <nome>`: Cria uma nova pasta.
- `ni <nome>`: Cria um novo arquivo vazio (New-Item).
- `rm <nome>`: Remove um arquivo ou pasta.
- `cp <origem> <destino>`: Copia arquivos ou diretórios.
- `mv <origem> <destino>`: Move ou renomeia arquivos e diretórios.
- `cat <arquivo>`: Lê o conteúdo de um arquivo no terminal.
- `Select-String "termo" <arquivo>`: Busca um texto dentro de um arquivo.
- `gc <arquivo> -Wait`: Monitora o arquivo em tempo real (log tailing).
- `tar -cvf backup.tar pasta/`: Compacta uma pasta em formato tar.
- `tar -xvf backup.tar`: Extrai o conteúdo de um arquivo tar.

---

## 🐧 Terminais: Bash e Linux

Comandos e coleções de uso comum para administração e uso diário em sistemas Linux foram movidos para o guia de servidor.

👉 **[Terminais e Administração Linux](./consulta/server.md)**

---

## 🩺 Diagnóstico de Sistema e Rede

- `ping -c 4 8.8.8.8`: Envia 4 pacotes de teste para o DNS do Google.
- `ip a`: Exibe endereços IP, MAC e status das interfaces de rede.
- `top`: Visualização simples de processos (nativo em quase todo Linux).
- `whoami`: Exibe o nome do usuário que está executando o comando.
- `id`: Mostra IDs de usuário e grupos aos quais você pertence.
- `uptime`: Diz há quanto tempo o servidor está ligado.

---

## 📚 Bibliotecas Recomendadas

### 🏗️ APIs e Backend
- **FastAPI**: Framework assíncrono moderno e rápido.
- **Pydantic**: O melhor para validação de dados e tipagem.
- **Loguru**: Torna o logging em Python extremamente simples e bonito.
- **Pytest**: O framework de testes mais popular e fácil de usar para Python.
- **Ruff**: Linter e formatador Python ultra-rápido. Substitui Black, Isort e Flake8.
  - `ruff check .`: Procura por erros e violações de estilo.
  - `ruff check . --fix`: Corrige automaticamente o que for possível.
  - `ruff format .`: Formata o código conforme as PEPs.

👉 **[FastAPI](./consulta/fastapi.md)** | **[Pydantic](./consulta/pydantic.md)** | **[Loguru](./consulta/loguru.md)** | **[Pytest](./consulta/pytest.md)** | **[JWT](./consulta/jwt.md)** | **[bcrypt](./consulta/bcrypt.md)** | **[HTTPX](./consulta/httpx.md)** | **[WebSockets](./consulta/websockets.md)** | **[Makefile](./consulta/make.md)** | **[Telegram Bot](./consulta/pyTelegramBotAPI.md)** | **[Reflex](./consulta/reflex.md)** | **[Criptografia](./consulta/cripto.md)**

### 📊 Engenharia de Dados
- **Polars**: O sucessor espiritual do Pandas, focado em performance (Rust-based).
- **Pandas**: A biblioteca padrão para análise e manipulação de dados em Python.
- **DuckDB**: Consultas SQL ultra-rápidas em arquivos locais (CSV, Parquet).

👉 **[Polars](./consulta/polars.md)** | **[Pandas](./consulta/pandas.md)** | **[NetworkX](./consulta/networkx.md)** | **[anytree](./consulta/anytree.md)** | **[Playwright](./consulta/Playwright.md)** | **[Agno (Agentes IA)](./consulta/agno.md)**

---

## 📖 Glossário Técnico

- **API (Application Programming Interface)**: Conjunto de regras que permite que softwares diferentes se comuniquem.
- **Alembic**: Ferramenta de migrações para bancos de dados SQL em Python.
- **Bash**: Shell e linguagem de comando para sistemas Unix.
- **Branch**: Uma linha de desenvolvimento separada no Git (ramificação).
- **Bootstrap**: Framework CSS para desenvolvimento front-end responsivo.
- **Bytecode**: Formato intermediário de código executado por máquinas virtuais (como a JVM).
- **CORS (Cross-Origin Resource Sharing)**: Mecanismo de segurança que controla o acesso a recursos entre domínios diferentes.
- **CLR (Common Language Runtime)**: Máquina virtual da Microsoft para rodar linguagens do ecossistema .NET.
- **CSS (Cascading Style Sheets)**: Linguagem para estilização de documentos HTML.
- **Container**: Unidade de software padronizada que empacota código e dependências.
- **Commit**: Gravação de um conjunto de alterações no histórico do Git.
- **DDoS (Distributed Denial of Service)**: Ataque que tenta sobrecarregar um servidor com tráfego massivo.
- **DELETE**: Verbo HTTP para remover recursos.
- **Django**: Framework web "high-level" para Python.
- **Docker**: Plataforma para criação e gerenciamento de containers.
- **DNS (Domain Name System)**: O "catálogo" da internet que traduz nomes de domínio em endereços IP.
- **Express**: Framework web minimalista para Node.js.
- **FastAPI**: Framework web moderno focado em performance e tipagem.
- **GET**: Verbo HTTP para recuperar informações de um servidor.
- **Git**: Sistema de controle de versão distribuído.
- **HTML (HyperText Markup Language)**: Linguagem de marcação padrão para páginas web.
- **HTTP (HyperText Transfer Protocol)**: Protocolo base para troca de dados na web.
- **IL (Intermediate Language)**: Código intermediário gerado por compiladores C#.
- **IP (Internet Protocol)**: Endereço numérico que identifica dispositivos em uma rede.
- **JVM (Java Virtual Machine)**: Máquina virtual que executa programas escritos em Java.
- **JavaScript**: Linguagem de programação essencial para a web moderna.
- **JSON (JavaScript Object Notation)**: Formato leve de troca de dados.
- **Markdown**: Linguagem de marcação simples para formatação de texto.
- **Next.js**: Framework React focado em SSR (Server Side Rendering) e performance.
- **Node.js**: Ambiente de execução para rodar JavaScript no servidor.
- **ORM (Object-Relational Mapping)**: Técnica que permite interagir com bancos de dados usando objetos em vez de SQL puro.
- **PATCH**: Verbo HTTP para atualizações parciais de recursos.
- **POST**: Verbo HTTP para enviar novos dados ao servidor.
- **PUT**: Verbo HTTP para substituir ou criar recursos.
- **Pydantic**: Biblioteca Python para validação de dados via Type Hints.
- **Python**: Linguagem de programação versátil e de fácil leitura.
- **React**: Biblioteca JavaScript para construção de interfaces de usuário.
- **REST (Representational State Transfer)**: Estilo de arquitetura para serviços web.
- **SQL (Structured Query Language)**: Linguagem para gerenciar dados em bancos relacionais.
- **SQLite**: Banco de dados relacional contido em um único arquivo.
- **Stash**: Recurso do Git para "esconder" alterações temporárias.
- **Streamlit**: Biblioteca para criar dashboards e apps de dados em minutos usando apenas Python.
- **Tailwind CSS**: Framework CSS focado em utilitários de baixo nível.
- **Tag**: No Git, uma marcação usada para rotular versões específicas (ex: v1.0).
- **TOML (Tom's Obvious, Minimal Language)**: Formato de configuração simples.
- **TypeScript**: Versão de JavaScript com tipagem estática.
- **V8**: Motor de JavaScript de alta performance desenvolvido pelo Google.
- **YAML (YAML Ain't Markup Language)**: Formato de serialização de dados comum em configurações.
