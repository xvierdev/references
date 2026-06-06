# 🐍 Dicas e Padrões Avançados em Python

Este guia cobre classes, herança, polimorfismo, interfaces e os padrões mais importantes para POO em Python.

---

## 📑 Sumário
1. [🏗️ Classes e Objetos](#-classes-e-objetos)
2. [⚙️ Construtores e Inicialização](#-construtores-e-inicialização)
3. [🧬 Herança e Polimorfismo](#-herança-e-polimorfismo)
4. [🧩 Composição e Mixins](#-composição-e-mixins)
5. [📌 Abstração, ABCs e Protocols](#-abstração-abcs-e-protocols)
6. [🗂️ `@dataclass` e Modelos de Dados](#-dataclass-e-modelos-de-dados)
7. [🔢 Enums e Constantes](#-enums-e-constantes)
8. [✨ Decoradores e Métodos de Classe](#-decoradores-e-métodos-de-classe)
9. [📊 JSON, Dicionários e Serialização](#-json-dicionários-e-serialização)
10. [🏷️ Tipagem Avançada](#-tipagem-avançada)
11. [🧠 Design Patterns](#-design-patterns)
12. [🧪 Metaclasses e `__slots__`](#-metaclasses-e-slots)
13. [📦 Pacotes, Módulos e Importações](#-pacotes-módulos-e-importações)
14. [⚡ Boas Práticas de POO](#-boas-práticas-de-poo)

---

## 🏗️ Classes e Objetos

### Definição básica
```python
class Pessoa:
    def __init__(self, nome: str, idade: int):
        self.nome = nome
        self.idade = idade

    def saudacao(self) -> str:
        return f"Olá, eu sou {self.nome}."
```

### Atributos de instância vs classe
```python
class Carro:
    rodas = 4  # atributo de classe

    def __init__(self, modelo: str):
        self.modelo = modelo  # atributo de instância
```

### Métodos especiais importantes
- `__repr__`: representação oficial para debug.
- `__str__`: representação amigável.
- `__eq__`: igualdade de objetos.
- `__hash__`: permite uso em `set` e `dict`.
- `__len__`, `__iter__`, `__getitem__`: make objects behave like coleções.

```python
class Pessoa:
    def __init__(self, nome: str, idade: int):
        self.nome = nome
        self.idade = idade

    def __repr__(self):
        return f"Pessoa(nome={self.nome!r}, idade={self.idade})"

    def __str__(self):
        return f"{self.nome}, {self.idade} anos"

    def __eq__(self, outro):
        return isinstance(outro, Pessoa) and self.nome == outro.nome and self.idade == outro.idade
```

---

## ⚙️ Construtores e Inicialização

### `__init__` e `__new__`
- `__init__`: inicializa instâncias já criadas.
- `__new__`: cria a instância antes de inicializar.

```python
class MinhaClasse:
    def __new__(cls, *args, **kwargs):
        instancia = super().__new__(cls)
        return instancia

    def __init__(self, valor):
        self.valor = valor
```

### Construtores alternativos com `@classmethod`
```python
from datetime import datetime

class Evento:
    def __init__(self, data: datetime):
        self.data = data

    @classmethod
    def from_iso(cls, texto: str):
        data = datetime.fromisoformat(texto)
        return cls(data)
```

### `@property` e encapsulamento
```python
class Retangulo:
    def __init__(self, largura: float, altura: float):
        self._largura = largura
        self._altura = altura

    @property
    def area(self) -> float:
        return self._largura * self._altura

    @property
    def largura(self) -> float:
        return self._largura

    @largura.setter
    def largura(self, valor: float):
        if valor <= 0:
            raise ValueError('Largura deve ser positiva')
        self._largura = valor
```

---

## 🧬 Herança e Polimorfismo

### Herança simples
```python
class Animal:
    def falar(self):
        return '...'

class Cachorro(Animal):
    def falar(self):
        return 'Au au'
```

### `super()` e método sobrescrito
```python
class Pessoa:
    def __init__(self, nome: str):
        self.nome = nome

class Funcionario(Pessoa):
    def __init__(self, nome: str, cargo: str):
        super().__init__(nome)
        self.cargo = cargo
```

### Polimorfismo via duck typing
```python
class Gato:
    def falar(self):
        return 'Miau'

class Cachorro:
    def falar(self):
        return 'Au au'

for animal in [Gato(), Cachorro()]:
    print(animal.falar())
```

### MRO e herança múltipla
```python
class A:
    pass
class B(A):
    pass
class C(A):
    pass
class D(B, C):
    pass

print(D.mro())
```

- `super()` segue a MRO.
- evite herança múltipla complexa; prefira composição.

---

## 🧩 Composição e Mixins

### Composição
```python
class Motor:
    def ligar(self):
        return 'Motor ligado'

class Carro:
    def __init__(self, motor: Motor):
        self.motor = motor

    def ligar(self):
        return self.motor.ligar()
```

### Mixins
Mixin é uma classe pequena que adiciona um comportamento específico.

```python
class JsonSerializableMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__)

class Produto(JsonSerializableMixin):
    def __init__(self, nome, preco):
        self.nome = nome
        self.preco = preco
```

- Use mixins quando quiser reutilizar comportamentos sem expressar relacionamento de domínio.

---

## 📌 Abstração, ABCs e Protocols

### ABC (Abstract Base Classes)
```python
from abc import ABC, abstractmethod

class Repositorio(ABC):
    @abstractmethod
    def salvar(self, obj):
        pass

class RepositorioMemoria(Repositorio):
    def salvar(self, obj):
        print('salvo')
```

### `Protocol` e tipagem estrutural
```python
from typing import Protocol

class Documento(Protocol):
    def ler(self) -> str:
        ...

class PDF:
    def ler(self) -> str:
        return 'texto'


def processar(doc: Documento):
    print(doc.ler())
```

### Interfaces e contratos
- Python não tem interfaces formais como Java.
- `ABC` e `Protocol` servem como contratos.
- Prefira duck typing quando possível.

---

## 🗂️ `@dataclass` e Modelos de Dados

### Uso básico
```python
from dataclasses import dataclass

@dataclass
class Pessoa:
    nome: str
    idade: int
```

### Recursos úteis
- `frozen=True` torna imutável.
- `repr=False` oculta em `__repr__`.
- `default_factory` para listas e dicionários.

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)
class Endereco:
    rua: str
    numero: int
    tags: list[str] = field(default_factory=list)
```

### `__post_init__`
```python
@dataclass
class Usuario:
    nome: str
    idade: int

    def __post_init__(self):
        if self.idade < 0:
            raise ValueError('idade negativa')
```

---

## 🔢 Enums e Constantes

### Enum básico
```python
from enum import Enum, auto

class Status(Enum):
    PENDENTE = auto()
    PROCESSANDO = auto()
    CONCLUIDO = auto()
```

### `IntEnum` e `Flag`
```python
from enum import IntEnum, Flag, auto

class Prioridade(IntEnum):
    BAIXA = 1
    MEDIA = 2
    ALTA = 3

class Permissao(Flag):
    READ = auto()
    WRITE = auto()
    EXECUTE = auto()
```

### Uso recomendado
- Enums são melhores que constantes soltas.
- Use `Enum` para estados fixos.
- Use `auto()` quando valores explícitos não importam.

---

## ✨ Decoradores e Métodos de Classe

### `@staticmethod` vs `@classmethod`
```python
class Pessoa:
    especie = 'Humano'

    @staticmethod
    def saudacao():
        return 'Olá'

    @classmethod
    def criar_com_idade_padrao(cls, nome):
        return cls(nome, 18)
```

### Decoradores com argumentos
```python
from functools import wraps

def registrar(rol):
    def decorator(func):
        @wraps(func)
        def envelope(*args, **kwargs):
            print(f'Registrando em {rol}')
            return func(*args, **kwargs)
        return envelope
    return decorator
```

### `@property`
- transforma método em atributo.
- ajuda encapsulamento sem mudar API.

---

## 📊 JSON, Dicionários e Serialização

### Serializando dataclasses
```python
from dataclasses import asdict, dataclass
import json

@dataclass
class Produto:
    nome: str
    preco: float

produto = Produto('Caneta', 1.50)
print(json.dumps(asdict(produto)))
```

### Dicionários aninhados e conversão
```python
data = {'nome': 'Ana', 'idades': [20, 30]}
json_str = json.dumps(data)
data2 = json.loads(json_str)
```

---

## 🏷️ Tipagem Avançada

### Anotações comuns
- `int`, `str`, `list[str]`, `dict[str, int]`
- `Optional[str]`
- `Union[int, str]`
- `Any`

### Generics e TypeVar
```python
from typing import TypeVar, Generic

T = TypeVar('T')

class Caixa(Generic[T]):
    def __init__(self, valor: T):
        self.valor = valor
```

### `Annotated`
```python
from typing import Annotated

ID = Annotated[int, 'ID do usuário']
```

### `Protocol`
- como interface estrutural.
- ótimo para funções que aceitam objetos compatíveis.

---

## 🧠 Design Patterns

### Strategy
Permite trocar o algoritmo em tempo de execução.

```python
from abc import ABC, abstractmethod

class EstrategiaPagamento(ABC):
    @abstractmethod
    def pagar(self, valor: float) -> str:
        pass

class PagamentoCartao(EstrategiaPagamento):
    def pagar(self, valor: float) -> str:
        return f"Pago R${valor} com cartão"

class PagamentoPix(EstrategiaPagamento):
    def pagar(self, valor: float) -> str:
        return f"Pago R${valor} via PIX"

class Pedido:
    def __init__(self, estrategia: EstrategiaPagamento):
        self.estrategia = estrategia

    def finalizar(self, valor: float) -> str:
        return self.estrategia.pagar(valor)

pedido = Pedido(PagamentoPix())
print(pedido.finalizar(100.0))
```

### Factory
Cria objetos sem expor a lógica de instância diretamente.

```python
from enum import Enum

class TipoNotificacao(Enum):
    EMAIL = 'email'
    SMS = 'sms'

class Notificacao:
    def enviar(self, mensagem: str) -> str:
        raise NotImplementedError

class NotificacaoEmail(Notificacao):
    def enviar(self, mensagem: str) -> str:
        return f"Enviando e-mail: {mensagem}"

class NotificacaoSMS(Notificacao):
    def enviar(self, mensagem: str) -> str:
        return f"Enviando SMS: {mensagem}"

class NotificacaoFactory:
    @staticmethod
    def criar(tipo: TipoNotificacao) -> Notificacao:
        if tipo == TipoNotificacao.EMAIL:
            return NotificacaoEmail()
        if tipo == TipoNotificacao.SMS:
            return NotificacaoSMS()
        raise ValueError('Tipo de notificação inválido')

notificacao = NotificacaoFactory.criar(TipoNotificacao.EMAIL)
print(notificacao.enviar('Olá'))
```

### Singleton
Garante que haja apenas uma instância de uma classe.

```python
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Config(metaclass=SingletonMeta):
    def __init__(self, valor: str):
        self.valor = valor

c1 = Config('A')
c2 = Config('B')
print(c1 is c2)
print(c2.valor)
```

### Observações sobre patterns
- Patterns são guias, não regras rígidas.
- Use o mais simples que resolve o problema.
- Preferir composição, testes e clareza sobre complexidade desnecessária.

---

## 🧪 Metaclasses e `__slots__`

### `__slots__`
Reduz uso de memória e bloqueia atributos dinâmicos.

```python
class Pessoa:
    __slots__ = ('nome', 'idade')

    def __init__(self, nome: str, idade: int):
        self.nome = nome
        self.idade = idade
```

- sem `__slots__`, instâncias usam `__dict__`.
- com `__slots__`, apenas atributos listados são permitidos.

### Metaclasses
Classes que criam classes.

```python
class UpperAttrMeta(type):
    def __new__(mcs, name, bases, namespace):
        uppercase_attrs = {
            key.upper(): value
            for key, value in namespace.items()
            if not key.startswith('__')
        }
        return super().__new__(mcs, name, bases, uppercase_attrs)

class MinhaClasse(metaclass=UpperAttrMeta):
    meu_attr = 'valor'

print(hasattr(MinhaClasse, 'MEU_ATTR'))
```

### Uso comum de metaclasses
- registrar classes automaticamente.
- validar ou modificar atributos em tempo de definição.
- implementar padrões de criação como Singleton.

---

## 📦 Pacotes, Módulos e Importações

### Estrutura de pacote
```
meu_pacote/
    __init__.py
    modulo_a.py
    subpacote/
        __init__.py
        modulo_b.py
```

### `__name__ == '__main__'`
```python
if __name__ == '__main__':
    main()
```

### Importações relativas
```python
from .modulo_a import func_a
from ..outro_pacote import func_b
```

---

## ⚡ Boas Práticas de POO

- Prefira composição em vez de herança quando fizer mais sentido.
- Use herança para representar uma relação “é um”.
- Mantenha `__init__` simples e delegue lógica complexa a métodos.
- Utilize `@property` para encapsular acessos.
- Evite classes com muitas responsabilidades (princípio SRP).
- Use `dataclass` para estruturas de dados simples.
- Documente as abstrações e contrate invariantes no código.
- Evite múltipla herança complexa sem um motivo claro.
- Use `ABC` ou `Protocol` apenas quando quiser definir contratos explícitos.

---

## 📝 Resumo rápido

- `class` define tipos compostos.
- `__init__` inicializa a instância.
- `super()` chama o pai em hierarquias.
- Herança + override = polimorfismo.
- `@dataclass` gera boilerplate automaticamente.
- `Enum` traz valores nomeados e seguros.
- `Protocol` é interface estrutural.
- Composição é geralmente mais segura que herança complexa.
