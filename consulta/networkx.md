# 🕸️ Guia Completo: NetworkX (Ciência de Redes em Python)

O `NetworkX` é a biblioteca padrão em Python para a criação, manipulação e estudo da estrutura, dinâmica e funções de redes complexas (grafos).

---

## 📑 Sumário
1. [🛠️ Instalação](#-instalação)
2. [🏗️ Tipos de Grafos](#️-tipos-de-grafos)
3. [➕ Adicionando Nós e Arestas](#-adicionando-nós-e-arestas)
4. [🔍 Acessando Dados e Atributos](#-acessando-dados-e-atributos)
5. [📊 Algoritmos Comuns](#-algoritmos-comuns)
6. [🎨 Visualização Básica](#-visualização-básica)
7. [📂 Exportação e Importação](#-exportação-e-importação)

---

## 🛠️ Instalação
```bash
pip install networkx
# Para visualização, você também precisará do Matplotlib
pip install matplotlib
```

---

## 🏗️ Tipos de Grafos
```python
import networkx as nx

# 1. Grafo Simples (Não direcionado)
G = nx.Graph()

# 2. Grafo Direcionado (Dígrafo)
DG = nx.DiGraph()

# 3. Multigrafo (Permite múltiplas arestas entre os mesmos nós)
MG = nx.MultiGraph()
```

---

## ➕ Adicionando Nós e Arestas

### **Trabalhando com Nós**
```python
G.add_node(1)                # Adiciona um nó único
G.add_nodes_from([2, 3, 4])  # Adiciona múltiplos nós de uma lista
G.add_node("A", cargo="CEO") # Adiciona nó com atributos
```

### **Trabalhando com Arestas (Conexões)**
```python
G.add_edge(1, 2)             # Conecta o nó 1 ao 2
G.add_edges_from([(2, 3), (3, 4)]) # Múltiplas conexões

# Arestas com pesos (Weights) - Essencial para algoritmos de caminho
G.add_edge("RJ", "SP", weight=430)
```

---

## 🔍 Acessando Dados e Atributos

```python
# Listar nós e arestas
print(G.nodes)
print(G.edges)

# Verificar se um nó existe
G.has_node("A")

# Acessar vizinhos de um nó
vizinhos = list(G.neighbors(1))

# Acessar atributos de uma aresta
print(G["RJ"]["SP"]["weight"]) # 430
```

---

## 📊 Algoritmos Comuns

### **Caminho Mais Curto (Shortest Path)**
```python
# Sem considerar peso
caminho = nx.shortest_path(G, source=1, target=4)

# Considerando peso (Dijkstra)
caminho_peso = nx.shortest_path(G, source="RJ", target="SP", weight="weight")
```

### **Centralidade (Importância dos Nós)**
```python
# Grau de centralidade (quem tem mais conexões)
deg_centrality = nx.degree_centrality(G)

# PageRank (algoritmo do Google para relevância)
pagerank = nx.pagerank(G)
```

### **Conectividade**
```python
print(nx.is_connected(G)) # Verifica se todos os nós estão alcançáveis
```

---

## 🎨 Visualização Básica
```python
import matplotlib.pyplot as plt

G = nx.complete_graph(5) # Cria um grafo onde todos se conectam a todos

nx.draw(G, with_labels=True, node_color='lightblue', edge_color='gray')
plt.show()
```

---

## 📂 Exportação e Importação
```python
# JSON (Ideal para aplicações Web/D3.js)
import json
data = nx.node_link_data(G)
with open("grafo.json", "w") as f:
    json.dump(data, f)

# GML / GraphML (Formatos padrão de Gephi e outras ferramentas)
nx.write_gml(G, "grafo.gml")
G_carregado = nx.read_gml("grafo.gml")
```

---

## 💡 Dicas de Performance
- Para redes muito grandes (milhões de nós), o `NetworkX` pode ser lento pois é escrito puramente em Python. Nesses casos, considere usar **`graph-tool`** ou **`igraph`** (que possuem core em C++).
- Use `nx.info(G)` para um resumo rápido das estatísticas do seu grafo.

---

## ✅ Checklist de NetworkX
- [ ] Escolheu o tipo de grafo correto (`Graph`, `DiGraph`, `MultiGraph`)?
- [ ] Atributos de peso (`weight`) foram adicionados para algoritmos de caminho?
- [ ] Verificou a conectividade do grafo antes de executar algoritmos globais?
- [ ] Usou nomes de nós descritivos ou IDs consistentes?
- [ ] Considerou o uso de `igraph` ou `graph-tool` para redes de larga escala?
- [ ] Exportou os dados em formatos padrão (`JSON`, `GML`) para interoperabilidade?
