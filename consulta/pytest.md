# 🧪 Guia Completo: Pytest (Testes Unitários, TDD e Automatizados)

O `pytest` é o framework de testes mais popular para Python, conhecido por sua sintaxe simples, fixtures poderosas e extensibilidade. Este guia foca em TDD, melhores práticas e cobertura completa das funcionalidades essenciais.

---

## 📑 Sumário
1. [🛠️ Instalação e Estrutura](#-instalação-e-estrutura)
2. [📝 Escrita de Testes Básicos](#-escrita-de-testes-básicos)
3. [📦 Convenções e Descoberta de Testes](#-convenções-e-descoberta-de-testes)
4. [🔧 Fixtures (Configuração e Reuso)](#-fixtures-configuração-e-reuso)
5. [🧪 Parametrização](#-parametrização)
6. [🚫 Pulos e XFail](#-pulos-e-xfail)
7. [🧠 Mocking e Monkeypatch](#-mocking-e-monkeypatch)
8. [🗄️ Testes com Banco de Dados](#-testes-com-banco-de-dados)
9. [🌐 Testes de Web Frameworks](#-testes-de-web-frameworks)
10. [⚡ Testes Assíncronos](#-testes-assíncronos)
11. [🧩 Testes de Exceção e Performance](#-testes-de-exceção-e-performance)
12. [📄 Configuração Pytest](#-configuração-pytest)
13. [🚀 Comandos Úteis](#-comandos-úteis)
14. [📚 Plugins Relevantes](#-plugins-relevantes)
15. [⚡ Princípios TDD e Boas Práticas](#-princípios-tdd-e-boas-práticas)

---

## 🛠️ Instalação e Estrutura

```bash
pip install pytest pytest-mock pytest-cov pytest-xdist pytest-asyncio
```

**Estrutura recomendada:**

```text
projeto/
├── src/
│   └── app.py
└── tests/
    ├── conftest.py
    ├── test_app.py
    └── unit/
        ├── test_modelos.py
        └── test_servicos.py
```

- Separe `tests/` em subdiretórios por tipo (unit, integration, e2e).
- `conftest.py` contém fixtures compartilhadas.
- Test files devem começar com `test_` ou terminar com `_test.py`.

---

## 📝 Escrita de Testes Básicos

Teste funções e comportamento com `assert` padrão.

```python
def soma(a, b):
    return a + b


def test_soma_positivos():
    assert soma(2, 3) == 5


def test_soma_negativos():
    assert soma(-1, -1) == -2
```

### Asserções avançadas
- `assert x == y`
- `assert x in y`
- `assert x is None`
- `assert isinstance(obj, Tipo)`
- `assert excinfo.value.args[0] == 'mensagem'`

O pytest fornece mensagens de erro claras sem precisar de `self.assertEqual`.

---

## 📦 Convenções e Descoberta de Testes

### Naming conventions
- arquivos: `test_*.py` ou `*_test.py`
- classes: `Test*`
- funções: `test_*`

### Descoberta automática
O pytest descobre testes recursivamente a partir do diretório atual.

```bash
pytest tests/
```

### Estrutura de teste em classes
```python
class TestCalculadora:
    def test_soma(self):
        assert soma(1, 1) == 2

    def test_subtracao(self):
        assert subtrai(3, 1) == 2
```

- Evite estados compartilhados entre testes da mesma classe.
- Use fixtures para setup/teardown.

---

## 🔧 Fixtures (Configuração e Reuso)

Fixtures são o coração do pytest: fornecem setup reutilizável e controle de ciclo de vida.

### Fixture simples
```python
import pytest

@pytest.fixture
def usuario_exemplo():
    return {'id': 1, 'nome': 'Xvier'}


def test_usuario_nome(usuario_exemplo):
    assert usuario_exemplo['nome'] == 'Xvier'
```

### Escopos de fixtures
- `function` (padrão)
- `class`
- `module`
- `package`
- `session`

```python
@pytest.fixture(scope='module')
def db_engine():
    return criar_engine()
```

### Fixtures autouse
Pegam efeito automaticamente em todos os testes do escopo.

```python
@pytest.fixture(autouse=True)
def limpar_dados():
    resetar_estado_global()
```

### Yield fixtures
Use `yield` para teardown.

```python
@pytest.fixture
def recurso():
    obj = criar_recurso()
    yield obj
    obj.limpar()
```

### Fixture factories e dependências
```python
@pytest.fixture
def usuario_factory():
    def factory(nome):
        return {'id': 1, 'nome': nome}
    return factory


def test_cria_usuario(usuario_factory):
    usuario = usuario_factory('João')
    assert usuario['nome'] == 'João'
```

---

## 🧪 Parametrização

### `@pytest.mark.parametrize`

```python
import pytest

@pytest.mark.parametrize('a, b, esperado', [
    (1, 1, 2),
    (2, 3, 5),
    (10, 20, 30),
])
def test_soma(a, b, esperado):
    assert soma(a, b) == esperado
```

### Parametrização de fixtures
```python
@pytest.fixture(params=[1, 2, 3])
def valor(request):
    return request.param


def test_valores(valor):
    assert valor in {1, 2, 3}
```

---

## 🚫 Pulos e XFail

### `skip`

```python
import pytest

@pytest.mark.skip(reason='Ainda não implementado')
def test_nao_rodar():
    assert False
```

### `skipif`

```python
@pytest.mark.skipif(sys.platform == 'win32', reason='Não compatível com Windows')
def test_linux_apenas():
    assert True
```

### `xfail`

```python
@pytest.mark.xfail(reason='Bug conhecido', strict=False)
def test_bugs():
    assert funcao_bugada() == 42
```

- `strict=True` faz o teste falhar se não ocorrer xfail.

---

## 🧠 Mocking e Monkeypatch

### `pytest-mock`

```python
def test_mocker(mocker):
    mock = mocker.patch('meu_modulo.funcao_externa')
    mock.return_value = 42
    assert meu_modulo.minha_funcao() == 42
```

### `monkeypatch`

```python
def test_monkeypatch(monkeypatch):
    def fake_get(url):
        class Response:
            status_code = 200
            text = 'ok'
        return Response()

    monkeypatch.setattr('httpx.get', fake_get)
    response = httpx.get('https://api.site.com')
    assert response.text == 'ok'
```

### Mocks com `unittest.mock`

```python
from unittest.mock import MagicMock

mock = MagicMock()
mock.atualizar.return_value = None
```

---

## 🗄️ Testes com Banco de Dados

### SQLite em memória

```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from my_app.models import Base, User

@pytest.fixture
def db_session():
    engine = create_engine('sqlite:///:memory:')
    Base.metadata.create_all(engine)
    Session = sessionmaker(bind=engine)
    session = Session()
    yield session
    session.close()
    Base.metadata.drop_all(engine)
```

### Testes de integração
- Separe testes unitários de testes de integração.
- Use fixtures com `scope='module'` ou `scope='session'` com cuidado.
- Use bancos de teste específicos em vez do banco de produção.

---

## 🧩 Testes de Exceção e Performance

### Verificar exceções

```python
import pytest

with pytest.raises(ValueError, match='idade negativa'):
    validar_idade(-1)
```

### Medir desempenho simples

```python
import pytest

@pytest.mark.benchmark
def test_performance():
    resultado = funcao_lenta()
    assert resultado == 1
```

- Para benchmarks avançados, use `pytest-benchmark`.

---

## 📄 Configuração Pytest

### `pytest.ini`

```ini
[pytest]
minversion = 7.0
addopts = -ra -q
testpaths = tests
python_files = test_*.py *_test.py
markers =
    integration: Testes de integração
    slow: Testes lentos
```
```

### `pyproject.toml`

```toml
[tool.pytest.ini_options]
minversion = "7.0"
addopts = "-ra -q"
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
markers = [
  "integration: Testes de integração",
  "slow: Testes lentos",
]
```

### Plugins via `addopts`
- `--cov=src`
- `-n auto`
- `-q`
- `-x`
- `--maxfail=1`

---

## 🚀 Comandos Úteis

- `pytest`: roda todos os testes.
- `pytest -v`: verbose.
- `pytest -s`: mostra `print()`.
- `pytest -k "nome"`: filtra por nome.
- `pytest -m "integration"`: filtra por marcador.
- `pytest --lf`: executa apenas falhas recentes.
- `pytest -x`: para no primeiro erro.
- `pytest --maxfail=3`: interrompe após 3 falhas.
- `pytest --durations=10`: exibe os 10 testes mais lentos.
- `pytest --tb=short|line|native`: controla traceback.
- `pytest --cov=src`: cobertura.
- `pytest -n auto`: execução paralela com `pytest-xdist`.

---

## 📚 Plugins Relevantes

- `pytest-cov`: cobertura de testes.
- `pytest-xdist`: execução paralela.
- `pytest-mock`: fixture `mocker`.
- `pytest-asyncio`: testes assíncronos.
- `pytest-django`: testes Django.
- `pytest-flask`: testes Flask.
- `pytest-benchmark`: benchmarks.

---
## 🌐 Testes de Web Frameworks

### FastAPI
```python
from fastapi.testclient import TestClient
from my_app.main import app

client = TestClient(app)

def test_raiz():
    response = client.get('/')
    assert response.status_code == 200
    assert response.json() == {'mensagem': 'ok'}
```

### Flask
```python
from meu_app import app

with app.test_client() as client:
    response = client.get('/')
    assert response.status_code == 200
```

### Django (pytest-django)
```python
import pytest

@pytest.mark.django_db
def test_pagina_home(client):
    response = client.get('/')
    assert response.status_code == 200
```

- Use `pytest.mark.django_db` para ativar acesso ao banco Django.
- Use `client` e `api_client` em testes REST.

---

## ⚡ Testes Assíncronos

### `pytest-asyncio`

```python
import pytest

@pytest.mark.asyncio
def test_async():
    resultado = await funcao_assincrona()
    assert resultado == 42
```

### `pytest-asyncio` com `httpx.AsyncClient`

```python
import pytest
from httpx import AsyncClient
from fastapi import FastAPI

app = FastAPI()

@app.get('/')
async def root():
    return {'mensagem': 'ok'}

@pytest.mark.asyncio
async def test_fastapi_async():
    async with AsyncClient(app=app, base_url='http://test') as client:
        response = await client.get('/')
        assert response.status_code == 200
        assert response.json() == {'mensagem': 'ok'}
```

### Async fixtures

```python
import pytest

@pytest.fixture
async def recurso_assincrono():
    obj = await criar_recurso_assincrono()
    yield obj
    await obj.fechar()
```

- Use `pytest.mark.asyncio` ou `pytest_asyncio.plugin` para testes async.
- Evite bloqueios em testes assíncronos, use cliente HTTP async e fixtures async.

---
## ⚡ Princípios TDD e Boas Práticas

### Ciclo TDD
1. Red: escrever um teste que falha.
2. Green: implementar o mínimo para passar.
3. Refactor: limpar o código mantendo os testes.

### Dicas de TDD
- escreva testes pequenos e claros.
- mantenha um teste por comportamento.
- evite lógica complexa nos próprios testes.
- use fixtures para setup e teardown.
- nomeie testes usando comportamento esperado.

### Boas práticas
- prefira testes determinísticos e isolados.
- evite dependências externas sempre que possível.
- use `pytest.raises` para exceções.
- use `parametrize` para casos de borda.
- mantenha o ambiente de teste limpo entre casos.
- documente `markers` e convenções em `pytest.ini`.

---

## 📝 Conclusão

Este guia cobre os principais conceitos do `pytest` para testes unitários, TDD, fixtures reutilizáveis, mocking, integração com banco de dados e configuração de projeto.

> Com uma configuração adequada e exemplos de TDD, este documento é muito perto de uma referência bíblica de `pytest` para o desenvolvimento Python.
