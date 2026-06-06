# 🗄️ Bíblia do SQL: O Guia Supremo (Referência Completa e Performance)

Este guia é a referência definitiva para o ecossistema SQL. Ele une a exaustividade técnica de sintaxe (DDL, DML, DCL, TCL) com estratégias avançadas de performance, segurança e padrões de produção.

---

## 📑 Sumário
1. [🏗️ DDL - Data Definition Language](#-ddl---data-definition-language)
2. [✍️ DML - Data Manipulation Language](#-dml---data-manipulation-language)
3. [🔐 DCL - Data Control Language](#-dcl---data-control-language)
4. [🔄 TCL - Transaction Control Language](#-tcl---transaction-control-language)
5. [🛡️ Segurança e SQL Injection](#️-segurança-e-sql-injection)
6. [🧾 Constraints e Chaves](#-constraints-e-chaves)
7. [⚡ Índices e Performance](#-índices-e-performance)
8. [🖼️ Views](#-views)
9. [📦 Stored Procedures & Funções](#-stored-procedures--funções)
10. [🛡️ Triggers](#-triggers)
11. [🤝 Joins (Junções)](#-joins-junções)
12. [🧠 Subqueries e CTEs](#-subqueries-e-ctes)
13. [🔄 Set Operations](#-set-operations)
14. [📊 Funções de Agregação e Agrupamento](#-funções-de-agregação-e-agrupamento)
15. [📌 Window Functions](#-window-functions)
16. [⚡ Performance Avançada: EXPLAIN ANALYZE](#-performance-avançada-explain-analyze)
17. [🚫 Anti-Patterns: O que evitar](#-anti-patterns-o-que-evitar)
18. [✅ Checklist de Queries de Produção](#-checklist-de-queries-de-produção)
19. [📚 Dicas e Boas Práticas](#-dicas-e-boas-práticas)

---

## 🏗️ DDL - Data Definition Language
Comandos que definem e alteram a estrutura do banco de dados.

### Criar banco e tabelas
```sql
CREATE DATABASE meu_banco;
CREATE TABLE usuarios (
    id INT PRIMARY KEY GENERATED ALWAYS AS IDENTITY, -- Padrão Moderno
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    criado_em TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

### Alterar tabelas
```sql
ALTER TABLE usuarios ADD COLUMN status VARCHAR(20) DEFAULT 'ativo';
ALTER TABLE usuarios DROP COLUMN status;
ALTER TABLE usuarios ALTER COLUMN nome SET NOT NULL;
ALTER TABLE usuarios RENAME COLUMN nome TO nome_completo;
ALTER TABLE usuarios RENAME TO clientes;
```

### Excluir objetos
```sql
DROP TABLE IF EXISTS usuarios;
DROP DATABASE IF EXISTS meu_banco;
TRUNCATE TABLE usuarios;
```

### Tipos de dados comuns
- `INT`, `BIGINT`, `SMALLINT`
- `DECIMAL(p, s)`, `NUMERIC(p, s)`
- `FLOAT`, `DOUBLE`
- `CHAR(n)`, `VARCHAR(n)`, `TEXT`
- `DATE`, `TIME`, `TIMESTAMP`, `TIMESTAMPTZ`
- `BOOLEAN`
- `JSON`, `JSONB`
- `BLOB`, `BYTEA`
- `UUID`, `ENUM`, `INTERVAL`

> A sintaxe exata varia entre MySQL, PostgreSQL, SQL Server e outros.

---

## ✍️ DML - Data Manipulation Language
Comandos para inserir, atualizar, excluir e consultar dados.

### INSERT e UPSERT
```sql
INSERT INTO usuarios (nome, email) VALUES ('Ana', 'ana@example.com');

-- Insert via Select
INSERT INTO usuarios (nome, email)
SELECT nome, email FROM leads WHERE optin = TRUE;

-- UPSERT (Insert or Update - PostgreSQL)
INSERT INTO usuarios (email, nome) VALUES ('ana@ex.com', 'Ana Silva')
ON CONFLICT (email) DO UPDATE SET nome = EXCLUDED.nome;
```

### UPDATE
```sql
UPDATE usuarios
SET status = 'inativo'
WHERE ultima_atividade < '2023-01-01';
```

### DELETE
```sql
DELETE FROM usuarios WHERE status = 'cancelado';
```

### SELECT básico
```sql
SELECT id, nome, email
FROM usuarios
WHERE status = 'ativo'
ORDER BY criado_em DESC
LIMIT 10 OFFSET 0;
```

### Claúsulas importantes
- `SELECT [DISTINCT] cols`
- `FROM tabela`
- `WHERE condição`
- `GROUP BY cols`
- `HAVING condição`
- `ORDER BY cols [ASC|DESC]`
- `LIMIT n [OFFSET m]`

---

## 🔐 DCL - Data Control Language
Controle de acesso e privilégios de usuários.

```sql
GRANT SELECT, INSERT ON usuarios TO analista;
REVOKE UPDATE ON usuarios FROM analista;
GRANT ALL PRIVILEGES ON banco.* TO 'app'@'localhost';
```

### Conceitos
- `GRANT`: concede permissão.
- `REVOKE`: remove permissão.
- `ROLE`: conjunto de privilégios (Ex: `CREATE ROLE leitor;`).
- `WITH GRANT OPTION`: permite que o usuário delegue privilégios.

---

## 🔄 TCL - Transaction Control Language
Gerencia transações e garantias ACID.

```sql
BEGIN TRANSACTION; -- Ou apenas BEGIN
UPDATE contas SET saldo = saldo - 100 WHERE id = 1;
UPDATE contas SET saldo = saldo + 100 WHERE id = 2;
COMMIT;
```

### Rollback e savepoints
```sql
SAVEPOINT antes_da_transferencia;
UPDATE contas SET saldo = saldo - 100 WHERE id = 1;
ROLLBACK TO antes_da_transferencia;
ROLLBACK;
```

### Isolamento de transação
- `READ UNCOMMITTED`
- `READ COMMITTED`
- `REPEATABLE READ`
- `SERIALIZABLE`

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

## 🛡️ Segurança e SQL Injection
**NUNCA** use interpolação de strings para entradas de usuário.

```python
# ERRADO (Vulnerável)
cursor.execute(f"SELECT * FROM users WHERE name = '{user_input}'")

# CORRETO (Parametrizado)
cursor.execute("SELECT * FROM users WHERE name = %s", (user_input,))
```

---

## 🧾 Constraints e Chaves
Definem regras de integridade de dados.

```sql
CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INT NOT NULL,
    status VARCHAR(20) NOT NULL,
    total DECIMAL(10, 2) NOT NULL CHECK (total >= 0),
    CONSTRAINT fk_cliente FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);
```

- `PRIMARY KEY`: identifica unicamente cada linha.
- `FOREIGN KEY`: garante integridade referencial.
- `UNIQUE`: valores únicos em uma coluna ou conjunto de colunas.
- `NOT NULL`: não permite valores nulos.
- `CHECK`: valida uma condição para cada linha.
- `DEFAULT`: valor padrão quando o campo não é informado.

### Adicionando restrições
```sql
ALTER TABLE usuarios ADD CONSTRAINT uk_email UNIQUE (email);
ALTER TABLE pedidos ADD CONSTRAINT chk_total CHECK (total >= 0);
```

---

## ⚡ Índices e Performance
Aceleram consultas, mas também aumentam custo de escrita.

```sql
CREATE INDEX idx_usuarios_email ON usuarios (email);
CREATE UNIQUE INDEX idx_unico_cpf ON clientes (cpf);
CREATE INDEX idx_pedidos_data ON pedidos (data_pedido DESC);
```

### Tipos de índice
- **Índice simples**: Em uma única coluna.
- **Índice composto**: `(coluna1, coluna2)`.
- **Índice único**: Impede duplicatas.
- **Índice parcial / filtrado**: `WHERE status = 'active'`.
- **Índice de expressão**: `LOWER(email)`.
- **Índice full-text**: GIN / GIST para buscas de texto.

### Boas práticas
- Crie índices em colunas usadas em `WHERE`, `JOIN` e `ORDER BY`.
- Evite índices em colunas com baixa cardinalidade (Ex: booleanos).
- Monitore o uso com `EXPLAIN`.

---

## 🖼️ Views
Tabelas lógicas baseadas em consultas.

```sql
CREATE VIEW vista_usuarios_ativos AS
SELECT id, nome, email
FROM usuarios
WHERE status = 'ativo';

SELECT * FROM vista_usuarios_ativos;
DROP VIEW IF EXISTS vista_usuarios_ativos;
CREATE OR REPLACE VIEW vista_usuarios_ativos AS ...;
```

### Tipos de views
- **Simples**: Apenas uma consulta nomeada.
- **Materializada**: Cacheada em disco (`CREATE MATERIALIZED VIEW`).
- **Atualizável**: Permite INSERT/UPDATE em alguns casos.

---

## 📦 Stored Procedures & Funções
Lógica armazenada no banco.

### Procedure (Execução)
```sql
CREATE PROCEDURE atualizar_status(IN p_id INT, IN p_status VARCHAR(20))
LANGUAGE plpgsql AS $$
BEGIN
    UPDATE usuarios SET status = p_status WHERE id = p_id;
END;
$$;

CALL atualizar_status(1, 'ativo');
```

### Função (Retorno)
```sql
CREATE FUNCTION total_com_desconto(valor DECIMAL(10,2), desconto INT)
RETURNS DECIMAL(10,2) AS $$
BEGIN
    RETURN valor * (1 - desconto / 100.0);
END;
$$ LANGUAGE plpgsql;

SELECT total_com_desconto(100, 10);
``` 

---

## 🛡️ Triggers
Ações automáticas disparadas por eventos.

```sql
CREATE TRIGGER atualiza_estoque
AFTER INSERT ON vendas
FOR EACH ROW
EXECUTE FUNCTION diminuir_estoque();
```

### Tipos de gatilhos
- `BEFORE INSERT`, `AFTER INSERT`
- `BEFORE UPDATE`, `AFTER UPDATE`
- `BEFORE DELETE`, `AFTER DELETE`
- `FOR EACH ROW` vs `FOR EACH STATEMENT`

---

## 🤝 Joins (Junções)
Combine dados de várias tabelas.

```sql
SELECT u.nome, p.nome AS produto
FROM usuarios u
INNER JOIN pedidos pd ON u.id = pd.cliente_id
LEFT JOIN produtos p ON pd.produto_id = p.id;
```

### Tipos de joins
- **INNER JOIN**: Linhas que combinam em ambas as tabelas.
- **LEFT JOIN**: Todas da esquerda + match da direita.
- **RIGHT JOIN**: Todas da direita + match da esquerda.
- **FULL OUTER JOIN**: Todas as linhas de ambas as tabelas.
- **CROSS JOIN**: Produto cartesiano (N * M).
- **SELF JOIN**: Join da tabela com ela mesma.

### Sintaxe com `USING`
```sql
SELECT * FROM pedidos JOIN clientes USING (cliente_id);
```

---

## 🧠 Subqueries e CTEs
Subconsultas e consultas comuns tornam queries modulares.

### Subqueries
```sql
SELECT nome FROM usuarios
WHERE id IN (SELECT cliente_id FROM pedidos WHERE total > 100);
```

### Subquery escalar
```sql
SELECT nome,
       (SELECT COUNT(*) FROM pedidos WHERE pedidos.cliente_id = usuarios.id) AS total_pedidos
FROM usuarios;
```

### EXISTS (Geralmente mais performático)
```sql
SELECT nome FROM usuarios u
WHERE EXISTS (
    SELECT 1 FROM pedidos p WHERE p.cliente_id = u.id AND p.total > 100
);
```

### CTE - Common Table Expression
```sql
WITH vendas_por_cliente AS (
    SELECT cliente_id, SUM(total) AS valor_total
    FROM pedidos GROUP BY cliente_id
)
SELECT u.nome, v.valor_total
FROM usuarios u
JOIN vendas_por_cliente v ON u.id = v.cliente_id;
```

### CTE recursiva (Hierarquias)
```sql
WITH RECURSIVE hierarquia AS (
    SELECT id, nome, gerente_id FROM funcionarios WHERE gerente_id IS NULL
  UNION ALL
    SELECT f.id, f.nome, f.gerente_id
    FROM funcionarios f
    JOIN hierarquia h ON f.gerente_id = h.id
)
SELECT * FROM hierarquia;
```

---

## 🔄 Set Operations
Combine e compare conjuntos de resultados.

- `UNION`: Une e remove duplicatas.
- `UNION ALL`: Une e mantém duplicatas (mais rápido).
- `INTERSECT`: Apenas resultados comuns.
- `EXCEPT` / `MINUS`: O que existe no primeiro mas não no segundo.

---

## 📊 Funções de Agregação e Agrupamento
Cálculos sobre conjuntos de linhas.

```sql
SELECT status,
       COUNT(*) AS total,
       SUM(total) AS valor_total,
       AVG(total) AS media,
       MIN(total) AS menor_valor,
       MAX(total) AS maior_valor,
       STRING_AGG(nome, ', ') AS lista_nomes -- No Postgres
FROM pedidos
GROUP BY status
HAVING COUNT(*) > 10
ORDER BY valor_total DESC;
```

### Funções comuns
- `COUNT(expr)`, `SUM(expr)`, `AVG(expr)`, `MIN(expr)`, `MAX(expr)`
- `GROUP_CONCAT(expr)` (MySQL) / `STRING_AGG(expr, ', ')` (Postgres)
- `COUNT(DISTINCT expr)` (Valores únicos)

---

## 📌 Window Functions
Agregações parciais e cálculos em janelas de linhas.

```sql
SELECT id, nome, total,
       RANK() OVER (ORDER BY total DESC) as ranking,
       DENSE_RANK() OVER (ORDER BY total DESC) as rank_sem_pulos,
       LAG(total) OVER (PARTITION BY cliente_id ORDER BY data) as anterior,
       LEAD(total) OVER (PARTITION BY cliente_id ORDER BY data) as proximo,
       SUM(total) OVER (PARTITION BY cliente_id) as soma_acumulada
FROM pedidos;
```

### Partições e ordenação
```sql
SELECT nome, salario,
       AVG(salario) OVER (PARTITION BY departamento) AS media_depto
FROM funcionarios;
```

---

## ⚡ Performance Avançada: EXPLAIN ANALYZE
Rode `EXPLAIN ANALYZE <query>` para ver o plano real de execução.

- **Seq Scan**: Leitura completa da tabela (ruim em tabelas grandes).
- **Index Scan**: Uso do índice (bom).
- **Index Only Scan**: Leu apenas o índice, sem tocar na tabela (ótimo).
- **Nested Loop**: Cruzamento de tabelas; verifique se a tabela interna tem índice.

---

## 🚫 Anti-Patterns: O que evitar

1. **SELECT ***: Gasta rede e CPU, impede Index Only Scan.
2. **LIKE '%termo%'**: O wildcard no início mata o índice B-Tree.
3. **Funções no WHERE**: `WHERE YEAR(data) = 2023` mata o índice. Use ranges.
4. **N+1 no Banco**: Fazer queries dentro de loops no código Python.
5. **Falta de LIMIT**: Queries que retornam milhões de linhas e travam o app.

---

## ✅ Checklist de Queries de Produção

- [ ] Query testada com `EXPLAIN ANALYZE`?
- [ ] Colunas do `WHERE` e `JOIN` estão indexadas?
- [ ] Usando `SELECT col1, col2` em vez de `SELECT *`?
- [ ] Entrada do usuário parametrizada (Segurança)?
- [ ] Operações de escrita em massa usam transações?
- [ ] Fuso horário considerado (`TIMESTAMPTZ`)?
- [ ] Lógica de `NULL` testada (`IS NULL`)?

---

## 📚 Dicas e Boas Práticas
- Normalize dados até a 3ª forma normal.
- Prefira colunas indexadas em filtros.
- Padronize nomes de tabelas e colunas.
- Documente a modelagem de dados.
- Evite lógica complexa demais em Triggers e Procedures (difícil de debugar).

---

## 📝 Resumo
Este guia cobre o ciclo completo do SQL, da criação do esquema à performance em produção.

> SQL é um padrão amplo; detalhes de sintaxe podem variar entre MySQL, PostgreSQL, SQL Server, Oracle e SQLite.
