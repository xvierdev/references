# 🌳 Guia Completo: anytree (Estruturas de Árvore em Python)

O `anytree` é uma biblioteca leve e poderosa para construir, navegar, transformar e renderizar árvores em Python. É ideal para hierarquias, árvores de decisão, menus, classificação de categorias e análise de estruturas de dados.

---

## 📑 Sumário
1. [🛠️ Instalação](#-instalação)
2. [🏗️ Criando uma Árvore Básica](#️-criando-uma-árvore-básica)
3. [🧩 NodeMixin e Classes Personalizadas](#-nodemixin-e-classes-personalizadas)
4. [🔧 Mutação e Reorganização](#-mutação-e-reorganização)
5. [🔍 Navegação e Propriedades de Nó](#-navegação-e-propriedades-de-nó)
6. [🧭 Travessia e Ordem](#-travessia-e-ordem)
7. [🎨 Renderização e Visualização](#-renderização-e-visualização)
8. [🚀 Recursos e Truques Avançados](#-recursos-e-truques-avançados)
9. [🔎 Busca e Resolução de Caminhos](#-busca-e-resolução-de-caminhos)
10. [📂 Exportação e Importação (Dict/JSON)](#-exportação-e-importação-dictjson)
11. [📌 Casos de Uso Reais](#-casos-de-uso-reais)
12. [🧠 Performance e Boas Práticas](#-performance-e-boas-práticas)
13. [✅ Checklist de anytree](#-checklist-de-anytree)

---

## 🛠️ Instalação

```bash
pip install anytree
```

Para exportação de imagem:

```bash
pip install graphviz
```

---

## 🏗️ Criando uma Árvore Básica

```python
from anytree import Node, RenderTree

root = Node("CEO")
desenvolvimento = Node("Desenvolvimento", parent=root)
marketing = Node("Marketing", parent=root)

tech_lead = Node("Tech Lead", parent=desenvolvimento)
dev_junior = Node("Dev Junior", parent=tech_lead)

rh = Node("RH", parent=root, responsavel="Ana", funcionarios=5)
```

### Acessando e criando filhos dinamicamente

```python
financeiro = Node("Financeiro")
financeiro.parent = root

gerente = Node("Gerente Financeiro", parent=financeiro)
```

---

## 🧩 NodeMixin e Classes Personalizadas

Use `NodeMixin` quando quiser transformar uma classe de domínio em um nó de árvore com atributos de negócio.

```python
from anytree import NodeMixin, RenderTree

class Pessoa(NodeMixin):
    def __init__(self, nome, cargo, parent=None):
        self.nome = nome
        self.cargo = cargo
        self.parent = parent

root = Pessoa("CEO", "Diretor")
tech_lead = Pessoa("Tech Lead", "Líder de Engenharia", parent=root)
dev = Pessoa("Dev", "Desenvolvedor", parent=tech_lead)

for pre, _, node in RenderTree(root):
    print(f"{pre}{node.nome} ({node.cargo})")
```

---

## 🔧 Mutação e Reorganização

### Adicionar nós

```python
vendas = Node("Vendas", parent=root)
```

### Remover nós

```python
marketing.parent = None  # remove Marketing da árvore
```

### Mover nós entre ramos

```python
dev_junior.parent = root  # move Dev Junior para debaixo de CEO
```

### Atualizar atributos em tempo de execução

```python
rh.funcionarios = 6
rh.responsavel = "Carlos"
```

---

## 🔍 Navegação e Propriedades de Nó

### Propriedades comuns

```python
print(node.name)
print(node.parent)
print(node.children)
print(node.siblings)
print(node.is_root)
print(node.is_leaf)
print(node.depth)
print(node.height)
print(node.level)
```

### Atributos de relacionamento

```python
print(dev_junior.parent.name)
print(root.children[0].siblings)
print(root.descendants)
print(dev_junior.ancestors)
```

### Verificando condição de folha e raiz

```python
if node.is_leaf:
    print("É um nó folha")
if node.is_root:
    print("É raiz")
```

---

## 🧭 Travessia e Ordem

### Travessia pré-ordem (pré-fixado)

```python
from anytree import PreOrderIter

for node in PreOrderIter(root):
    print(node.name)
```

### Travessia pós-ordem (pós-fixado)

```python
from anytree import PostOrderIter

for node in PostOrderIter(root):
    print(node.name)
```

### Travessia em largura

```python
from anytree import LevelOrderGroupIter

for level in LevelOrderGroupIter(root):
    print([node.name for node in level])
```

### Iterar apenas caminhos

```python
from anytree import LevelOrderIter

for node in LevelOrderIter(root):
    print(node.path)
```

---

## 🎨 Renderização e Visualização

### Estilos de Renderização no Terminal
O `RenderTree` aceita diferentes estilos para personalizar as linhas da árvore.

```python
from anytree import RenderTree, AsciiStyle, ContStyle, DoubleStyle

# Estilo Padrão (ContStyle)
for pre, _, node in RenderTree(root, style=ContStyle()):
    print(f"{pre}{node.name}")

# Estilo ASCII (Simples)
for pre, _, node in RenderTree(root, style=AsciiStyle()):
    print(f"{pre}{node.name}")

# Estilo Linha Dupla
for pre, _, node in RenderTree(root, style=DoubleStyle()):
    print(f"{pre}{node.name}")
```

### Customizando a saída
Você pode formatar a saída para incluir outros atributos do nó.

```python
for pre, _, node in RenderTree(root):
    status = "✅" if getattr(node, "ativo", True) else "❌"
    print(f"{pre}{node.name} {status}")
```

### Exportação Graphviz (Imagens e PDF)

```python
from anytree.exporter import DotExporter

# Gera um arquivo .dot para ser processado pelo Graphviz
DotExporter(root).to_dotfile("organizacao.dot")

# Gera uma imagem diretamente (requer Graphviz instalado no sistema)
# DotExporter(root).to_picture("organizacao.png")
```

---

## 🚀 Recursos e Truques Avançados

### 🧭 Walker: Caminho entre dois nós quaisquer
O `Walker` calcula o trajeto exato (subida e descida) para ir de um nó A para um nó B.

```python
from anytree import Walker

w = Walker()
up, common, down = w.walk(dev_junior, marketing)
# up: nós para subir até o ancestral comum
# common: o ancestral comum aos dois
# down: nós para descer até o alvo
```

### 🛡️ Prevenção de Ciclos e Integridade
O `anytree` protege a estrutura da sua árvore. Se você tentar atribuir um filho como pai de um de seus ancestrais, a biblioteca lançará um `LoopError`.

```python
# Isso lançará anytree.iterators.error.LoopError
# dev_junior.parent = tech_lead (OK)
# tech_lead.parent = dev_junior (ERRO: Ciclo detectado)
```

### 🏗️ Criando Árvore a partir de Caminhos (Paths)
Um padrão comum é construir uma árvore a partir de uma lista de strings de diretório/caminho.

```python
def build_tree_from_paths(paths):
    root = Node("root")
    resolver = Resolver("name")
    for path in paths:
        parts = path.strip("/").split("/")
        current = root
        for part in parts:
            try:
                current = resolver.get(current, part)
            except:
                current = Node(part, parent=current)
    return root

caminhos = ["vendas/norte/joao", "vendas/sul/maria", "rh/folha"]
arvore_vendas = build_tree_from_paths(caminhos)
```

---

## 🔎 Busca e Resolução de Caminhos

### Buscar nó por condição

```python
from anytree import find, findall

rh = find(root, lambda node: node.name == "RH")
devs = findall(root, lambda node: "Dev" in node.name)
```

### Resolver caminhos estilo Unix

```python
from anytree import Resolver

resolver = Resolver("name")
no_alvo = resolver.get(root, "/Desenvolvimento/Tech Lead")
```

---

## 📂 Exportação e Importação (Dict/JSON)

### Exportar para dicionário

```python
from anytree.exporter import DictExporter

exporter = DictExporter()
dados_dict = exporter.export(root)
```

### Importar de dicionário

```python
from anytree.importer import DictImporter

importer = DictImporter()
nova_raiz = importer.import_(dados_dict)
```

### Persistir em JSON

```python
import json

with open("arvore.json", "w", encoding="utf-8") as f:
    json.dump(dados_dict, f, ensure_ascii=False, indent=2)
```

---

## 📌 Casos de Uso Reais

- Organogramas de empresa
- Menus de navegação
- Hierarquias de categorias e tags
- Estruturas de decisão e árvores de regras
- Parsing de HTML/XML em modelos de nó
- Relacionamentos pai-filho em dados de auditoria

### Exemplo: menu de navegação

```python
home = Node("Home")
produtos = Node("Produtos", parent=home)
software = Node("Software", parent=produtos)
hardware = Node("Hardware", parent=produtos)
contato = Node("Contato", parent=home)
```

---

## 🧠 Performance e Boas Práticas

- Use `anytree` para árvores de pequeno a médio porte.
- Evite manipular milhões de nós em memória; prefira bancos de grafos ou estruturas especializadas.
- Prefira `find`/`findall` e iteradores em vez de loops manuais de pesquisa quando possível.
- Atualize `.parent` diretamente para mover nós sem recriar a árvore inteira.
- Use `NodeMixin` para integrar a árvore ao seu modelo de domínio sem duplicação de dados.

---

## ✅ Checklist de anytree

- [ ] Saber instalar `anytree` e `graphviz`.
- [ ] Criar árvores básicas com `Node` e `parent`.
- [ ] Usar `NodeMixin` para classes personalizadas.
- [ ] Adicionar, remover e mover nós dinamicamente.
- [ ] Entender propriedades: `is_root`, `is_leaf`, `depth`, `height`, `level`, `siblings`.
- [ ] Fazer travessias: pré-ordem, pós-ordem e nível.
- [ ] Exportar para `RenderTree` com estilos (`AsciiStyle`, `ContStyle`).
- [ ] Usar `Walker` para encontrar caminhos entre dois nós.
- [ ] Entender a proteção contra ciclos (`LoopError`).
- [ ] Criar árvores dinamicamente a partir de caminhos (Strings).
- [ ] Exportar para `.dot`, dicionários e JSON.
- [ ] Resolver caminhos com `Resolver`.
- [ ] Importar árvores de `DictImporter`.
- [ ] Aplicar `anytree` em organogramas, menus e hierarquias de categorias.


---

## 💡 Notas finais

`anytree` é excelente quando você precisa modelar hierarquias claras com mínima complexidade. Para aplicações de grande escala, avalie soluções de grafos e bancos de dados especializados, mas mantenha `anytree` como referência para prototipagem e lógica de árvore em Python.
