# 🖥️ Guia de Referência do Módulo `sys`

O módulo `sys` fornece acesso a variáveis e funções que interagem com o interpretador Python e partes da runtime do sistema.

---

## 📑 Sumário
1. [📥 Argumentos de Linha de Comando](#-argumentos-de-linha-de-comando)
2. [🚪 Finalizando a Execução](#-finalizando-a-execução)
3. [🔍 Informações do Interpretador](#-informações-do-interpretador)
4. [📦 Caminhos de Importação](#-caminhos-de-importação)
5. [⚙️ Configurações de Runtime](#-configurações-de-runtime)
6. [📥 Entrada e Saída Padrão](#-entrada-e-saída-padrão)
7. [🔧 Utilitários de Memória e Codificação](#-utilitários-de-memória-e-codificação)
8. [🧭 `sys.flags` e `sys.audit`](#-sysflags-e-sysaudit)
9. [💡 Quando usar `sys` vs `os`?](#-quando-usar-sys-vs-os)

---

## 📥 Argumentos de Linha de Comando

`sys.argv` é a lista de argumentos passados ao script Python.

```python
import sys

print(sys.argv)
# Exemplo de chamada: python meu_script.py arg1 arg2
# Resultado: ['meu_script.py', 'arg1', 'arg2']

if len(sys.argv) > 1:
    arquivo = sys.argv[1]
    print(f"Arquivo solicitado: {arquivo}")
```

### Dicas
- Use `argparse` ou `click` para parsing mais robusto.
- `sys.argv[0]` pode ser o caminho completo do script.

---

## 🚪 Finalizando a Execução

`sys.exit()` encerra o processo Python imediatamente.

```python
import sys

if error_occurred:
    sys.exit(1)
else:
    sys.exit(0)
```

- `sys.exit()` sem argumento retorna `0`.
- `sys.exit(1)` indica erro.
- `sys.exit('mensagem')` imprime a mensagem em `stderr` e retorna código `1`.

---

## 🔍 Informações do Interpretador

```python
import sys

print(sys.version)
print(sys.version_info)
print(sys.implementation)
print(sys.platform)
print(sys.executable)
print(sys.prefix)
print(sys.base_prefix)
```

### Campos úteis
- `sys.version`: string completa com versão do Python.
- `sys.version_info`: tuple nomeada com major/minor/micro.
- `sys.implementation`: detalhes da implementação Python.
- `sys.platform`: identifica o sistema operacional.
- `sys.executable`: caminho do executável Python em uso.
- `sys.prefix` / `sys.base_prefix`: caminho do ambiente virtual ou instalação base.

---

## 📦 Caminhos de Importação

`sys.path` define onde o Python procura módulos.

```python
import sys

print(sys.path)

# Inserir no início para priorizar um caminho local
sys.path.insert(0, '/meu/caminho/personalizado')

# Remover um caminho que não é mais necessário
sys.path.remove('/meu/caminho/personalizado')
```

### Observações
- Modificar `sys.path` em tempo de execução é útil, mas pode esconder problemas de importação.
- `sys.path[0]` geralmente é o diretório do script em execução.
- Preferível usar pacotes instalados ou `PYTHONPATH` para configurações permanentes.

---

## ⚙️ Configurações de Runtime

### Limite de recursão

```python
import sys

print(sys.getrecursionlimit())
sys.setrecursionlimit(2000)
```

Aumente com cuidado; valores muito altos podem causar `RecursionError` ou esgotar a pilha.

### Endian-ness e tamanho

```python
import sys

print(sys.byteorder)
print(sys.maxsize)
```

- `sys.byteorder`: `little` ou `big`.
- `sys.maxsize`: valor máximo para índices de lista e alguns limites internos.

### `sys.modules`

```python
import sys

print(sys.modules['sys'])
```

- `sys.modules` é o cache dos módulos importados.
- Útil para inspeção, mas não substitui `importlib.reload()`.

---

## 📥 Entrada e Saída Padrão

```python
import sys

sys.stdout.write('Saída padrão\n')
sys.stdout.flush()

sys.stderr.write('Erro padrão\n')
```

### Leitura de stdin

```python
import sys

entrada = sys.stdin.read()
print('Lido:', entrada)
```

### Redirecionamento
- `sys.stdout` e `sys.stderr` podem ser substituídos por objetos com `write()`.
- Útil em testes ou quando você quer capturar a saída do interpretador.

---

## 🔧 Utilitários de Memória e Codificação

```python
import sys

x = [1, 2, 3]
print(sys.getsizeof(x))
print(sys.getdefaultencoding())
print(sys.getfilesystemencoding())
```

### O que observar
- `sys.getsizeof()` mede o tamanho em bytes do objeto e não toda a memória usada por objetos referenciados.
- `sys.getdefaultencoding()` normalmente é `utf-8` em Python 3.
- `sys.getfilesystemencoding()` mostra a codificação usada para nomes de arquivos.

---

## 🧭 `sys.flags` e `sys.audit`

```python
import sys
print(sys.flags)
print(sys.audit)
```

### `sys.flags`
- Indica as opções de inicialização do Python (`debug`, `optimize`, `inspect`, etc.).
- Exemplo: `sys.flags.debug` é `True` se o Python foi iniciado com `-d`.

### `sys.audit`
- API de auditoria do Python (Python 3.8+).
- Permite enviar eventos de auditoria para callbacks do sistema de segurança.

---

## 💡 Quando usar `sys` vs `os`?
- **`sys`**: interage com o interpretador Python.
  - argumentos de linha de comando
  - caminho de importação
  - limites e flags do runtime
  - entrada/saída do interpretador
- **`os`**: interage com o sistema operacional.
  - arquivos e diretórios
  - variáveis de ambiente
  - processos e sinais
  - permissões e caminhos do sistema

---

## 📝 Resumo
- `sys.argv`: argumentos do script.
- `sys.exit()`: encerra o processo.
- `sys.path`: caminhos de importação.
- `sys.version` / `sys.version_info`: informações da versão.
- `sys.getrecursionlimit()` / `sys.setrecursionlimit()`: controle de recursão.
- `sys.stdin`, `sys.stdout`, `sys.stderr`: I/O padrão.
- `sys.getsizeof()`: tamanho do objeto.
- `sys.flags`: opções de inicialização do Python.
