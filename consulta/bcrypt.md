# 🔑 Guia de bcrypt (Hashing de Senhas)

O `bcrypt` é uma biblioteca para hashing de senhas segura, projetada para ser lenta e resistente a ataques de força bruta.

---

## 📑 Sumário
1. [🛠️ Instalação](#️-instalação)
2. [📝 Uso Prático](#️-uso-prático)
3. [🔧 Ajuste de custo (`rounds`)](#️-ajuste-de-custo-rounds)
4. [🧾 Entendendo o formato do hash](#️-entendendo-o-formato-do-hash)
5. [🧠 Limite de 72 bytes e Unicode](#️-limite-de-72-bytes-e-unicode)
6. [🔄 Rehash e migração](#️-rehash-e-migração)
7. [🛡️ `bcrypt.kdf`](#️-bcryptkdf)
8. [📌 Quando usar bcrypt (e quando não usar)](#️-quando-usar-bcrypt-e-quando-não-usar)
9. [📊 Comparação rápida](#️-comparação-rápida)
10. [💡 Boas práticas de armazenamento](#️-boas-práticas-de-armazenamento)

---

## 🛠️ Instalação
```bash
pip install bcrypt
```

---

## 📝 Uso Prático

```python
import bcrypt

# 1. Gerar hash de uma senha (cadastro)
senha_plana = "minha_senha_secreta"
# Gera um salt aleatório e calcula o hash
salt = bcrypt.gensalt()
senha_hash = bcrypt.hashpw(senha_plana.encode("utf-8"), salt)

print(f"Hash gerado: {senha_hash}")

# 2. Verificar senha (login)
senha_digitada = "minha_senha_secreta"

if bcrypt.checkpw(senha_digitada.encode("utf-8"), senha_hash):
    print("Senha correta! Acesso permitido.")
else:
    print("Senha incorreta!")
```

---

## 🔧 Ajuste de custo (`rounds`)
O `rounds` controlam a complexidade do hashing. Quanto maior o valor, mais tempo leva para gerar e verificar o hash.

```python
salt = bcrypt.gensalt(rounds=12)
senha_hash = bcrypt.hashpw(b"senha", salt)
```

- `rounds=12` é um valor comum em 2024, mas pode subir com o avanço de hardware.
- Teste o tempo de hash no seu ambiente: um valor aceitável geralmente fica entre 100ms e 500ms.
- Atualize o custo periodicamente para manter a segurança.

---

## 🧾 Entendendo o formato do hash
Um hash `bcrypt` tem o formato `$2b$12$...` ou similar.

Exemplo:
```
$2b$12$wJ2nHckYw7z9aV92gKkT3.TUQ8DHWxy7zvQ/ooQq5Y.uyqUlYg4oy
```

- `$2b$`: o identificador da versão do bcrypt.
- `12`: o fator de custo (`rounds`).
- As partes seguintes incluem o `salt` e o hash em base64.
- O `salt` está embutido no próprio hash, portanto você só precisa armazenar o hash.

---

## 🧠 Limite de 72 bytes e Unicode
O bcrypt processa no máximo 72 bytes da senha.

- Senhas mais longas são truncadas silenciosamente, o que pode causar vulnerabilidades.
- Para evitar isso, normalize e limite o tamanho da senha antes do hashing.

```python
senha_plana = "senha muito longa..."
senha_bytes = senha_plana.encode("utf-8")[:72]
senha_hash = bcrypt.hashpw(senha_bytes, bcrypt.gensalt())
```

### Unicode e normalização
Use normalização Unicode (`NFC` ou `NFKC`) para garantir que representações equivalentes sejam tratadas da mesma forma.

```python
import unicodedata

senha = unicodedata.normalize("NFC", "pásswörd")
senha_hash = bcrypt.hashpw(senha.encode("utf-8"), bcrypt.gensalt())
```

---

## 🔄 Rehash e migração
É uma boa prática rehash quando o custo aumenta ou quando a senha foi criada com um parâmetro fraco.

```python
if bcrypt.checkpw(senha_digitada.encode("utf-8"), senha_hash):
    rounds = bcrypt.gensalt().decode().split("$")[2]
    if int(rounds) < 12:
        novo_hash = bcrypt.hashpw(senha_digitada.encode("utf-8"), bcrypt.gensalt(rounds=12))
        # atualize o hash no banco
```

### Quando rehash é necessário
- custo (`rounds`) abaixo do padrão atual
- hash foi gerado com versão antiga (`$2a$` ou `$2y$`)
- política de segurança mudou

---

## 🛡️ `bcrypt.kdf`
O `bcrypt` também oferece uma função de derivação de chave (KDF) para autenticação forte de senha ou geração de chaves.

```python
from bcrypt import kdf

key = kdf(
    password=b"senha",
    salt=b"salt_forte" * 2,
    desired_key_bytes=32,
    rounds=100,
)
```

`bcrypt.kdf` é útil quando você precisa de uma chave simétrica derivada de uma senha, mas para hashing de senha de usuário o `bcrypt.hashpw` continua sendo a escolha padrão.

---

## 📌 Quando usar bcrypt (e quando não usar)
Use `bcrypt` para:
- hashing de senhas de usuário
- armazenamento seguro de credenciais
- autenticação de login

Não use `bcrypt` para:
- hashes gerais de arquivos ou dados arbitrários
- proteger chaves criptográficas
- quando precisar de hashing de alta performance em grandes volumes

Para novos projetos, considere também `argon2` como alternativa moderna e recomendada pelo OWASP.

---

## 📊 Comparação rápida
- `bcrypt`: seguro, amplamente suportado, ideal para senhas de usuário.
- `argon2`: melhor resistência a ataques de GPU e RAM-hard, recomendado em cenários novos.
- `scrypt`: também resistente a GPU, mas menos usado em aplicações web comuns.

---

## 💡 Boas práticas de armazenamento
1. **Armazene apenas o hash**, nunca a senha plana.
2. **Use TLS** ao transmitir senhas.
3. **Não repita salts**: `bcrypt.gensalt()` gera salts únicos.
4. **Não compare hashes manualmente**: use `bcrypt.checkpw`.
5. **Tenha uma política de expiração** e permita reset seguro de senhas.
6. **Monitore tentativas de login** para detectar brute-force.
7. **Use 12+ rounds** como base e ajuste para seu ambiente.
8. **Atualize hashes gradualmente** quando o custo aumentar.

---

## 🧠 Observações finais
O `bcrypt` é uma excelente escolha para hashing de senhas, mas não é a única ferramenta. Um guia completo deve incluir custo, truncamento, normalização, rehash e o contexto correto de uso.
