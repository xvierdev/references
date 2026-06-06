# 🛠️ Guia de Automação com Makefile

O `make` é uma ferramenta de automação de compilação e tarefas que utiliza um arquivo chamado `Makefile` para definir um conjunto de tarefas a serem executadas. É amplamente utilizado para simplificar comandos longos e repetitivos.

---

## 📑 Sumário
1. [🏗️ Estrutura Básica](#-estrutura-básica)
2. [📂 Variáveis e Comandos](#-variáveis-e-comandos)
3. [🔧 Regras e Dependências](#-regras-e-dependências)
4. [🪄 Variáveis Avançadas](#-variáveis-avançadas)
5. [🚀 Exemplo Prático para Python](#-exemplo-prático-para-python)
6. [🐳 Deploy Docker](#-deploy-docker)
7. [💡 Dicas e Boas Práticas](#-dicas-e-boas-práticas)
8. [✅ Checklist](#-checklist-de-makefile)

---

## 🏗️ Estrutura Básica
Um Makefile consiste em um conjunto de regras com a seguinte sintaxe:

```makefile
alvo: dependencias
	comando
```
- **Alvo (Target)**: Nome da tarefa (ex: `run`, `test`, `clean`).
- **Dependências**: Outros alvos que devem ser executados antes deste.
- **Comando**: A instrução shell a ser executada. **Importante:** Deve ser precedido por um **TAB** (não espaços).

---

## 📂 Variáveis e Comandos
Você pode definir variáveis para tornar o Makefile mais flexível.

```makefile
PYTHON = python3
APP_FILE = main.py

run:
	$(PYTHON) $(APP_FILE)
```

### **O Alvo `.PHONY`**
Por padrão, o `make` verifica se existe um arquivo com o mesmo nome do alvo no diretório. Se existir, ele não executa o comando. Para evitar isso com comandos de automação, usamos o `.PHONY`.

```makefile
.PHONY: run test clean help
```

---

## 🔧 Regras e Dependências

### Dependências Diretas
Um alvo pode depender de arquivos ou outros alvos. Se uma dependência for mais nova que o alvo, o `make` executa a regra.

```makefile
app: main.o util.o
	$(CC) -o app main.o util.o
```

### Alvo padrão
O `make` executa o primeiro alvo definido no Makefile como padrão. Você também pode usar `.DEFAULT_GOAL` para definir explicitamente.

```makefile
.DEFAULT_GOAL := help
```

### Comandos úteis do `make`
- `make`: executa o alvo padrão
- `make help`: mostra ajuda se você tiver um alvo `help`
- `make -n`: mostra os comandos que seriam executados, sem rodar
- `make -B`: força a reconstrução de todos os alvos
- `make -j N`: executa até N comandos em paralelo


### Regras Implícitas e Padrões
O `make` possui regras implícitas que reduzem a repetição. Exemplo de regra padrão:

```makefile
%.o: %.c
	$(CC) -c $< -o $@
```

### Variáveis Automáticas
- `$@` = nome do alvo
- `$<` = primeira dependência
- `$^` = todas as dependências
- `$?` = dependências mais novas que o alvo

```makefile
main.o: main.c defs.h
	$(CC) -c $< -o $@
```

### Dependências de Ordem Somente
Use `|` quando a dependência não deve forçar reconstrução do alvo, apenas garantir ordem.

```makefile
install: | build
	cp bin/app /usr/local/bin
```

### Continuação de Comandos
Use `\` para quebrar comandos longos em várias linhas.

```makefile
lint:
	ruff check . --fix \
		--select E,W
```

---

## 🪄 Variáveis Avançadas

### Tipos de Atribuição
- `=`: expande quando usado
- `:=`: expande imediatamente
- `?=`: atribui apenas se a variável não estiver definida
- `+=`: acrescenta valor

```makefile
PYTHON = python3
PYTHON := python3
PYTHON ?= python3
PATHS += src tests
```

### Variáveis Especiais e Shell
- `MAKEFLAGS`: opções de execução do `make`
- `SHELL`: shell usado para executar comandos
- `.ONESHELL`: executa todas as linhas da receita no mesmo shell

```makefile
SHELL = /bin/bash
.ONESHELL:
```

---

## 🚀 Exemplo Prático para Python
Aqui está um template completo para o seu projeto:

```makefile
# Variáveis
PYTHON = python3
STREAMLIT = streamlit
UV = uv

.PHONY: help install run-st run-uv test lint clean

help: ## Mostra esta ajuda
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-15s\033[0m %s\n", $$1, $$2}'

install: ## Instala dependências via pip
	pip install -r requirements.txt

run-st: ## Roda o Streamlit padrão
	$(STREAMLIT) run main.py

run-uv: ## Roda o Streamlit usando UV
	$(UV) run $(STREAMLIT) run main.py

test: ## Roda os testes com Pytest
	pytest -v

lint: ## Roda o Ruff para checar estilo e erros
	ruff check . --fix

clean: ## Remove arquivos temporários do Python
	find . -type d -name "__pycache__" -exec rm -rf {} +
	find . -type f -name "*.pyc" -delete
```

---

## 🐳 Deploy Docker

Use o Make para criar imagens Docker, rodar containers e limpar artefatos localmente.

```makefile
DOCKER_IMAGE = myapp:latest
DOCKERFILE = Dockerfile

.PHONY: docker-build docker-run docker-clean

docker-build: ## Constrói a imagem Docker
	docker build -t $(DOCKER_IMAGE) -f $(DOCKERFILE) .

docker-run: ## Roda o container Docker localmente
	docker run --rm -p 8501:8501 $(DOCKER_IMAGE)

docker-clean: ## Remove imagens Docker antigas
	docker image rm $(DOCKER_IMAGE) || true
```

### Exemplo de Dockerfile para Streamlit

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 8501
CMD ["streamlit", "run", "main.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

---

## 💡 Dicas e Boas Práticas

1.  **Tabulações**: O erro mais comum no Makefile é usar espaços em vez de TAB nos comandos. Se o `make` reclamar de "missing separator", verifique as tabulações.
2.  **@ no comando**: Usar `@` antes de um comando (ex: `@echo "Iniciando..."`) evita que o próprio comando seja impresso no terminal, mostrando apenas a sua saída.
3.  **Encadeamento**: Você pode criar alvos que chamam outros:
    ```makefile
    deploy: lint test
    	@echo "Enviando para produção..."
    ```
4.  **Uso no Windows**: Para usar Makefile no Windows, você pode instalar o `make` via **Chocolatey** (`choco install make`) ou usar o ambiente **Git Bash**.

---

## ✅ Checklist de Makefile
- [ ] Todos os comandos estão precedidos por um **TAB** (não espaços)?
- [ ] O alvo `.PHONY` foi definido para todos os alvos que não representam arquivos?
- [ ] Variáveis foram usadas para comandos e caminhos repetitivos?
- [ ] Existe um alvo `help` para documentar as tarefas disponíveis?
- [ ] Usou `@` para silenciar a impressão de comandos puramente informativos?
- [ ] As dependências entre alvos foram configuradas corretamente?
- [ ] Limpeza de arquivos temporários (`clean`) foi implementada?
- [ ] O Makefile foi testado em um ambiente limpo?

---

## 🛠️ Como Usar
No terminal, basta digitar `make` seguido do nome do alvo:
```bash
make lint
make test
make run-uv
```
Se digitar apenas `make`, ele executará o primeiro alvo definido no arquivo.
