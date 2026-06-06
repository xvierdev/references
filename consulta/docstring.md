# 📝 Guia Profissional de Docstrings em Python

Docstrings são strings literais que aparecem logo após a definição de um módulo, função, classe ou método. Elas são essenciais para documentação automática, navegação de IDE e para que outros desenvolvedores entendam o código sem precisar entrar nos detalhes da implementação.

---

## 📑 Sumário
1. [📜 Padrão PEP 257](#-padrão-pep-257)
2. [🏢 Estilo Google (Recomendado)](#-estilo-google-recomendado)
3. [📊 Estilo NumPy/SciPy](#-estilo-numpy-scipy)
4. [📘 Estilo Sphinx / reStructuredText](#-estilo-sphinx--restructuredtext)
5. [🧾 Tipos de docstrings](#-tipos-de-docstrings)
6. [💡 Melhores Práticas](#-melhores-práticas)
7. [🛠️ Ferramentas e automação](#-ferramentas-e-automações)

---

## 📜 Padrão PEP 257
A PEP 257 define as convenções básicas para docstrings.
- Use aspas duplas triplas: `"""Docstring aqui"""`.
- A primeira linha deve ser um resumo curto, terminando com ponto final.
- Se houver mais detalhes, deixe uma linha em branco após o resumo.
- Uma docstring de uma linha só é aceitável quando a documentação é sucinta.
- O corpo do texto deve ter linhas com no máximo 72 caracteres, quando possível.

### Exemplo de função
```python
def somar(a, b):
    """Realiza a soma de dois números.

    Esta função recebe dois parâmetros numéricos e retorna a sua soma total.
    """
    return a + b
```

### Exemplo de classe
```python
class Pessoa:
    """Representa uma pessoa com nome e idade."""

    def __init__(self, nome, idade):
        self.nome = nome
        self.idade = idade
```

### Exemplo de módulo
```python
"""Funções utilitárias para processamento de arquivos e logs."""

import os

# código do módulo...
```

---

## 🏢 Estilo Google (Recomendado)
O estilo Google é muito legível e funciona bem em APIs, bibliotecas e projetos de backend. Ele é especialmente útil quando combinado com type hints.

### Exemplo de função
```python
def processar_dados(nome: str, idade: int, ativo: bool = True) -> dict:
    """Processa as informações de um usuário para o sistema.

    Args:
        nome (str): O nome completo do usuário.
        idade (int): A idade do usuário (deve ser positiva).
        ativo (bool, optional): O status do usuário no sistema. Defaults to True.

    Returns:
        dict: Um dicionário contendo o resumo processado.

    Raises:
        ValueError: Se a idade for menor que zero.
    """
    if idade < 0:
        raise ValueError("A idade não pode ser negativa.")
    return {"user": nome, "status": "ok"}
```

### Exemplo de método de classe
```python
class ContaBancaria:
    """Modelo básico de conta bancária."""

    def __init__(self, titular: str, saldo: float = 0.0):
        """Inicializa a conta com titular e saldo inicial."""
        self.titular = titular
        self.saldo = saldo

    def depositar(self, valor: float) -> None:
        """Adiciona valor ao saldo da conta."""

        Args:
            valor (float): Valor a ser depositado.
        """
        self.saldo += valor
```

### Exemplo com @property
```python
class Retangulo:
    """Representa um retângulo com largura e altura."""

    def __init__(self, largura: float, altura: float):
        self.largura = largura
        self.altura = altura

    @property
    def area(self) -> float:
        """Retorna a área do retângulo."""
        return self.largura * self.altura
```

### Exemplo assíncrono
```python
async def buscar_usuarios(api_url: str) -> list:
    """Busca a lista de usuários de uma API externa.

    Args:
        api_url (str): URL da API.

    Returns:
        list: Lista de usuários recebida da API.
    """
    ...
```

### Exemplo com decorador
```python
def exige_autenticacao(func):
    """Decorador que verifica autenticação antes de executar a função."""

    def wrapper(*args, **kwargs):
        """Wrapper que aplica a lógica de autenticação."""
        return func(*args, **kwargs)

    return wrapper
```

---

## 📊 Estilo NumPy / SciPy
Esse estilo é muito utilizado em projetos científicos e de análise de dados. Ele é mais detalhado e tem seções claras de parâmetros, retornos e notas.

### Exemplo de classe com NumPy
```python
class Calculadora:
    """
    Uma classe simples para realizar operações matemáticas.

    Attributes
    ----------
    precisao : int
        O número de casas decimais para os resultados.
    """

    def __init__(self, precisao=2):
        self.precisao = precisao

    def dividir(self, a, b):
        """
        Divide dois números.

        Parameters
        ----------
        a : float
            O dividendo.
        b : float
            O divisor.

        Returns
        -------
        float
            O resultado da divisão formatado pela precisão.

        Raises
        ------
        ZeroDivisionError
            Se b for zero.
        """
        if b == 0:
            raise ZeroDivisionError("Divisão por zero não permitida.")
        return round(a / b, self.precisao)
```

### Exemplo de função com NumPy
```python
def normalizar(valores):
    """
    Normaliza uma sequência de valores.

    Parameters
    ----------
    valores : array-like
        Valores a serem normalizados.

    Returns
    -------
    ndarray
        Valores normalizados entre 0 e 1.
    """
    ...
```

---

## 📘 Estilo Sphinx / reStructuredText
O estilo Sphinx é ideal quando você publica documentação com Sphinx, Read the Docs ou ferramentas que interpretam reStructuredText.

### Exemplo de função
```python
def converter_temperatura(celsius: float) -> float:
    """Converte Celsius para Fahrenheit.

    :param celsius: Temperatura em Celsius.
    :type celsius: float
    :return: Temperatura em Fahrenheit.
    :rtype: float
    :raises ValueError: Se a temperatura for menor que -273.15.
    """
    if celsius < -273.15:
        raise ValueError("Temperatura abaixo do zero absoluto.")
    return celsius * 9 / 5 + 32
```

### Exemplo de classe
```python
class Servico:
    """Classe de serviço de exemplo.

    :param nome: Nome do serviço.
    :type nome: str
    """

    def __init__(self, nome: str):
        self.nome = nome
```

---

## 🧾 Tipos de docstrings
- Módulo: documenta o propósito do módulo, dependências externas e uso principal.
- Função / método: explica o objetivo, parâmetros, retorno e exceções.
- Classe: descreve responsabilidade, atributos públicos e comportamento importante.
- Propriedade: descreve o valor retornado e sua semântica.
- Decoradores: explique o propósito do decorador e o contrato que ele mantém.

---

## 💡 Melhores Práticas

1. **Use o modo imperativo**: `Retorna`, `Processa`, `Calcula`.
2. **Evite detalhes de implementação**: a docstring deve descrever o contrato, não o algoritmo.
3. **Documente parâmetros importantes**: inclua significado, formato esperado e valores especiais.
4. **Seja consistente**: adote um estilo por projeto e mantenha-o.
5. **Prefira docstrings a comentários redundantes**: comentários de implementação devem estar no código, não no docstring.
6. **Use uma linha em branco entre resumo e corpo** quando houver explicação adicional.
7. **Não descreva o nome do parâmetro** se ele já for óbvio demais, mas documente quando o comportamento precisar de contexto.
8. **Documente propriedades e métodos públicos**; itens privados (`_nome`) podem ter docstrings mais simples.
9. **Mantenha as docstrings curtas para helpers simples**; não force se uma função for autoexplicativa e local.
10. **Use `Raises` e `Returns` quando o comportamento não for trivial**.

---

## 🛠️ Ferramentas e automação
- Extensão VS Code: **Python Docstring Generator**.
- Ferramentas de documentação: **Sphinx**, **MkDocs**, **pdoc**, **pydoc**.
- Analisadores estáticos: **flake8-docstrings**, **pydocstyle**.
- Comando útil: `pydocstyle <arquivo>` para validar conformidade com PEP 257.

> Um bom guia de docstrings deve ser claro, consistente e prático. Este documento já oferece uma base forte e pode ser considerado um manual de referência para escrita de docstrings em Python.
