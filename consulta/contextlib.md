# 🛠️ Guia Completo: contextlib (Gerenciadores de Contexto)

O módulo `contextlib` fornece utilitários para trabalhar com a instrução `with`, facilitando a criação e o gerenciamento de gerenciadores de contexto (Context Managers).

---

## 📑 Sumário
1. [🏗️ O que é um Context Manager?](#-o-que-é-um-context-manager)
2. [✨ Decorador @contextmanager](#-decorador-contextmanager)
3. [🛡️ contextlib.suppress](#-contextlibsuppress)
4. [🚪 contextlib.closing](#-contextlibclosing)
5. [📚 contextlib.ExitStack](#-contextlibexitstack)
6. [⚡ asynccontextmanager](#-asynccontextmanager)
7. [🧱 nullcontext](#-nullcontext)
8. [💡 Dicas e Boas Práticas](#-dicas-e-boas-práticas)
9. [✅ Checklist](#-checklist-de-contextlib)

---

## 🏗️ O que é um Context Manager?
É um objeto que define o comportamento ao entrar e sair de um bloco de código usando a palavra-chave `with`. Ele é implementado através dos métodos mágicos `__enter__` e `__exit__`.

```python
class MeuContexto:
    def __enter__(self):
        print("Entrando no contexto...")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Saindo do contexto.")

with MeuContexto():
    print("Executando código...")
```

---

## ✨ Decorador @contextmanager
Permite criar um gerenciador de contexto usando uma função geradora (`yield`), tornando o código muito mais simples.

```python
from contextlib import contextmanager

@contextmanager
def abrir_recurso(nome):
    print(f"Iniciando recurso {nome}")
    try:
        yield f"RECURSO:{nome}" # O valor retornado vai para o 'as' do with
    finally:
        # Tudo após o yield será executado ao sair do with (mesmo com erro)
        print(f"Limpando recurso {nome}")

with abrir_recurso("Banco de Dados") as db:
    print(f"Usando {db}")
```

---

## 🛡️ contextlib.suppress
Ignora silenciosamente exceções específicas. Útil para limpezas onde o erro não importa.

```python
from contextlib import suppress
import os

# Tenta deletar um arquivo, mas não faz nada se ele não existir
with suppress(FileNotFoundError):
    os.remove("arquivo_inexistente.txt")
```

---

## 🚪 contextlib.closing
Garante que um objeto que possui o método `.close()` (mas não é um context manager nativo) seja fechado ao final do bloco.

```python
from contextlib import closing
from urllib.request import urlopen

with closing(urlopen('https://www.google.com')) as page:
    for line in page:
        print(line)
        break # Fecha automaticamente após o primeiro print
```

---

## 📚 contextlib.ExitStack
Permite gerenciar um número dinâmico de context managers em um único bloco. É a solução moderna para aninhamentos profundos.

```python
from contextlib import ExitStack

arquivos = ["a.txt", "b.txt", "c.txt"]

with ExitStack() as stack:
    # Abre múltiplos arquivos dinamicamente
    handlers = [stack.enter_context(open(f, 'w')) for f in arquivos]
    
    for h in handlers:
        h.write("Olá!")
# Todos os arquivos são fechados automaticamente aqui
```

---

## ⚡ asynccontextmanager
O `asynccontextmanager` é a versão assíncrona de `@contextmanager`, usada em `async with`.

```python
from contextlib import asynccontextmanager

@asynccontextmanager
def abrir_recurso_async(nome):
    print(f"Iniciando recurso assíncrono {nome}")
    try:
        yield f"RECURSO_ASYNC:{nome}"
    finally:
        print(f"Limpando recurso assíncrono {nome}")

async def main():
    async with abrir_recurso_async("Serviço") as recurso:
        print(f"Usando {recurso}")
```

---

## 🧱 nullcontext
Use `nullcontext()` quando você precisar de um context manager condicional que não faça nada.

```python
from contextlib import nullcontext

ctx = nullcontext() if condição else abrir_recurso("Banco")
with ctx as recurso:
    print(recurso)
```

---

## 💡 Dicas e Boas Práticas

1.  **Garanta a limpeza**: Sempre use `try...finally` dentro do seu `@contextmanager` para garantir que o código de saída seja executado.
2.  **Async**: Se estiver usando funções assíncronas, use `asynccontextmanager` do mesmo módulo.
3.  **Redirecionamento de I/O**: Use `redirect_stdout` e `redirect_stderr` para capturar ou silenciar saídas de terminal temporariamente.
    ```python
    from contextlib import redirect_stdout
    import io

    f = io.StringIO()
    with redirect_stdout(f):
        print("Isso não vai para o terminal")
    ```
4.  **NullContext**: Use `nullcontext()` quando precisar de um context manager opcional que não faz nada.

---

## ✅ Checklist de contextlib
- [ ] Implementou `__enter__` e `__exit__` para classes personalizadas?
- [ ] Usou `@contextmanager` para simplificar geradores simples?
- [ ] Garantiu a limpeza com `try...finally` dentro de geradores?
- [ ] Usou `ExitStack` para gerenciar múltiplos contextos dinâmicos?
- [ ] Usou `suppress` para ignorar erros esperados e não críticos?
- [ ] Utilizou `closing` para objetos que possuem apenas o método `.close()`?
- [ ] Validou o uso de `asynccontextmanager` em ambientes assíncronos?
