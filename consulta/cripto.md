# 🔐 Guia Supremo de Criptografia e Identificação

Este guia cobre os cenários mais comuns de proteção de dados em Python, desde a criptografia local até a comunicação segura e a verificação de integridade.

---

## 📑 Sumário
1. [🛠️ Instalação](#️-instalação)
2. [🔒 Criptografia Simétrica (Fernet)](#-criptografia-simétrica-fernet)
3. [🔐 Criptografia Simétrica Avançada (AES-GCM)](#-criptografia-simétrica-avançada-aes-gcm)
4. [🔑 Proteção por Senha (Derivação de Chave)](#-proteção-por-senha-derivação-de-chave)
5. [🗝️ Criptografia Assimétrica (RSA)](#️-criptografia-assimétrica-rsa)
6. [🔰 Chaves modernas (Ed25519 / X25519)](#️-chaves-modernas-ed25519--x25519)
7. [🧾 Certificados TLS e X.509](#️-certificados-tls-e-x509)
8. [✍️ Assinaturas Digitais e HMAC](#️-assinaturas-digitais-e-hmac)
9. [🧪 Criptografia Híbrida](#️-criptografia-híbrida)
10. [🆔 UUID - Identificadores Únicos](#-uuid---identificadores-únicos)
11. [🧮 Hash de Arquivos](#-hash-de-arquivos)
12. [🧱 Gerenciamento de Chaves e Boas Práticas](#-gerenciamento-de-chaves-e-boas-práticas)

---

## 🛠️ Instalação
A biblioteca padrão da indústria em Python para criptografia moderna é a `cryptography`.

```bash
pip install cryptography
```

Também use `bcrypt`, `argon2-cffi` ou `passlib` para hashing de senhas.

---

## 🔒 Criptografia Simétrica (Fernet)
**Caso de uso:** proteger arquivos ou dados locais de leitura não autorizada.

Fernet é uma API de alto nível que combina criptografia e autenticação em um único fluxo seguro.

```python
from cryptography.fernet import Fernet

# 1. Gerar e salvar uma chave (faça isso apenas uma vez)
chave = Fernet.generate_key()
with open("chave.key", "wb") as key_file:
    key_file.write(chave)

# 2. Carregar a chave e criptografar dados
f = Fernet(chave)
mensagem = "Dados sensíveis do checklist".encode()
token = f.encrypt(mensagem)

# 3. Descriptografar
decifrado = f.decrypt(token)
print(decifrado.decode())
```

> Nota: Fernet usa AES em CBC com HMAC SHA256 e já cuida de nonce/IV e autenticidade.

---

## 🔐 Criptografia Simétrica Avançada (AES-GCM)
**Caso de uso:** necessidades de desempenho ou interoperabilidade, quando você precisa de autenticação de dados integrada.

AES-GCM fornece confidencialidade e integridade em um único algoritmo.

```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os

key = AESGCM.generate_key(bit_length=256)
aesgcm = AESGCM(key)
nonce = os.urandom(12)  # Nunca reutilize o mesmo nonce com a mesma chave

texto = b"dados muito secretos"
associated_data = b"cabecalho-autenticado"
ct = aesgcm.encrypt(nonce, texto, associated_data)

plaintext = aesgcm.decrypt(nonce, ct, associated_data)
print(plaintext)
```

> Importante: 
> - use um nonce/IV único por mensagem
> - associe dados extras (AAD) quando precisar proteger cabeçalhos ou metadados
> - não reutilize chave + nonce

---

## 🔑 Proteção por Senha (Derivação de Chave)
**Caso de uso:** permitir que o usuário abra dados com uma senha memorável.

A senha nunca deve ser usada diretamente como chave criptográfica; use um KDF seguro.

```python
import base64
import os
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.fernet import Fernet

password = b"minha_senha_secreta"
salt = os.urandom(16)

kdf = PBKDF2HMAC(
    algorithm=hashes.SHA256(),
    length=32,
    salt=salt,
    iterations=480000,
)

key = base64.urlsafe_b64encode(kdf.derive(password))
f = Fernet(key)
```

> Dica: armazene o `salt` junto com os dados criptografados para permitir a derivação posterior.

---

## 🗝️ Criptografia Assimétrica (RSA)
**Caso de uso:** enviar dados a terceiros sem compartilhar a chave secreta.

RSA funciona bem para trocar chaves ou pequenos blocos de dados.

```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes

private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
public_key = private_key.public_key()

mensagem = b"Mensagem secreta para o terceiro"
cifrado = public_key.encrypt(
    mensagem,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None,
    ),
)

original = private_key.decrypt(
    cifrado,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None,
    ),
)
print(original)
```

### Serialização de chaves RSA
```python
from cryptography.hazmat.primitives import serialization

pem_priv = private_key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.PKCS8,
    encryption_algorithm=serialization.BestAvailableEncryption(b"senha-segura"),
)

pem_pub = public_key.public_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PublicFormat.SubjectPublicKeyInfo,
)
```

> Use `BestAvailableEncryption` apenas para armazenar a chave privada em disco de forma segura.

---

## ✍️ Assinaturas Digitais e HMAC
### Assinaturas RSA (confidencialidade + autenticidade)
**Caso de uso:** provar que uma mensagem veio do remetente e não foi alterada.

```python
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.primitives import hashes

mensagem = b"dados a assinar"
signature = private_key.sign(
    mensagem,
    padding.PSS(
        mgf=padding.MGF1(hashes.SHA256()),
        salt_length=padding.PSS.MAX_LENGTH,
    ),
    hashes.SHA256(),
)

public_key.verify(
    signature,
    mensagem,
    padding.PSS(
        mgf=padding.MGF1(hashes.SHA256()),
        salt_length=padding.PSS.MAX_LENGTH,
    ),
    hashes.SHA256(),
)
```

### HMAC (autenticação de mensagem simétrica)
**Caso de uso:** verificar integridade em um canal onde ambos os lados compartilham a mesma chave.

```python
from cryptography.hazmat.primitives import hmac, hashes

chave = b"chave compartilhada segreda"
h = hmac.HMAC(chave, hashes.SHA256())
h.update(b"mensagem importante")
mac = h.finalize()

verificador = hmac.HMAC(chave, hashes.SHA256())
verificador.update(b"mensagem importante")
verificador.verify(mac)
```

> Lembre-se: HMAC não autentica o remetente individualmente, apenas garante que a mensagem não foi alterada por quem conhece a chave.

---

## 🧪 Criptografia Híbrida
**Caso de uso:** combinar performance de criptografia simétrica com a segurança de troca de chaves assimétricas.

1. Gere uma chave AES para o conteúdo.
2. Criptografe o conteúdo com AES-GCM.
3. Criptografe a chave AES com RSA.

```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os

# Chave simétrica temporária
aes_key = AESGCM.generate_key(bit_length=256)
aesgcm = AESGCM(aes_key)
nonce = os.urandom(12)
plaintext = b"dados secretos grandes"
ct = aesgcm.encrypt(nonce, plaintext, None)

# Encriptar a chave AES com RSA
enc_aes_key = public_key.encrypt(
    aes_key,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None,
    ),
)
```

> Esse padrão é usado em muitos protocolos seguros, incluindo TLS.

---

## 🔰 Chaves modernas (Ed25519 / X25519)
**Caso de uso:** troca de chaves e assinaturas com algoritmos mais leves e seguros.

Ed25519 oferece assinaturas digitais rápidas e seguras, enquanto X25519 é uma excelente escolha para troca de chaves Diffie-Hellman.

```python
from cryptography.hazmat.primitives.asymmetric import ed25519, x25519

# Ed25519 para assinaturas
gera_assinatura = ed25519.Ed25519PrivateKey.generate()
pub_assinatura = gera_assinatura.public_key()

mensagem = b"dados a assinar"
signature = gera_assinatura.sign(mensagem)
pub_assinatura.verify(signature, mensagem)

# X25519 para troca de chaves
a_private = x25519.X25519PrivateKey.generate()
a_public = a_private.public_key()
b_private = x25519.X25519PrivateKey.generate()
b_public = b_private.public_key()

shared_a = a_private.exchange(b_public)
shared_b = b_private.exchange(a_public)
assert shared_a == shared_b
```

> Use Ed25519/X25519 quando possível, pois eles fornecem segurança moderna com menos erros de uso do que RSA.

---

## 🧾 Certificados TLS e X.509
**Caso de uso:** autenticar servidores, clientes e garantir canais HTTPS seguros.

O `cryptography` permite criar e assinar certificados autoassinados para testes e gerar pedidos de assinatura (CSRs) para CA.

```python
from cryptography import x509
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.x509.oid import NameOID
import datetime

private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
subject = issuer = x509.Name([
    x509.NameAttribute(NameOID.COUNTRY_NAME, u"BR"),
    x509.NameAttribute(NameOID.STATE_OR_PROVINCE_NAME, u"SP"),
    x509.NameAttribute(NameOID.LOCALITY_NAME, u"São Paulo"),
    x509.NameAttribute(NameOID.ORGANIZATION_NAME, u"Minha Empresa"),
    x509.NameAttribute(NameOID.COMMON_NAME, u"meuservidor.local"),
])

cert = (
    x509.CertificateBuilder()
    .subject_name(subject)
    .issuer_name(issuer)
    .public_key(private_key.public_key())
    .serial_number(x509.random_serial_number())
    .not_valid_before(datetime.datetime.utcnow())
    .not_valid_after(datetime.datetime.utcnow() + datetime.timedelta(days=365))
    .add_extension(x509.SubjectAlternativeName([x509.DNSName(u"localhost")]), critical=False)
    .sign(private_key, hashes.SHA256())
)

with open("cert.pem", "wb") as f:
    f.write(cert.public_bytes(serialization.Encoding.PEM))

with open("key.pem", "wb") as f:
    f.write(private_key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.TraditionalOpenSSL,
        encryption_algorithm=serialization.NoEncryption(),
    ))
```

> Em produção, prefira certificados emitidos por uma CA confiável e nunca use chaves autoassinadas fora de ambientes de teste.

---

## 🆔 UUID - Identificadores Únicos
UUIDs garantem que um ID não se repita e são úteis para arquivos, tokens e nomes de entidades.

```python
import uuid

id_aleatorio = uuid.uuid4()
print(id_aleatorio)

id_tempo = uuid.uuid1()
id_nome = uuid.uuid5(uuid.NAMESPACE_DNS, "python.org")
```

> Use `uuid4()` para a maioria dos casos. Evite `uuid1()` se não quiser dados de hardware/tempo expostos.

---

## 🧮 Hash de Arquivos
**Caso de uso:** verificar integridade ou detectar alterações.

```python
import hashlib

def calcular_hash_arquivo(caminho, algoritmo="sha256"):
    hash_obj = hashlib.new(algoritmo)
    with open(caminho, "rb") as f:
        for bloco in iter(lambda: f.read(4096), b""):
            hash_obj.update(bloco)
    return hash_obj.hexdigest()

print(calcular_hash_arquivo("meu_projeto.zip", "sha256"))
```

### Hash de senhas
Para senhas, use hashing adaptativo como bcrypt, Argon2 ou scrypt, não SHA256 simples.

```python
import bcrypt

senha = b"minha-senha"
hash_senha = bcrypt.hashpw(senha, bcrypt.gensalt())
assert bcrypt.checkpw(senha, hash_senha)
```

---

## 🧱 Gerenciamento de Chaves e Boas Práticas
1. **Nunca hardcode chaves ou segredos em código-fonte.** Use variáveis de ambiente, cofres ou sistemas de segredos.
   - Exemplos: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager.
2. **Use chaves diferentes para propósitos diferentes.** Não use a mesma chave para criptografia e HMAC.
3. **Proteja chaves privadas.** Faça backup seguro e limite acesso.
4. **Troque chaves regularmente** quando possível (rotacionamento).
5. **Valide entradas** antes de descriptografar ou assinar.
6. **Prefira bibliotecas de alto nível** como Fernet quando não precisar de controle absoluto sobre os parâmetros criptográficos.
7. **Não reinvente algoritmos criptográficos.** Confie em `cryptography` e algoritmos bem estudados.
8. **Use parâmetros fortes:** RSA 2048+ ou 3072+, AES-256, salt aleatório, 100k+ iterações para KDFs.
9. **Proteja contra ataques de tempo:** use verificações constantes quando comparar MACs ou hashes.
10. **Considere o ciclo de vida dos segredos:** rotação, expiração, revogação e destruição segura.
11. **Documente o fluxo de criptografia** no projeto para que seu time saiba como gerar, armazenar e usar chaves corretamente.

---

## 🧠 Observações finais
Este documento agora cobre tanto os casos práticos quanto temas avançados necessários para usar criptografia com segurança em Python. Ainda assim, criptografia em produção exige revisão especializada e testes de segurança adicionais.
