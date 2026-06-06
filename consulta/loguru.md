# 🪵 Guia Supremo: Loguru (Logging Moderno em Python)

O `loguru` é uma biblioteca que torna o logging em Python extremamente simples, poderoso e bonito. Ele elimina a complexidade do módulo padrão `logging`, oferecendo uma interface única e pré-configurada para quase todas as necessidades.

---

## 📑 Sumário
1. [🛠️ Instalação](#-instalação)
2. [🚀 Uso Rápido (Quick Start)](#-uso-rápido-quick-start)
3. [🎯 Sinks: Destinos dos Logs](#-sinks-destinos-dos-logs)
4. [🎨 Formatação e Cores](#-formatação-e-cores)
5. [📂 Rotação, Retenção e Compressão](#-rotação-retenção-e-compressão)
6. [🚨 Captura de Exceções (Traceback Rico)](#-captura-de-exceções-traceback-rico)
7. [🔍 Filtros e Níveis Customizados](#-filtros-e-níveis-customizados)
8. [🧵 Thread-Safety e Async](#-thread-safety-e-async)
9. [🔌 Integração com Logging Padrão](#-integração-com-logging-padrão)
10. [🧠 Boas Práticas e Performance](#-boas-práticas-e-performance)
11. [✅ Checklist Loguru](#-checklist-loguru)

---

## 🛠️ Instalação

```bash
pip install loguru
```

---

## 🚀 Uso Rápido (Quick Start)

Diferente do `logging` padrão, você não precisa de configurações complexas de *handlers*. Basta importar e usar.

```python
from loguru import logger

logger.debug("Isso é para desenvolvedores (depuração)")
logger.info("Informação geral do sistema")
logger.success("Operação concluída com sucesso!")
logger.warning("Algo estranho aconteceu")
logger.error("Ocorreu um erro!")
logger.critical("Falha catastrófica!")
```

---

## 🎯 Sinks: Destinos dos Logs

No `loguru`, tudo é um **Sink**. Um sink pode ser o console (`sys.stderr`), um arquivo, uma função ou até uma URL.

### Enviando para um arquivo
```python
logger.add("app.log")
```

### Removendo o log padrão do console
Se você quiser que os logs fiquem apenas no arquivo:
```python
import sys
logger.remove() # Remove o handler padrão (stderr)
logger.add(sys.stderr, level="INFO") # Adiciona de volta apenas para INFO ou superior
logger.add("file.log", level="DEBUG") # Tudo vai para o arquivo
```

---

## 🎨 Formatação e Cores

Você pode personalizar totalmente como o log aparece.

```python
logger.add(
    sys.stderr, 
    format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | <level>{level: <8}</level> | <cyan>{name}</cyan>:<cyan>{function}</cyan>:<cyan>{line}</cyan> - <level>{message}</level>",
    colorize=True
)
```

**Tags de Cores Comuns:**
- `<green>`, `<red>`, `<blue>`, `<yellow>`, `<cyan>`, `<magenta>`
- `<level>`: Muda a cor dependendo do nível do log (Erro=Vermelho, Sucesso=Verde).

---

## 📂 Rotação, Retenção e Compressão

Gerenciar arquivos de log nunca foi tão fácil.

```python
logger.add(
    "file.log",
    rotation="500 MB",    # Cria novo arquivo ao chegar em 500MB
    # rotation="12:00",   # Rotaciona todo dia ao meio-dia
    # rotation="1 week",  # Rotaciona semanalmente
    retention="10 days",  # Mantém apenas logs dos últimos 10 dias
    compression="zip",    # Compacta logs antigos para economizar espaço
    encoding="utf-8"
)
```

---

## 🚨 Captura de Exceções (Traceback Rico)

O `loguru` pode capturar exceções automaticamente com variáveis locais e mensagens detalhadas.

### Usando o Decorador `@logger.catch`
```python
@logger.catch
def minha_funcao_perigosa(x, y):
    return x / y

minha_funcao_perigosa(10, 0) # Gera um log de erro lindamente formatado com variáveis
```

### Usando em blocos Try/Except
```python
try:
    processar_dados()
except Exception:
    logger.exception("Algo deu muito errado no processamento:")
```

---

## 🔍 Filtros e Níveis Customizados

### Filtragem
Você pode enviar logs de módulos específicos para arquivos específicos.

```python
logger.add("database.log", filter="meu_projeto.db")
```

### Níveis Customizados
```python
logger.level("AUDITORIA", no=38, color="<magenta>")
logger.log("AUDITORIA", "Acesso ao banco de dados pelo usuário X")
```

---

## 🧵 Thread-Safety e Async

O `loguru` é thread-safe por padrão. Para uso com `asyncio`, use o parâmetro `enqueue=True` para garantir que as escritas não bloqueiem o loop de eventos.

```python
logger.add("async.log", enqueue=True)
```

---

## 🔌 Integração com Logging Padrão

Se você usa bibliotecas que dependem do `logging` padrão (como FastAPI ou Django), você pode redirecionar os logs delas para o Loguru.

```python
import logging

class InterceptHandler(logging.Handler):
    def emit(self, record):
        try:
            level = logger.level(record.levelname).name
        except ValueError:
            level = record.levelno
        logger.opt(depth=6, exception=record.exc_info).log(level, record.getMessage())

logging.basicConfig(handlers=[InterceptHandler()], level=0)
```

---

## 🧠 Boas Práticas e Performance

1. **Use `serialize=True`** se for enviar logs para o **ELK Stack** ou **Loki** (gera formato JSON).
2. **Evite `logger.remove()` em bibliotecas**: Deixe que o usuário final da sua lib decida para onde os logs vão.
3. **Use `diagnose=False` e `backtrace=False` em Produção**: Tracebacks completos com variáveis locais podem expor segredos sensíveis e consumir mais memória.
4. **Use `enqueue=True` em sistemas de alta carga** para não gargalar a execução principal com I/O de disco.

---

## ✅ Checklist Loguru

- [ ] Instalou via `pip install loguru`.
- [ ] Configurou `logger.add` para arquivos com `rotation` e `retention`.
- [ ] Personalizou o `format` para facilitar a leitura.
- [ ] Usou `@logger.catch` em funções críticas/principais.
- [ ] Configurou `enqueue=True` se o app for assíncrono.
- [ ] Verificou se segredos não estão sendo logados via `diagnose`.
- [ ] Integração com `logging` padrão configurada (se necessário).
- [ ] Logs de erro configurados com `compression="zip"`.

---

## 💡 Notas Finais

O `loguru` é a escolha definitiva para quem quer parar de lutar com configurações de logging e focar no que importa: a aplicação. Ele é elegante, intuitivo e resolve 99% dos problemas de rastreabilidade com poucas linhas de código.
