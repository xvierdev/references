# 🧭 Bíblia do Git: O Guia Supremo

O Git é a ferramenta mais importante na caixa de ferramentas de um desenvolvedor. Este guia cobre desde o básico essencial até as manobras avançadas de recuperação de histórico e produtividade extrema.

---

## 📑 Sumário
1. [⚙️ Configuração e Produtividade](#️-configuração-e-produtividade)
2. [🏗️ Fluxo de Trabalho e Staging Parcial](#️-fluxo-de-trabalho-e-staging-parcial)
3. [🌿 Branches e Estratégias de Merge](#-branches-e-estratégias-de-merge)
4. [🔄 Rebase Interativo e Limpeza de Histórico](#-rebase-interativo-e-limpeza-de-histórico)
5. [🧹 Desfazendo Mudanças (Reset, Revert, Restore)](#-desfazendo-mudanças-reset-revert-restore)
6. [🛠️ Ferramentas de Resgate (Reflog, Bisect, Cherry-pick)](#️-ferramentas-de-resgate-reflog-bisect-cherry-pick)
7. [📦 Submodules, Worktrees e LFS](#-submodules-worktrees-e-lfs)
8. [⚓ Hooks e Automação](#-hooks-e-automação)
9. [📋 Convenções de Commit (Conventional Commits)](#-convenções-de-commit-conventional-commits)
10. [🚀 Dicas de Ouro e Aliases](#-dicas-de-ouro-e-aliases)
11. [✅ Checklist de Git Profissional](#-checklist-de-git-profissional)

---

## 1. ⚙️ Configuração e Produtividade

### Identidade e Editor
```bash
git config --global user.name "Seu Nome"
git config --global user.email voce@exemplo.com
git config --global core.editor "code --wait" # Usa o VS Code como editor
```

### Comportamento Global Recomendado
```bash
git config --global pull.rebase true      # Pull faz rebase por padrão (histórico limpo)
git config --global fetch.prune true      # Limpa referências a branches remotas deletadas
git config --global rebase.autosquash true # Facilita limpeza automática de commits fixup
git config --global init.defaultBranch main # Define 'main' como padrão
```

---

## 2. 🏗️ Fluxo de Trabalho e Staging Parcial

### O Comum (The Basics)
```bash
git init
git clone <url>
git status
git add <arquivo>
git commit -m "feat: descrição curta"
git push origin <branch>
```

### Staging Parcial (`add -p`) - O Segredo dos Commits Limpos
Não dê `git add .` se você alterou várias coisas diferentes. Use o modo interativo para escolher quais partes do arquivo entrarão no commit.
```bash
git add -p <arquivo>
```
*   `y`: adiciona este bloco (hunk).
*   `n`: não adiciona este bloco.
*   `s`: divide o bloco em partes menores.
*   `e`: edita o bloco manualmente.

### Comandos Modernos (Git 2.23+)
Substitua o confuso `checkout` por comandos focados:
- `git switch <branch>`: Mudar de branch.
- `git switch -c <nova-branch>`: Criar e mudar.
- `git restore <arquivo>`: Descartar mudanças no arquivo (unstage ou undo).

---

## 3. 🌿 Branches e Estratégias de Merge

### Ciclo de Vida Profissional
1. `git switch -c feature/nome-da-tarefa` (Criar branch)
2. Commits atômicos (vários pequenos commits)
3. `git fetch origin main` (Baixar novidades da main)
4. `git rebase origin/main` (Pôr seu trabalho no topo da main atual)
5. Resolver conflitos (se houver)
6. `git push origin feature/nome-da-tarefa -f` (Force push após rebase)
7. Abrir Pull Request.

### Merge vs Rebase vs Squash
- **Merge**: Preserva a história exata, mas cria "grafos espaguete".
- **Rebase**: Mantém uma linha do tempo perfeitamente linear.
- **Squash**: Transforma 10 commits de "trabalho em progresso" em 1 commit limpo na branch principal.

---

## 4. 🔄 Rebase Interativo e Limpeza de Histórico
O Rebase interativo é a sua "máquina do tempo" para organizar commits antes de enviá-los.

```bash
git rebase -i HEAD~5 # Abre editor com os últimos 5 commits
```

**Comandos principais no editor:**
- `pick`: Mantém o commit.
- `reword`: Muda apenas a mensagem.
- `edit`: Para o rebase para você alterar o conteúdo do commit.
- `squash`: Junta este commit com o anterior e pede uma nova mensagem.
- `fixup`: Junta este commit com o anterior, descartando a mensagem atual (ótimo para correções rápidas).
- `drop`: Deleta o commit.

---

## 📋 Convenções de Commit (Conventional Commits)
Um histórico legível por humanos e máquinas. Formato: `<tipo>[escopo]: <descrição>`

- **feat**: Uma nova funcionalidade.
- **fix**: Correção de um bug.
- **docs**: Alteração apenas na documentação.
- **style**: Formatação, ponto e vírgula (sem alteração de lógica).
- **refactor**: Mudança de código que não altera comportamento.
- **test**: Adição ou correção de testes.
- **chore**: Atualização de build, dependências, etc.

*Exemplo: `feat(auth): add login with google`*

---

## 5. 🧹 Desfazendo Mudanças (Reset, Revert, Restore)

### Reset (O bisturi)
- **--soft**: Desfaz o commit, mas mantém as mudanças na área de *staging* (prontas para novo commit).
- **--mixed** (padrão): Desfaz o commit e tira do *staging*, mas mantém no disco.
- **--hard**: **DELETA** tudo e volta ao estado do commit alvo. Cuidado!

### Revert (O seguro)
Cria um novo commit que desfaz exatamente o que o commit anterior fez. É a forma segura de desfazer mudanças em branches públicas/compartilhadas.

### Restore (O moderno)
- `git restore <file>`: Desfaz mudanças locais (volta ao estado do último commit).
- `git restore --staged <file>`: Tira o arquivo da área de staging.

---

## 6. 🛠️ Ferramentas de Resgate (Reflog, Bisect, Cherry-pick)

- **Reflog**: O log dos logs. Mostra todos os movimentos do `HEAD`. Se você deu um `reset --hard` por engano, o commit ainda está lá, e o `reflog` te mostra onde.
  ```bash
  git reflog
  git reset --hard <sha-do-reflog>
  ```
- **Bisect**: Busca binária para achar o commit que introduziu um bug via busca binária.
  ```bash
  git bisect start
  git bisect bad                 # Esta versão está ruim
  git bisect good <v1.0.0>       # Aquela versão estava boa
  # Git vai trocando de commits até achar o culpado
  ```
- **Cherry-pick**: Pega um commit de outra branch e aplica na atual.
  ```bash
  git cherry-pick <commit-hash>
  ```

---

## 7. 📦 Submodules, Worktrees e LFS
- **Submodules**: Repositórios dentro de repositórios.
- **Worktrees**: Tenha a mesma branch aberta em pastas diferentes para tarefas paralelas.
  ```bash
  git worktree add ../pasta-tarefa-x feature/x
  ```
- **LFS**: Gerencie arquivos binários gigantes (modelos de IA, vídeos) sem deixar o `.git` pesado.

---

## 8. ⚓ Hooks e Automação
Scripts que rodam antes/depois de eventos do Git. Use para garantir qualidade:
- **pre-commit**: Roda linters e testes antes de deixar commitar.
- **commit-msg**: Garante que a mensagem segue o padrão (Conventional Commits).

---

## 9. 🚀 Dicas de Ouro e Aliases
Adicione isso ao seu `~/.gitconfig` para ser 2x mais rápido:

```ini
[alias]
  st = status
  co = checkout
  br = branch
  ci = commit
  lg = log --oneline --graph --decorate --all
  unstage = restore --staged
  last = log -1 HEAD
  amend = commit --amend --no-edit
```

---

## 10. ✅ Checklist de Git Profissional

- [ ] Identidade global (`user.name`, `user.email`) configurada.
- [ ] Branches nomeadas com prefixos (`feat/`, `fix/`, `docs/`).
- [ ] Commits seguem o padrão **Conventional Commits**.
- [ ] **Staging parcial** (`add -p`) usado para manter commits atômicos.
- [ ] **Rebase** usado em vez de Merge para manter histórico linear.
- [ ] **Squash/Fixup** realizados antes de finalizar o Pull Request.
- [ ] **Reflog** e **Bisect** conhecidos para emergências e bugs.
- [ ] **Git Hooks** ativos para garantir linting e testes.
- [ ] **LFS** configurado para arquivos binários pesados.
- [ ] Branches remotas deletadas são limpas periodicamente (`fetch --prune`).

---

## Recursos
- Git Pro Book: https://git-scm.com/book/en/v2
- Pro Git (book): https://git-scm.com/book/
