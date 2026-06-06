# 📝 Guia Completo de Markdown

Markdown é uma linguagem de marcação leve usada para formatar texto de maneira simples e legível, amplamente utilizada em arquivos README, documentações e fóruns.

---

## 📑 Sumário
1. [🏗️ Títulos](#-títulos)
2. [🧱 Títulos Setext](#-títulos-setext)
3. [✍️ Formatação de Texto](#️-formatação-de-texto)
4. [📋 Listas](#-listas)
5. [🔗 Links e Imagens](#-links-e-imagens)
6. [💻 Código](#-código)
7. [📊 Tabelas](#-tabelas)
8. [📎 Outros Elementos](#-outros-elementos)
9. [🌐 Markdown Flavors](#-markdown-flavors)
10. [✨ Sintaxe Avançada](#-sintaxe-avançada)

---

## 🏗️ Títulos
Use o caractere `#` seguido de um espaço para criar títulos.
```markdown
# Título 1 (H1)
## Título 2 (H2)
### Título 3 (H3)
#### Título 4 (H4)
##### Título 5 (H5)
###### Título 6 (H6)
```

---

## 🧱 Títulos Setext
Os títulos Setext usam sublinhados com `=` ou `-`.
```markdown
Título 1
=======

Título 2
-------
```

---

## ✍️ Formatação de Texto

| Estilo | Sintaxe | Resultado |
| :--- | :--- | :--- |
| **Negrito** | `**texto**` ou `__texto__` | **Negrito** |
| *Itálico* | `*texto*` ou `_texto_` | *Itálico* |
| ***Negrito e Itálico*** | `***texto***` ou `___texto___` | ***Negrito e Itálico*** |
| ~~Tachado~~ | `~~texto~~` | ~~Tachado~~ |
| `Código em linha` | `` `texto` `` | `Código em linha` |

---

## 📋 Listas

### Listas Não Ordenadas
Use `-`, `*` ou `+`.
```markdown
- Item 1
- Item 2
  - Subitem 2.1
```

### Listas Ordenadas
Use números seguidos de ponto.
```markdown
1. Primeiro item
2. Segundo item
```

### Listas de Tarefas (Task Lists)
```markdown
- [x] Tarefa concluída
- [ ] Tarefa pendente
```

---

## 🔗 Links e Imagens

### Links Inline
`[Texto do Link](URL "Título opcional")`
Exemplo: [Google](https://www.google.com)

### Links de Referência
```markdown
[Google][google]

[google]: https://www.google.com "Google"
```

### Autolinks
`<https://www.google.com>` resulta em <https://www.google.com>

### Imagens Inline
`![Texto alternativo](URL da imagem "Título opcional")`
Exemplo:
`![Logo Python](https://www.python.org/static/community_logos/python-logo.png)`

### Imagens de Referência
```markdown
![Logo Python][python-logo]

[python-logo]: https://www.python.org/static/community_logos/python-logo.png "Python"
```

---

## 💻 Código

### Código em Linha
Use crases simples: `` `texto` ``.
Exemplo: O comando `pip install` é usado para instalar pacotes.

### Blocos de Código Fenced
Use três crases (```) ou `~~~` e, opcionalmente, o nome da linguagem para realce de sintaxe.

```python
def hello():
    print("Olá, Mundo!")
```

~~~
# Também funciona em muitos renderizadores
print("Olá, Mundo!")
~~~

### Blocos de Código Indentados
Também é possível criar blocos de código usando indentação de quatro espaços.

    def hello():
        print("Olá, Mundo!")

---

## 📊 Tabelas
Use canos `|` para separar colunas e hifens `-` para a linha de cabeçalho. Use `:` para alinhar.

```markdown
| Alinhado à Esquerda | Centralizado | Alinhado à Direita |
| :--- | :---: | ---: |
| Dado 1 | Dado 2 | Dado 3 |
```

---

## 📎 Outros Elementos

### Citações (Blockquotes)
Use `>` antes do texto.
> "A simplicidade é o último grau de sofisticação." - Leonardo da Vinci

Aninhamento:
```markdown
> Citação externa
> > Citação interna
```

### Linha Horizontal
Use três ou mais hifens, asteriscos ou underscores.
`---`

### Quebra de Linha
Para forçar uma quebra de linha, termine uma linha com dois ou mais espaços e aperte Enter, ou use a tag `<br>`.

### Escape de Caracteres
Use `\` para escapar caracteres especiais.
```markdown
\*não itálico\* -> *não itálico*
\# não é título
```

### HTML e Comentários
Markdown geralmente aceita HTML embutido, mas o suporte varia por renderizador. Em plataformas como GitHub e MkDocs, muitos elementos HTML funcionam, mas em outros sites pode ser bloqueado por segurança.
```markdown
<span style="color: red;">texto em vermelho</span>
```
Comentários HTML:
```html
<!-- Este é um comentário invisível -->
```

---

## 🌐 Markdown Flavors
Markdown tem vários sabores e extensões, e nem todos os recursos funcionam em todas as plataformas.

- **CommonMark**: padrão moderno e consistente usado como base para muitos renderizadores.
- **GitHub Flavored Markdown (GFM)**: suporta tabelas, task lists, autolinks, strikethrough, footnotes limitados e sintaxe de referência.
- **Markdown Extra**: adiciona tabelas, notas de rodapé, definidores e atributos.
- **MkDocs / MkDocs Material**: adiciona suporte a admonitions, emojis e outras extensões baseadas em Markdown.

### Diferenças comuns
- `~~tachado~~` funciona no GitHub, mas não em Markdown básico puro.
- Notas de rodapé podem não ser suportadas em todos os renderizadores.
- `[`shortcode`]` e outros recursos específicos podem ser exclusivos de ferramentas estáticas.

---

## ✨ Sintaxe Avançada

### Notas de Rodapé (GitHub Flavored Markdown)
```markdown
Aqui está uma nota de rodapé.[^1]

[^1]: Esta é a nota de rodapé.
```

### Abreviações (GitHub Flavored Markdown)
```markdown
*[HTML]: HyperText Markup Language
```

### Dicas Finais
- No GitHub, você pode usar emojis como `:rocket:` 🚀.
- Markdown aceita a maioria das tags HTML para formatações mais avançadas, mas suporte varia por renderizador.
- Use linhas em branco antes e depois de listas, tabelas e blocos de código para evitar problemas de renderização.
- Para listas aninhadas, indente com 2 ou 4 espaços e use uma linha em branco antes da lista de subitens quando necessário.
- `~~~` e `` ``` `` são equivalentes na maioria dos renderizadores de bloco de código fenced.
- Autolinks como `<https://example.com>` funcionam no GitHub e em CommonMark.
- Nem todos os renderizadores suportam todas as extensões do Markdown. Verifique o ambiente (GitHub, GitLab, MkDocs, Jekyll, etc.).
- Quando precisar de recursos avançados específicos, prefira documentar o sabor/ambiente usado no projeto.
