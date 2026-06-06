# 📂 Guia de Referência Pathlib

O módulo `pathlib` oferece uma API orientada a objetos para manipular caminhos de arquivo de forma clara, segura e multiplataforma. Ele combina a parte funcional de `os.path` com operações de E/S nativas do sistema.

---

## 📑 Sumário
1. [📦 Criando objetos Path](#-criando-objetos-path)
2. [🔗 Manipulação e componentes de caminho](#-manipulação-e-componentes-de-caminho)
3. [🔍 Validações e inspeções](#-validações-e-inspeções)
4. [🛠️ Ações no sistema de arquivos](#️-ações-no-sistema-de-arquivos)
5. [📄 Leitura e escrita de conteúdo](#-leitura-e-escrita-de-conteúdo)
6. [📂 Navegação e busca](#-navegação-e-busca)
7. [⚠️ Boas práticas e considerações](#️-boas-práticas-e-considerações)
8. [💡 Dicas avançadas](#-dicas-avançadas)

---

## 📦 Criando objetos Path
```python
from pathlib import Path

# Caminho relativo ao diretório atual
base_path = Path(".")

# Caminho absoluto
abs_path = Path("/home/user/documentos")

# Caminho para o usuário atual
home = Path.home()

# Caminho do diretório de trabalho atual
cwd = Path.cwd()

# Caminho em Windows (PureWindowsPath também existe para manipulação sem E/S)
win_path = Path(r"C:\Users\user\docs")
```

### `PurePath` vs `Path`
- `PurePath` trata apenas da manipulação de strings de caminhos.
- `Path` adiciona operações de E/S específicas do sistema.

```python
from pathlib import PurePosixPath, PureWindowsPath

pure_posix = PurePosixPath("/tmp/file.txt")
pure_win = PureWindowsPath(r"C:\temp\file.txt")
```

---

## 🔗 Manipulação e componentes de caminho
```python
caminho = Path("projeto/src/main.py")
print(caminho.name)      # main.py
print(caminho.stem)      # main
print(caminho.suffix)    # .py
print(caminho.suffixes)  # ['.py']
print(caminho.parent)    # projeto/src
print(caminho.parents[0]) # projeto/src
print(caminho.parents[1]) # projeto
print(caminho.parts)     # ('projeto', 'src', 'main.py')
print(caminho.drive)     # '' ou 'C:' no Windows
print(caminho.root)      # '' ou '/' no POSIX
print(caminho.anchor)    # combinação de drive+root
print(caminho.is_absolute())
```

### Construção e alteração de caminhos
```python
novo_caminho = Path("projeto") / "src" / "main.py"
mesmo_caminho = Path("projeto").joinpath("src", "main.py")

alterado = caminho.with_name("app.py")
alt_sufixo = caminho.with_suffix(".txt")
relativo = caminho.relative_to("projeto")
print(relativo)  # src/main.py
```

### Normalização e resolução
```python
print((Path(".") / ".." / "arquivo.txt").resolve())
print(Path("arquivo.txt").absolute())
```

---

## 🔍 Validações e inspeções
```python
arquivo = Path("dados.csv")
print(arquivo.exists())      # True se existe
print(arquivo.is_file())      # True se é arquivo
print(arquivo.is_dir())       # True se é diretório
print(arquivo.is_symlink())   # True se é link simbólico
print(arquivo.stat())         # Estatísticas do arquivo
print(arquivo.lstat())        # Estatísticas sem seguir symlink

try:
    print(arquivo.owner())
    print(arquivo.group())
except Exception:
    pass  # Nem todos os sistemas suportam owner/group
```

### Comparação de caminhos
```python
a = Path("./arquivo.txt")
b = Path("/home/user/projeto/arquivo.txt")
print(a.resolve() == b.resolve())
print(a.samefile(b))
```

---

## 🛠️ Ações no sistema de arquivos
```python
pasta = Path("nova_pasta")
pasta.mkdir(parents=True, exist_ok=True)

arquivo = pasta / "info.txt"
arquivo.touch(exist_ok=True)
arquivo.write_text("conteúdo\n", encoding="utf-8")

arquivo.rename(pasta / "detalhes.txt")
arquivo.replace(pasta / "detalhes.txt")

# Exclusão
if (pasta / "detalhes.txt").exists():
    (pasta / "detalhes.txt").unlink()

# Remover diretório vazio
(Path("pasta_vazia")).rmdir()
```

### Trabalhando com links simbólicos
```python
target = Path("/tmp/original.txt")
link = Path("/tmp/link.txt")
link.symlink_to(target)
print(link.is_symlink())
print(link.readlink())
```

---

## 📄 Leitura e escrita de conteúdo
```python
arquivo = Path("README.md")
texto = arquivo.read_text(encoding="utf-8")
arquivo.write_text("# Meu Projeto\n", encoding="utf-8")

bytes_data = arquivo.read_bytes()
arquivo.write_bytes(bytes_data)
```

### Uso com `open()` e `with`
```python
with arquivo.open("r", encoding="utf-8") as f:
    for linha in f:
        print(linha.strip())
```

### Salvamento seguro usando um arquivo temporário
```python
destino = Path("dados.json")
temp = destino.with_suffix(".tmp")
temp.write_text(json_text, encoding="utf-8")
temp.replace(destino)
```

---

## 📂 Navegação e busca
```python
diretorio = Path("src")
for item in diretorio.iterdir():
    print(item)

for python_file in diretorio.glob("*.py"):
    print(python_file)

for all_py in diretorio.rglob("*.py"):
    print(all_py)
```

### Correspondência de padrões
```python
p = Path("src/app/main.py")
print(p.match("*.py"))
print(p.match("src/**/*.py"))
```

### Conversão para string multiplataforma
```python
print(p.as_posix())
print(Path("C:\\temp\\file.txt").as_posix())
```

---

## 🔄 Interoperabilidade com os.path
`pathlib` e `os.path` podem ser usados juntos, mas o ideal é manter `Path` como fonte primária.

```python
from pathlib import Path
import os

path = Path("src/app/main.py")
print(os.path.basename(path))
print(os.path.dirname(path))

str_path = str(path)
joined = os.path.join(str_path, "..", "out")
print(joined)

path_from_os = Path(os.path.join("src", "app", "main.py"))
```

### Recomendações
- Use `Path` para operações modernas de caminho e `os.path` somente se precisar de compatibilidade com APIs legadas.
- Converta `Path` em string com `str(path)` antes de chamar funções `os.path`.
- Se precisar de um caminho POSIX normalizado em string, use `path.as_posix()`.
- Para APIs que exigem bytes, use `path.as_posix().encode("utf-8")` ou `path.__fspath__()`.


## ⚠️ Boas práticas e considerações
- Use sempre `Path` para operações de caminho em vez de concatenar strings.
- Evite manipular `Path` como string; prefira métodos como `with_name`, `with_suffix` e `relative_to`.
- Valide `exists()` antes de `unlink()` ou `rmdir()` para evitar erros.
- Use `mkdir(parents=True, exist_ok=True)` para criar hierarquias de diretórios de forma segura.
- Para arquivos binários use `read_bytes()` / `write_bytes()`; para texto use `read_text()` / `write_text()`.
- Em deploys multiplataforma, evite misturar `os.path` e `pathlib` sem necessidade.

---

## 💡 Dicas avançadas
- `Path.home()` e `Path.cwd()` são preferíveis a `os.path.expanduser('~')` e `os.getcwd()`.
- `Path.as_uri()` gera URI de caminhos locais no formato `file:///...`.
- Use `Path.match()` para filtros rápidos, mas prefira `glob()` ou `rglob()` para listagens completas.
- Se precisar apenas manipular strings de caminhos sem E/S, use `PurePosixPath` ou `PureWindowsPath`.
- Nenhum método de `Path` é thread-safe por si só; proteja acesso concorrente quando necessário.

---

## ✅ Checklist de pathlib
- [ ] Substituiu concatenações de strings pelo operador `/`?
- [ ] Usou `.resolve()` para obter caminhos absolutos e tratar links simbólicos?
- [ ] Garantiu a criação de diretórios com `mkdir(parents=True, exist_ok=True)`?
- [ ] Validou a existência do arquivo antes de operações destrutivas (`unlink`)?
- [ ] Utilizou `read_text`/`write_text` para manipulações simples de arquivos?
- [ ] Preferiu `Path.home()` e `Path.cwd()` em vez de funções do módulo `os`?
