# 🐼 Bíblia do Pandas

Guia completo e prático para trabalhar com `pandas` em análise de dados, limpeza, modelagem e produção.

---

## 📑 Sumário rápido
1. [Início rápido](#-1-início-rápido)
2. [I/O (CSV, Excel, Parquet, SQL)](#-2-io)
3. [Indexação e seleção (`loc`, `iloc`, `at`, `iat`, `query`)](#-3-indexação-e-seleção)
4. [Operações com colunas](#-4-operações-com-colunas)
5. [Missing data](#-5-tratamento-de-valores-faltantes)
6. [Reshaping (pivot, melt, stack/unstack, explode)](#-6-reshaping-pivot-melt-stackunstack-explode)
7. [GroupBy, agregações e janelas móveis](#-7-groupby-agregações-e-janelas-móveis)
8. [Joins e merges avançados](#-8-joins-e-merges-avançados)
9. [Tipos de dados e otimização de memória](#-9-tipos-de-dados-e-otimização-de-memória)
10. [Time series e resample](#-10-time-series-e-resample)
11. [Accessors: `.str`, `.dt`, `.cat`](#-11-accessors-úteis)
12. [Performance e escala](#-12-performance-e-escala)
13. [Boas práticas e armadilhas comuns](#-13-boas-práticas-e-armadilhas)
14. [Cookbook (receitas práticas)](#-14-cookbook--receitas-práticas)

---

## 🚀 1. Início rápido
```python
import pandas as pd
```

## 📂 2. I/O
```python
# Leitura
df = pd.read_csv("dados.csv", parse_dates=["data"], dtype={"id": "Int64"})
df = pd.read_excel("dados.xlsx", sheet_name=0)
df = pd.read_parquet("dados.parquet")
df = pd.read_sql("SELECT * FROM tabela", con=engine)

# Escrita
df.to_csv("saida.csv", index=False)
df.to_parquet("saida.parquet", compression="snappy")
df.to_sql("tabela", con=engine, if_exists="append", index=False)
```

### Exemplo `read_sql` com SQLite in-memory
```python
import sqlite3

conn = sqlite3.connect(":memory:")
df_tmp = pd.DataFrame({"id": [1, 2], "val": [100, 200]})
df_tmp.to_sql("tabela_tmp", conn, index=False)
df_read = pd.read_sql_query("SELECT * FROM tabela_tmp", conn)
print(df_read)
```

---

## 🧭 3. Indexação e seleção
```python
# Seleção por rótulo
df.loc[10, "nome"]
df.loc[10:20, ["nome", "idade"]]

# Seleção por posição
df.iloc[0, 1]
df.iloc[:5, :3]

# Acesso rápido a elementos
df.at[10, "nome"]
df.iat[0, 1]

# Boolean indexing
df[(df["idade"] > 18) & (df["status"] == "Ativo")]

# Query para filtragem legível
df.query("idade > 18 and status == 'Ativo'")

# Usar .loc com máscaras
mask = df["score"] > 0.8
df.loc[mask, "categoria"] = "alto"
```

---

## 🧾 4. Operações com colunas
```python
# Seleção
df_sub = df[["nome", "idade"]]

# Renomear
df = df.rename(columns={"nome": "nome_completo"})

# Criar/atualizar
df["salario_bonus"] = df["salario"] * 1.1

# Converter tipos
df["data"] = pd.to_datetime(df["data"], utc=True)
df["idade"] = df["idade"].astype("Int64")  # dtype nullable

# Remover
df = df.drop(columns=["coluna_inutil"])
```

---

## ❓ 5. Tratamento de valores faltantes
```python
# Contar nulos
df.isna().sum()

# Remover linhas/colunas
df.dropna(subset=["idade"])  # remove linhas sem idade

# Preencher
df["idade"] = df["idade"].fillna(df["idade"].median())

# Interpolação
df["valor"] = df["valor"].interpolate(method="linear")

# Preenchimento condicional
df.loc[df["score"].isna(), "score"] = 0
```

---

## 🔀 6. Reshaping: pivot, melt, stack/unstack, explode
```python
# Pivot simples
pv = df.pivot(index="data", columns="categoria", values="valor")

# Pivot table com agregação
pt = df.pivot_table(index=["user_id"], columns=["month"], values="sales", aggfunc="sum", fill_value=0)

# Melt (wide -> long)
ldf = df.melt(id_vars=["id"], value_vars=["jan", "feb"], var_name="month", value_name="revenue")

# Stack / unstack
stacked = df.set_index(["a", "b"]).stack()
unstacked = stacked.unstack()

# Explode listas em linhas
df_expl = df.explode("tags")
```

---

## 📊 7. GroupBy, agregações e janelas móveis
```python
# GroupBy básico
g = df.groupby("categoria").agg(total=("valor", "sum"), avg=("valor", "mean"))

# Aggregações customizadas
g = df.groupby("user").agg({"valor": ["sum", lambda x: x.max() - x.min()]})

# Transform e apply
df["rank"] = df.groupby("categoria")["valor"].transform(lambda x: x.rank(ascending=False))

# Rolling
df["rolling_mean_7d"] = df["valor"].rolling(window=7, min_periods=1).mean()

# Expanding
df["cum_sum"] = df["valor"].expanding().sum()
```

---

## 🤝 8. Joins e merges avançados
```python
# Merge padrão
df_merged = pd.merge(left, right, on="id", how="left")

# Merge asof (junção por proximidade temporal)
df_merged = pd.merge_asof(left.sort_values("time"), right.sort_values("time"), on="time", by="symbol", direction="backward")

# Merge ordered (join mantendo ordem)
df_ordered = pd.merge_ordered(df1, df2, on="date", fill_method="ffill")
```

### Exemplo prático de `merge_asof`
```python
# Suponha eventos e preços ordenados por `time` e agrupados por `symbol`
events = pd.DataFrame({
    "symbol": ["A", "A", "B"],
    "time": pd.to_datetime(["2020-01-01 09:00", "2020-01-01 09:05", "2020-01-02 10:00"]),
    "event": ["e1", "e2", "e3"]
})
prices = pd.DataFrame({
    "symbol": ["A", "A", "B"],
    "time": pd.to_datetime(["2020-01-01 08:59", "2020-01-01 09:03", "2020-01-02 09:59"]),
    "price": [10.0, 11.0, 20.0]
})

events = events.sort_values("time")
prices = prices.sort_values("time")
merged = pd.merge_asof(events, prices, on="time", by="symbol", direction="backward")
print(merged)
```

---

## 🧠 9. Tipos de dados e otimização de memória
```python
# Tipos categóricos
df["categoria"] = df["categoria"].astype("category")

# Nullable integers
df["id"] = df["id"].astype("Int64")

# Conversão com ganhos de memória
df["flag"] = df["flag"].astype("bool")

# Ver uso de memória
df.memory_usage(deep=True)
```

---

## ⏱️ 10. Time series e resample
```python
df["data"] = pd.to_datetime(df["data"])
df = df.set_index("data")

# Resample diário -> mensal
monthly = df["valor"].resample("M").sum()

# Shift e lag
df["lag1"] = df["valor"].shift(1)

# Timezone
df["data"] = df["data"].dt.tz_localize("UTC").dt.tz_convert("America/Sao_Paulo")
```

---

## 🔍 11. Accessors úteis
```python
# String operations
df["email"].str.lower().str.contains("@gmail.com")

# Datetime access
df["data"].dt.year
df["data"].dt.to_period("M")

# Categorical
df["cat"].cat.codes
```

---

## ⚡ 12. Performance e escala
```python
# Ler em chunks
for chunk in pd.read_csv("large.csv", chunksize=100_000):
    process(chunk)

# Usar dtypes ao ler
pd.read_csv("large.csv", dtype={"id": "int32", "flag": "bool"})

# Usar parquet para leitura/escrita rápida
df.to_parquet("data.parquet")

# Usar pd.eval / numexpr para expressões vetorizadas
df["z"] = pd.eval("df.x + df.y")
```

### Exemplo: processamento por chunks
```python
import tempfile, os

tmp = tempfile.NamedTemporaryFile(delete=False, mode='w', newline='')
try:
    tmp.write('a,b\n')
    for i in range(1000):
        tmp.write(f"{i},{i*2}\n")
    tmp.flush()
    tmp_name = tmp.name
finally:
    tmp.close()

count = 0
for chunk in pd.read_csv(tmp_name, chunksize=200):
    count += len(chunk)
print('total_rows', count)
os.unlink(tmp_name)
```

Considerar Dask para datasets que não cabem em memória.

---

## ⚠️ 13. Boas práticas e armadilhas
- Prefira atribuição explícita em vez de `inplace=True` (ex.: `df = df.drop(...)`).
- Atenção ao `SettingWithCopyWarning`: use `.loc` para atribuições seguras.
- Sempre valide tipos antes de operações intensivas (join/groupby).
- Teste pipelines com amostras e use `chunksize` para produção.

---

## 🧰 14. Cookbook — receitas práticas
1) Limpeza rápida
```python
df = (df.dropna(subset=["id"])  # garantir id
        .assign(data=lambda d: pd.to_datetime(d["data"]))
        .astype({"id": "Int64"}))
```

2) Join temporal (preço mais recente antes do evento)
```python
events = events.sort_values("time")
prices = prices.sort_values("time")
merged = pd.merge_asof(events, prices, on="time", by="symbol", direction="backward")
```

3) Pivot complexo
```python
tbl = (df.pivot_table(index=["user_id"], columns=["category"], values="value", aggfunc="sum", fill_value=0)
       .reset_index())
```

---

## ✅ Recursos adicionais
- Documentação oficial: https://pandas.pydata.org/
- Livros e cheatsheets: pandas cheat sheet (O'Reilly / PyData)