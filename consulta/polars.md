# 🧊 Bíblia do Polars

O Polars é uma biblioteca de DataFrame ultra-rápida em Rust, projetada para processamento de dados escalável e concorrente em Python.

---

## 📑 Sumário
1. [🚀 Introdução e Instalação](#-introdução-e-instalação)
2. [📂 Criação de DataFrames](#-criação-de-dataframes)
3. [📚 Leitura e Escrita](#-leitura-e-escrita)
4. [🔍 Seleção e Filtragem](#-seleção-e-filtragem)
5. [🧱 Expressões e Operações de Coluna](#-expressões-e-operações-de-coluna)
6. [📊 Agrupamentos e Agregações](#-agrupamentos-e-agregações)
7. [🤝 Joins e Combinações](#-joins-e-combinações)
8. [🔄 Reshape e Transformações](#-reshape-e-transformações)
9. [⚡ Lazy API e Performance](#-lazy-api-e-performance)
10. [� Exemplo Prático de ETL](#-exemplo-prático-de-etl)
11. [🪄 Tipos, Nulos e Conversões](#-tipos-nulos-e-conversões)
12. [💡 Dicas e Boas Práticas](#-dicas-e-boas-práticas)

---

## 🚀 Introdução e Instalação

```bash
pip install polars
```

```python
import polars as pl
```

### Principais conceitos
- `DataFrame`: estrutura tabular principal.
- `Series`: coluna única de dados.
- `Expr`: expressão usada em `select`, `with_columns`, `filter`, `group_by`.
- `LazyFrame`: plano de execução adiado para otimização.

---

## 📂 Criação de DataFrames

### De dicionários
```python
df = pl.DataFrame({
    "nome": ["Ana", "Bruno", "Carlos"],
    "idade": [22, 35, 29],
    "salario": [3000, 4500, 3900]
})
```

### De listas e tuplas
```python
df = pl.DataFrame([
    ("Ana", 22, 3000),
    ("Bruno", 35, 4500)
], schema=["nome", "idade", "salario"])
```

### De Series
```python
serie = pl.Series("pontuacao", [10, 20, 30])
df = pl.DataFrame([serie])
```

### Estruturas aninhadas
```python
df = pl.DataFrame({
    "usuario": ["alice", "bob"],
    "meta": [{"likes": 10, "shares": 3}, {"likes": 7, "shares": 1}]
})
```

---

## 📚 Leitura e Escrita

### Leitura de arquivos
```python
df = pl.read_csv("dados.csv")
df = pl.read_parquet("dados.parquet")
df = pl.read_json("dados.json")
df = pl.read_ipc("dados.ipc")
```

### Escrita de arquivos
```python
df.write_csv("saida.csv")
df.write_parquet("saida.parquet")
df.write_json("saida.json")
df.write_ipc("saida.ipc")
```

### Scan (Lazy em arquivo)
```python
lazy = pl.scan_csv("dados.csv")
```

---

## 🔍 Seleção e Filtragem

### Selecionar colunas
```python
df.select(["nome", "idade"])
```

### Colunas por expressão
```python
df.select([
    pl.col("nome"),
    (pl.col("salario") * 1.1).alias("salario_bonus")
])
```

### Filtrar linhas
```python
df_filtrado = df.filter(pl.col("idade") > 18)
```

### Condições compostas
```python
df_filtrado = df.filter(
    (pl.col("idade") > 18) & (pl.col("status") == "Ativo")
)
```

### Operações de string
```python
df.filter(pl.col("email").str.contains("@gmail.com"))
```

### Seleção por índice e fatias
```python
df[0]          # primeira coluna

df[:2]         # primeiras 2 linhas

df[1:4, ["nome", "idade"]]
```

---

## 🧱 Expressões e Operações de Coluna

### `with_columns`
```python
df = df.with_columns([
    (pl.col("salario") * 1.1).alias("salario_bonus"),
    pl.col("data").str.strptime(pl.Date, "%Y-%m-%d").alias("data")
])
```

### Renomear e eliminar
```python
df = df.rename({"nome": "nome_completo", "idade": "anos"})
df = df.drop("coluna_inutil")
```

### Conversões de tipo
```python
df = df.with_columns(pl.col("idade").cast(pl.Int32))
```

### Operadores matemáticos e lógicos
```python
pl.col("salario") + 100
pl.col("idade") * 2
(pl.col("idade") > 18) | (pl.col("status") == "Ativo")
```

### `when/then/otherwise`
```python
df = df.with_columns(
    pl.when(pl.col("idade") >= 18)
      .then("adulto")
      .otherwise("menor")
      .alias("faixa_etaria")
)
```

### Agregações em colunas
```python
df.select([
    pl.col("salario").mean().alias("media"),
    pl.col("salario").sum().alias("total")
])
```

### Operações em listas
```python
df = df.with_columns(
    pl.col("tags").list.lengths().alias("qtd_tags")
)
```

---

## 📊 Agrupamentos e Agregações

### Group By simples
```python
res = df.group_by("setor").agg([
    pl.count("id").alias("total_func"),
    pl.col("salario").mean().alias("media_salarial"),
    pl.col("salario").sum().alias("folha_pagamento")
])
```

### Agregações múltiplas
```python
res = df.groupby("setor").agg([
    pl.col("salario").min().alias("salario_min"),
    pl.col("salario").max().alias("salario_max"),
    pl.col("salario").median().alias("salario_mediana")
])
```

### Aggregation expressions
```python
df.groupby("setor").agg(
    pl.col("salario").agg([pl.mean().alias("media"), pl.std().alias("desvio")])
)
```

### `groupby_rolling` / `groupby_dynamic`
```python
df.groupby_rolling(index_column="data", period="7d").agg(
    pl.col("salario").sum().alias("soma_7d")
)
```

---

## 🤝 Joins e Combinações

### Join simples
```python
df_final = df_a.join(df_b, on="id", how="inner")
```

### Tipos de join
- `inner`, `left`, `outer`, `cross`, `semi`, `anti`

### Join com múltiplas colunas
```python
df_a.join(df_b, on=["id", "data"], how="left")
```

### Concatenação vertical e horizontal
```python
df_concat = pl.concat([df1, df2])
df_cols = pl.concat([df1, df2], how="horizontal")
```

---

## 🔄 Reshape e Transformações

### Melt
```python
long = df.melt(
    id_vars=["id"],
    value_vars=["jan", "fev"],
    variable_name="mes",
    value_name="valor"
)
```

### Pivot
```python
pivot = df.pivot(
    values="valor",
    index="id",
    columns="mes",
    aggregate_function="sum"
)
```

### Explode
```python
df = df.with_columns(pl.col("tags").explode())
```

### Stack e unstack
```python
stacked = df.stack(["a", "b"])
```

### `with_row_count`
```python
df = df.with_row_count("row_nr")
```

### Ordenar e selecionar top N
```python
df.sort("salario", descending=True).head(10)
```

---

## ⚡ Lazy API e Performance

### Criar LazyFrame
```python
lazy = pl.scan_csv("dados.csv")
```

### Operações lazy
```python
lazy = (
    pl.scan_csv("dados.csv")
      .filter(pl.col("idade") > 18)
      .select(["nome", "salario"])
      .sort("salario", reverse=True)
)
```

### Coletar resultados
```python
df_final = lazy.collect()
```

### Vantagens do LazyFrame
- Otimização de plano de execução
- Fusão de operações
- Menos cópias de memória
- Processamento em lote e paralelismo automático

### Cache e parquet otimizado
```python
lazy = pl.scan_parquet("dados.parquet")
```

---

## � Exemplo Prático de ETL

### Pipeline completo com LazyFrame e joins
```python
vendas = (
    pl.scan_csv("vendas.csv")
      .with_columns(
          pl.col("data_venda").str.strptime(pl.Date, "%Y-%m-%d")
      )
      .filter(pl.col("valor") > 0)
)

clientes = (
    pl.scan_csv("clientes.csv")
      .with_columns(
          pl.col("data_cadastro").str.strptime(pl.Date, "%Y-%m-%d")
      )
)

produtos = pl.scan_csv("produtos.csv")

pipeline = (
    vendas
      .join(clientes, left_on="cliente_id", right_on="id", how="left")
      .join(produtos, left_on="produto_id", right_on="id", how="left")
      .with_columns([
          (pl.col("valor") * pl.col("quantidade")).alias("valor_total"),
          pl.col("data_venda").dt.month().alias("mes"),
          pl.col("data_venda").dt.year().alias("ano")
      ])
      .groupby(["ano", "mes", "categoria"])
      .agg([
          pl.sum("valor_total").alias("receita"),
          pl.count("id").alias("vendas"),
          pl.mean("valor_total").alias("ticket_medio")
      ])
      .sort(["ano", "mes"], reverse=[False, False])
)

resultado = pipeline.collect()
resultado.write_parquet("relatorio_vendas.parquet")
```

### Observações
- O `scan_csv` cria um `LazyFrame` e adianta a execução.
- O pipeline só é executado no `collect()`.
- O `join` no nível de `LazyFrame` mantém as junções otimizadas.
- Use `.with_columns()` para calcular métricas agregadas no fluxo.

---

## �🪄 Tipos, Nulos e Conversões

### Tipos de dados comuns
- `pl.Int8`, `pl.Int16`, `pl.Int32`, `pl.Int64`
- `pl.Float32`, `pl.Float64`
- `pl.Utf8`, `pl.Boolean`
- `pl.Date`, `pl.Datetime`, `pl.Time`
- `pl.Duration`, `pl.List`, `pl.Struct`

### Trabalhando com valores nulos
```python
df = df.with_columns(
    pl.col("salario").fill_null(0)
)

df = df.drop_nulls(subset=["nome"])
```

### Preencher e substituir nulos
```python
df = df.fill_null("desconhecido")
```

### Conversões de tipo
```python
df = df.with_columns(
    pl.col("data").str.strptime(pl.Date, "%Y-%m-%d")
)
```

### Datas e horários
```python
df = df.with_columns(
    pl.col("data").dt.year().alias("ano"),
    pl.col("data").dt.month().alias("mes")
)
```

---

## 💡 Dicas e Boas Práticas

- Prefira `LazyFrame` em pipelines de ETL grandes.
- Use `pl.col()` e expressões ao invés de `apply()` sempre que possível.
- Evite iterar linha a linha; use operações vetorizadas.
- Use `pl.concat` para juntar DataFrames semelhantes.
- Use `pl.scan_parquet` para arquivos grandes e processamento preguiçoso.
- Verifique tipos com `df.dtypes` e `df.schema`.
- Use `df.describe()` para inspecionar dados rapidamente.

---

## ✅ Checklist de Polars
- [ ] Usou `LazyFrame` (`scan_*`) para processamento de grandes volumes?
- [ ] Evitou o uso de `.apply()` e preferiu expressões nativas (`pl.col`)?
- [ ] Validou os tipos de dados com `.schema` após transformações?
- [ ] Utilizou `.collect()` apenas no final do pipeline lazy?
- [ ] Tratou valores nulos explicitamente com `fill_null` ou `drop_nulls`?
- [ ] Aproveitou o paralelismo nativo ao evitar loops Python sobre DataFrames?

---

## 📚 Referências Rápidas
- `pl.DataFrame`, `pl.Series`
- `pl.col`, `pl.lit`, `pl.when`
- `df.select`, `df.filter`, `df.with_columns`
- `df.group_by`, `df.join`, `pl.concat`
- `pl.scan_csv`, `lazy.collect`, `df.write_parquet`
