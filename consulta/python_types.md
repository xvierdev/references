# 🐍 Guia de Tipos e Coleções em Python

Este guia explica os tipos embutidos mais comuns, as coleções especiais da biblioteca `collections` e as principais anotações de tipo do Python moderno.

---

## 📑 Sumário
1. [📦 Tipos Embutidos (Built-in)](#-tipos-embutidos-built-in)
2. [🧠 Tipos Numéricos](#-tipos-numéricos)
3. [📝 Tipos de Texto e Binários](#-tipos-de-texto-e-binários)
4. [📚 Coleções Padrão](#-coleções-padrão)
5. [📚 Biblioteca `collections`](#-biblioteca-collections)
6. [📌 Tipos do Módulo `typing`](#-tipos-do-módulo-typing)
7. [🧩 Dicas de Uso](#-dicas-de-uso)

---

## 📦 Tipos Embutidos (Built-in)

### **bool**
Tipo booleano com apenas dois valores: `True` ou `False`.
```python
esta_ativo = True
if esta_ativo:
    print("Ligado")
```

### **int**
Inteiros de precisão arbitrária. Usa-se para contagens, índices e valores sem casas decimais.
```python
idade = 30
valor = 10_000
```

### **float**
Números de ponto flutuante.
```python
temperatura = 36.6
```

### **complex**
Números complexos com parte real e imaginária.
```python
z = 3 + 4j
print(z.real, z.imag) # 3.0 4.0
```

### **str**
Sequência de caracteres Unicode.
```python
nome = "Alice"
print(nome.upper())
```

### **bytes**
Dados imutáveis em formato binário.
```python
b = b"hello"
print(b[0]) # 104
```

### **bytearray**
Semelhante a `bytes`, mas mutável.
```python
b = bytearray(b"hello")
b[0] = 72
print(b) # bytearray(b'Hello')
```

### **memoryview**
Visão de memória sobre um objeto binário, sem cópia.
```python
b = bytearray(b"hello")
v = memoryview(b)
v[0] = 72
print(bytes(v)) # b'Hello'
```

### **range**
Sequência de inteiros que é gerada sob demanda.
```python
for i in range(0, 10, 2):
    print(i)
```

### **list**
Coleção ordenada e mutável. Permite duplicatas.
```python
frutas = ["maçã", "banana", "laranja"]
frutas.append("uva")
```

### **tuple**
Coleção ordenada e imutável.
```python
ponto = (10, 20)
```

### **dict**
Coleção de pares chave-valor. Chaves devem ser únicas.
```python
pessoa = {"nome": "Alice", "idade": 25}
```

### **set**
Coleção não ordenada de elementos únicos.
```python
numeros = {1, 2, 2, 3}
```

### **frozenset**
Versão imutável de `set`.
```python
conjunto = frozenset([1, 2, 3])
```

### **NoneType**
Representa a ausência de valor.
```python
valor = None
```

### **object**
Tipo base de todo objeto em Python.
```python
x = object()
```

---

## 🧠 Tipos Numéricos

### `int`
- Sem limite de tamanho.
- Suporta operações de bitwise.

### `float`
- Baseado em IEEE 754.
- Use `math.isclose()` para comparações de ponto flutuante.

### `complex`
- Útil em cálculos científicos e sinais.
- A parte real e imaginária são acessíveis por `real` e `imag`.

### Operações comuns
```python
x = 5
y = 2.5
print(x + y, x * y, x ** 2)
```

---

## 📝 Tipos de Texto e Binários

### `str`
- Strings Unicode.
- Métodos úteis: `split()`, `join()`, `format()`, `replace()`.

### `bytes`
- Imutável.
- Ideal para I/O binário, redes e criptografia.

### `bytearray`
- Mutável e eficiente para manipulação de bytes.

### `memoryview`
- Permite criar fatias sem cópia em objetos binários.

---

## 📚 Coleções Padrão

### `list`
- Use quando precisar modificar a sequência.
- Iteração e indexação são O(1).

### `tuple`
- Use para dados imutáveis ou retornos múltiplos.
- Ótimo como chave de `dict` quando imutável.

### `dict`
- Busca por chave em tempo médio O(1).
- Boa escolha para mapeamentos e JSON-like.

### `set` e `frozenset`
- `set` para operações mutáveis.
- `frozenset` para valores imutáveis, úteis em chaves ou cache.

### `range`
- Eficiente para listas de inteiros sequenciais.
- Não consumindo memória para sequências grandes.

---

## 📚 Biblioteca `collections`

### **defaultdict**
Fornece valores padrão para chaves inexistentes.
```python
from collections import defaultdict

grupos = defaultdict(int)
grupos["a"] += 1
```

### **OrderedDict**
Mantém a ordem de inserção e adiciona utilitários extras.
```python
from collections import OrderedDict

d = OrderedDict([('a', 1), ('b', 2)])
d.move_to_end('a')
```

### **Counter**
Conta elementos de iteráveis.
```python
from collections import Counter

c = Counter("abracadabra")
print(c.most_common(2))
```

### **deque**
Fila de duas pontas de alta performance.
```python
from collections import deque

d = deque([1, 2, 3])
d.appendleft(0)
d.rotate(1)
```

### **namedtuple**
Tupla com campos nomeados.
```python
from collections import namedtuple
Carro = namedtuple('Carro', ['marca', 'modelo'])
```

### **ChainMap**
Combina vários mapeamentos em uma única visualização.
```python
from collections import ChainMap

a = {'x': 1}
b = {'y': 2}
cm = ChainMap(a, b)
```

### **UserDict**, **UserList**, **UserString**
Classes base para criar subclasses de coleções com comportamento personalizado.
```python
from collections import UserDict

class MeuDict(UserDict):
    pass
```

---

## 📌 Tipos do Módulo `typing`

### Anotações básicas
```python
from typing import List, Dict, Optional, Tuple

valores: List[int] = [1, 2, 3]
config: Dict[str, str] = {"host": "localhost"}
usuario: Optional[str] = None
```

### Tipos compostos
```python
dados: Tuple[str, int] = ("Alice", 30)
```

### `Union` e `Optional`
```python
from typing import Union

valor: Union[int, str] = 10
valor = "dez"
```

### `Any`
```python
from typing import Any

dado: Any = "qualquer coisa"
```

### `TypedDict`
```python
from typing import TypedDict

class UserDict(TypedDict):
    name: str
    age: int
```

### `NamedTuple`
```python
from typing import NamedTuple

class Cliente(NamedTuple):
    nome: str
    idade: int
```

### `Protocol` e `TypeVar`
```python
from typing import Protocol, TypeVar

T = TypeVar('T')

class SuportaSalvar(Protocol):
    def save(self) -> None:
        ...
```

---

## 🧩 Dicas de Uso
- Use **list** para coleções mutáveis.
- Use **tuple** para dados imutáveis e chaves de dicionário.
- Use **set** para exclusão de duplicatas e operações de conjunto.
- Use **frozenset** quando precisar de imutabilidade em conjuntos.
- Use **defaultdict** para agrupar dados sem checar chaves.
- Use **deque** para filas com desempenho constante em ambas as pontas.
- Prefira `dict` nativo em Python 3.7+ em vez de `OrderedDict`, exceto quando precisar de métodos extras.
- Use `typing` para documentar contratos de função e melhorar IDEs.
- Evite `==` em floats quando precisão for crítica; use `math.isclose()`.

---

## 📚 Referências Rápidas
- Tipos built-in: `bool`, `int`, `float`, `complex`, `str`, `bytes`, `bytearray`, `memoryview`, `range`
- Coleções: `list`, `tuple`, `dict`, `set`, `frozenset`
- `collections`: `defaultdict`, `Counter`, `deque`, `OrderedDict`, `namedtuple`, `ChainMap`
- `typing`: `List`, `Dict`, `Tuple`, `Optional`, `Union`, `Any`, `TypedDict`, `NamedTuple`, `Protocol`, `TypeVar`
