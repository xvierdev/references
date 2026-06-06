# 🐳 Bíblia do Docker & Docker Compose

Guia completo para construir, empacotar e orquestrar aplicações com Docker e Docker Compose. Abrange desde conceitos básicos até práticas de produção.

---

## 📑 Sumário
1. [🐳 O que é Docker](#-o-que-é-docker)
2. [🛠️ Instalação Rápida](#️-instalação-rápida)
3. [🏗️ Conceitos: Imagens vs Containers](#️-conceitos-imagens-vs-containers)
4. [💻 Docker CLI: Guia de Referência](#-docker-cli-guia-de-referência)
5. [📝 Dockerfile: Construção de Imagens](#-dockerfile-construção-de-imagens)
6. [⚡ Multi-stage Builds (Alta Performance)](#-multi-stage-builds-alta-performance)
7. [💾 Volumes e Bind Mounts (Persistência)](#-volumes-e-bind-mounts-persistência)
8. [🌐 Redes Docker (Comunicação)](#-redes-docker-comunicação)
9. [🐙 Docker Compose: O Maestro](#-docker-compose-o-maestro)
10. [🚀 Recursos Avançados do Compose](#-recursos-avançados-do-compose)
11. [🧪 Exemplo Completo: Full-Stack App](#-exemplo-completo-full-stack-app)
12. [🩺 Diagnóstico e Monitoramento](#-diagnóstico-e-monitoramento)
13. [🛡️ Segurança e Melhores Práticas](#️-segurança-e-melhores-práticas)
14. [✅ Checklist de Produção](#-checklist-de-produção)

---

## 1. 🐳 O que é Docker
O Docker empacota aplicações e suas dependências em imagens imutáveis que rodam em containers isolados. Isso garante que "funciona na minha máquina" também signifique "funciona no servidor".

## 2. 🛠️ Instalação Rápida
- **Linux**: Use o script oficial: `curl -fsSL https://get.docker.com | sh`.
- **Windows/Mac**: Instale o Docker Desktop.
- **Pós-instalação (Linux)**: Adicione seu usuário ao grupo docker: `sudo usermod -aG docker $USER`.

## 3. 🏗️ Conceitos: Imagens vs Containers
- **Imagem**: O "molde" ou blueprint (ReadOnly). Ex: `python:3.11`.
- **Container**: A instância em execução do molde (Read/Write layer no topo).
- **Registry**: Onde as imagens moram (Docker Hub, GHCR, AWS ECR).

## 4. 💻 Docker CLI: Guia de Referência

### Gestão de Containers
```bash
docker run -it --name meu-app -p 80:80 image:tag  # Iniciar com terminal interativo
docker start/stop/restart <name_or_id>             # Ciclo de vida
docker ps                                          # Listar rodando
docker ps -a                                       # Listar todos (até parados)
docker rm -f $(docker ps -aq)                      # Remover TODOS os containers
docker exec -it <container> bash                   # Abrir terminal dentro do container
docker logs -f --tail 100 <container>              # Logs em tempo real (últimas 100 linhas)
docker cp <host_path> <container>:<dest_path>      # Copiar arquivos para dentro/fora
```

### Gestão de Imagens
```bash
docker build -t my-image:v1 .                      # Construir imagem
docker images                                      # Listar locais
docker rmi <image_id>                              # Remover imagem
docker tag my-image:v1 user/repo:v1                # Taggear para upload
docker push user/repo:v1                           # Enviar para o registry
docker pull user/repo:v1                           # Baixar imagem
docker history <image>                             # Ver camadas da imagem
```

### Limpeza e Manutenção (O Essencial)
```bash
docker system prune -a --volumes                   # Limpeza PROFUNDA (apaga tudo não usado)
docker image prune                                 # Apaga imagens "dangling" (sem tag)
docker volume prune                                # Apaga volumes órfãos
```

### Inspeção e Monitoramento
```bash
docker inspect <id>                                # JSON com todos os detalhes do objeto
docker stats                                       # CPU, RAM e I/O em tempo real (estilo 'top')
docker top <container>                             # Processos rodando dentro do container
docker diff <container>                            # O que mudou no filesystem desde o início
```

## 5. 📝 Dockerfile: Construção de Imagens
O Dockerfile é a "receita" para construir sua imagem.

### Instruções Essenciais
- **FROM**: Imagem base (prefira versões `-slim` ou `-alpine` para menor tamanho).
- **WORKDIR**: Define o diretório de trabalho (evita usar caminhos absolutos em tudo).
- **COPY vs ADD**: Use `COPY`. O `ADD` só é necessário para baixar URLs ou extrair `.tar.gz` automaticamente.
- **RUN**: Executa comandos no build (instalação de pacotes).
- **ENV**: Define variáveis de ambiente permanentes.
- **ARG**: Variáveis apenas para o tempo de build (`docker build --build-arg ...`).
- **USER**: Define um usuário não-root (Segurança!).
- **EXPOSE**: Documenta qual porta o container escuta.
- **ENTRYPOINT vs CMD**:
  - `ENTRYPOINT`: O comando que sempre será executado.
  - `CMD`: Argumentos padrão para o entrypoint (podem ser sobrescritos no `docker run`).

### Exemplo Profissional
```dockerfile
# 1. Definição de argumentos e imagem base
ARG PYTHON_VERSION=3.11-slim
FROM python:${PYTHON_VERSION}

# 2. Configurações de sistema e ambiente
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    APP_HOME=/app

WORKDIR $APP_HOME

# 3. Usuário de segurança
RUN adduser --system --group appuser

# 4. Instalação de dependências (Cache Otimizado)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 5. Copia o código e troca o dono
COPY . .
RUN chown -R appuser:appuser $APP_HOME

# 6. Troca para usuário comum
USER appuser

# 7. Execução
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 6. ⚡ Multi-stage Builds (Alta Performance)
Reduza o tamanho da sua imagem final em até 90% separando o ambiente de build do ambiente de execução.

```dockerfile
# ESTÁGIO 1: Compilação/Build
FROM python:3.11-slim as builder
WORKDIR /build
RUN apt-get update && apt-get install -y gcc g++ libffi-dev
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# ESTÁGIO 2: Imagem Final (Mínima)
FROM python:3.11-slim
WORKDIR /app

# Copia apenas as libs instaladas, sem as ferramentas de build (gcc, etc)
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

COPY . .
EXPOSE 8000
CMD ["uvicorn", "app:main", "--host", "0.0.0.0"]
```

## 7. 💾 Volumes e Bind Mounts (Persistência)
Sem volumes, todos os dados criados no container são perdidos ao removê-lo.

- **Bind Mounts**: Mapeia uma pasta do seu computador para dentro do container.
  - Ideal para: Desenvolvimento (Hot Reload).
  - Ex: `-v ./meu-codigo:/app`
- **Named Volumes**: Criados e gerenciados pelo Docker em `/var/lib/docker/volumes/`.
  - Ideal para: Bancos de dados em produção.
  - Ex: `-v pg_data:/var/lib/postgresql/data`

### Comandos de Volume
```bash
docker volume ls                   # Listar volumes
docker volume create meu_vol       # Criar manualmente
docker volume inspect meu_vol      # Ver onde os dados estão no host
```

## 8. 🌐 Redes Docker (Comunicação)
Permite que containers se comuniquem pelo nome do serviço/container.

- **Bridge (Padrão)**: Cria uma rede virtual interna.
- **Host**: O container usa a rede do seu computador diretamente (sem isolamento de portas).
- **None**: Sem rede.
- **Overlay**: Para comunicação entre múltiplos servidores (Docker Swarm).

### Exemplo de rede customizada
```bash
docker network create minha-rede
docker run -d --name db --network minha-rede postgres
docker run -it --network minha-rede alpine ping db  # Funciona pelo nome!
```

---

## 🩺 Diagnóstico e Monitoramento
Ferramentas para quando o container não sobe ou está lento.

- **Logs**: `docker logs -f <id>` (Primeiro passo sempre).
- **Eventos**: `docker events` (Mostra o que o Docker está fazendo em tempo real).
- **Stats**: `docker stats` (Monitorar consumo de RAM/CPU).
- **Portas**: `docker port <container>` (Ver quais portas estão mapeadas).
- **Healthchecks**: Verifique o status com `docker ps`. Se estiver `(unhealthy)`, o app interno falhou.

## 9. 🐙 Docker Compose: O Maestro
O Compose permite subir toda a sua infraestrutura (App, Banco, Redis, Worker) com um único comando.

### Comandos Essenciais do Compose
```bash
docker compose up -d              # Sobe tudo em background
docker compose down               # Para e REMOVE containers e redes
docker compose stop/start         # Apenas pausa/resume (não remove)
docker compose logs -f <servico>  # Logs específicos de um serviço
docker compose ps                 # Status dos serviços do projeto
docker compose exec <servico> sh  # Entrar no container de um serviço
docker compose build --no-cache   # Forçar rebuild total
docker compose config             # Valida o arquivo YAML
```

### Exemplo `docker-compose.yml` Robusto
```yaml

services:
  app:
    build: 
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    env_file: .env
    environment:
      - DEBUG=true
    depends_on:
      db:
        condition: service_healthy # Só sobe se o DB passar no healthcheck
    networks:
      - backend

  db:
    image: postgres:15-alpine
    volumes:
      - pg_data:/var/lib/postgresql/data
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
  pg_data:
```

---

## 🚀 Recursos Avançados do Compose

### 🏗️ Profiles (Perfis)
Permite subir apenas partes da sua infraestrutura. Útil para separar ferramentas de `debug` ou `admin`.
```yaml
services:
  pgadmin:
    image: dpage/pgadmin4
    profiles: ["debug"] # Só sobe se usar: docker compose --profile debug up
```

### 🔗 Depends On (Condicional)
O `depends_on` puro apenas garante a ordem de início. O `condition: service_healthy` garante que o banco já está pronto para receber conexões antes do seu app tentar conectar.

### 🧩 Extensões e Reuso (YAML Anchors)
Evite repetição de código no seu YAML.
```yaml
x-common-keys: &common
  restart: always
  networks:
    - backend

services:
  app1:
    <<: *common
    image: app1
  app2:
    <<: *common
    image: app2
```

## 🧪 Exemplo Completo: Full-Stack App
Um sistema robusto geralmente combina:
1. **Dockerfile Multi-stage** para o Frontend (React) e Backend (Python).
2. **Nginx** como Proxy Reverso para rotear tráfego.
3. **Postgres** com volumes persistentes.
4. **Redis** para cache.
5. **Networks isoladas** (frontend e backend).

---

## 🛡️ Segurança e Melhores Práticas
- **Non-Root User**: Crie um usuário no Dockerfile e use `USER appuser`.
- **Read-Only Filesystem**: Se o seu app não precisa escrever no disco, use `read_only: true` no Compose.
- **Docker Secrets**: Para senhas em produção, evite `-e PASSWORD=123`. Use `secrets`.
- **Scan de Imagens**: Use `docker scan my-image` ou **Trivy** para achar vulnerabilidades.
- **Limites de Recursos**: Sempre defina `deploy: resources: limits` para evitar que um container trave o servidor.

## 🚀 CI/CD e Deploy
1. **Build e Push**: Construa a imagem no GitHub Actions/GitLab CI e envie para um Registry.
2. **Tags Imutáveis**: Nunca use apenas `:latest` em produção. Use o hash do commit ou versão (`:v1.0.4`).
3. **Pull Automático**: Use ferramentas como **Watchtower** ou pipelines para atualizar containers.
4. **Orquestração**: Para alta escala, o próximo passo após o Compose é o **Kubernetes**.

---

## ✅ Checklist de Produção

- [ ] A imagem final usa um estágio limpo (sem compiladores/headers)?
- [ ] O container roda com um usuário **não-root**?
- [ ] Existe um **Healthcheck** configurado para serviços críticos?
- [ ] Os logs estão sendo enviados para o `stdout/stderr` (padrão Docker)?
- [ ] Limites de **Memória e CPU** foram definidos no Compose?
- [ ] **Restart Policy** está definida como `unless-stopped`?
- [ ] Volumes estão usando caminhos absolutos ou volumes nomeados?
- [ ] O arquivo `.env` não está sendo enviado para o repositório Git?
- [ ] Todas as portas desnecessárias estão fechadas (apenas 80/443 expostas)?

---

## Recursos
- Documentação oficial: https://docs.docker.com/
- Compose: https://docs.docker.com/compose/
