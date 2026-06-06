# 🕸️ Bíblia do Reflex: Guia Definitivo para Aplicações Web Full-Stack em Python

O **Reflex** permite construir aplicações web completas usando apenas Python. Ele compila o frontend para React/Next.js e usa FastAPI no backend, unindo estado, UI, backend e deploy em um único fluxo.

---

## 📑 Sumário
1. [🛠️ Instalação e Inicialização](#️-instalação-e-inicialização)
2. [🏗️ Estrutura de Projeto Reflex](#️-estrutura-de-projeto-reflex)
3. [⚙️ Configuração de `rxconfig.py`](#️-configuração-de-rxconfigpy)
4. [🧠 Estado e Fluxo de Dados (Avançado)](#️-estado-e-fluxo-de-dados-avançado)
5. [🎨 Componentes e Renderização Dinâmica](#️-componentes-e-renderização-dinâmica)
6. [🎨 Estilização e Temas](#️-estilização-e-temas)
7. [📊 Visualização de Dados e Gráficos](#️-visualização-de-dados-e-gráficos)
8. [📝 Formulários e Notificações (Toasts)](#️-formulários-e-notificações-toasts)
9. [🔐 Autenticação, Cookies e Segurança](#️-autenticação-cookies-e-segurança)
10. [🗄️ Banco de Dados e Models](#️-banco-de-dados-e-models)
11. [📡 Backend, API Routes e WebHooks](#️-backend-api-routes-e-webhooks)
12. [🕸️ WebSockets e Tempo Real](#️-websockets-e-tempo-real)
13. [🖼️ Assets e Arquivos Estáticos](#️-assets-e-arquivos-estáticos)
14. [🧪 Testes, Debug e Logs](#️-testes-debug-e-logs)
15. [🚀 Build, Export e Deploy](#️-build-export-e-deploy)
16. [🛡️ Produção e Boas Práticas](#️-produção-e-boas-práticas)
17. [✅ Checklist da Bíblia Reflex](#️-checklist-da-bíblia-reflex)

---

## 🛠️ Instalação e Inicialização
```bash
pip install reflex
mkdir meu_app && cd meu_app
reflex init
```

O comando `reflex init` cria a estrutura básica, inclui `rxconfig.py`, `assets/`, `public/` e o pacote do app.

---

## 🏗️ Estrutura de Projeto Reflex
Um projeto Reflex completo normalmente tem:

- `assets/`: arquivos estáticos usados no frontend.
- `public/`: arquivos públicos servidos diretamente (favicon, robots.txt, etc.).
- `rxconfig.py`: configurações do app, deploy e build.
- `meu_app/__init__.py`: inicialização do app.
- `meu_app/pages.py`: páginas do site.
- `meu_app/state.py`: classes de estado (State).
- `meu_app/components/`: componentes customizados.
- `meu_app/backend.py`: lógica do backend e rotas custom.
- `meu_app/models.py`: modelos do banco de dados.

Exemplo de estrutura:

```
meu_app/
├── __init__.py
├── backend.py
├── components.py
├── models.py
├── pages.py
├── state.py
assets/
public/
rxconfig.py
README.md
```

---

## ⚙️ Configuração de `rxconfig.py`
O `rxconfig.py` define nome do app, pacotes, build e deploy.

```python
from reflex import Config

config = Config(
    app_name="meu_app",
    db_url="sqlite:///reflex.db",
    env="dev",
    frontend_url="http://localhost:3000",
    api_url="http://localhost:8000",
    port=3000,
)
```

### Parâmetros importantes
- `app_name`: nome do projeto.
- `db_url`: string de conexão do banco.
- `env`: `dev` ou `prod`.
- `frontend_url` / `api_url`: URLs de frontend e backend.
- `port`: porta do servidor local.

---

## 🧠 Estado e Fluxo de Dados (Avançado)
O `State` é o coração do Reflex. Para aplicações profissionais, use padrões de escalabilidade.

### Variáveis Computadas (@rx.var)
Use `@rx.var` para valores que dependem de outros estados. Elas são recalculadas apenas quando necessário e não podem ser alteradas diretamente.

```python
class State(rx.State):
    items: list[int] = [1, 2, 3]

    @rx.var
    def total_items(self) -> int:
        return len(self.items)
```

### Substates (Estados Fragmentados)
Não coloque tudo em um único `State`. Divida a lógica em classes menores para melhorar a performance e organização.

```python
class AuthState(rx.State):
    user: str = ""

class CartState(rx.State):
    items: list[str] = []

class MainState(rx.State):
    # O MainState pode acessar sub-estados se necessário
    pass
```

### Ações do State
- Funções do state são chamadas por eventos do frontend.
- Executam no servidor e atualizam o estado automaticamente.
- Podem retornar `rx.redirect(...)` para navegação.

```python
    def go_home(self):
        return rx.redirect("/")

    def login(self):
        self.logged_in = True
        return rx.redirect("/dashboard")
```

### Ciclo de vida
- `on_mount`: usado para inicializar dados quando uma página é carregada.
- `@rx.on_mount`: registra ações que rodam ao montar a página.

```python
    @rx.on_mount
    def init(self):
        self.count = 10
```

---

## 🎨 Componentes e Renderização Dinâmica

### Renderização Condicional (rx.cond)
Não use `if/else` tradicional de Python dentro do retorno da UI. Use `rx.cond`.

```python
rx.cond(
    State.logged_in,
    rx.text("Bem-vindo de volta!"),
    rx.button("Fazer Login", on_click=State.login)
)
```

### Iteração de Listas (rx.foreach)
Para exibir listas de dados, use `rx.foreach`.

```python
def render_item(item: str):
    return rx.list_item(item)

rx.list(
    rx.foreach(
        State.items,
        render_item
    )
)
```

### Páginas e Layouts
Cada página é uma função que retorna componentes. Use um layout padrão para garantir consistência.

```python
def layout(page: rx.Component):
    return rx.vstack(
        rx.navbar(
            rx.link("Home", href="/"),
            rx.link("Sobre", href="/sobre"),
        ),
        page,
    )

def index():
    return layout(
        rx.vstack(
            rx.heading("Página Inicial"),
            rx.text("Esta é a página principal."),
        )
    )
```

### Componentes customizados
```python
def Card(title: str, content: str):
    return rx.box(
        rx.heading(title, size="lg"),
        rx.text(content),
        border="1px solid",
        padding="4",
        border_radius="md",
    )
```

### Navegação e links
```python
rx.link("Perfil", href="/profile")
```

### Páginas protegidas
```python
def protected_page():
    if not State.logged_in:
        return rx.redirect("/login")
    return rx.text("Área protegida")
```

---

## 🎨 Estilização e Temas
### Estilos inline
```python
rx.box(
    bg="gray.100",
    color="black",
    padding="6",
    border_radius="xl",
)
```

### Temas globais
Use temas para cores, fontes e espaçamento.

```python
app = rx.App(theme="light")
```

### CSS e assets
- Coloque CSS em `assets/`.
- Use `public/` para arquivos servidos diretamente.
- Importe imagens com `rx.image(src="/logo.png")`.

---

## 📊 Visualização de Dados e Gráficos
O Reflex possui integração nativa com o **Recharts**, permitindo criar dashboards poderosos.

```python
import reflex as rx

def dashboard_view():
    return rx.recharts.bar_chart(
        rx.recharts.bar(
            data_key="vendas", fill="#8884d8"
        ),
        rx.recharts.x_axis(data_key="mes"),
        rx.recharts.y_axis(),
        data=State.vendas_data,
        width="100%",
        height=300,
    )
```

---

## 📝 Formulários e Notificações (Toasts)
### Campos controlados
```python
rx.input(
    placeholder="Seu nome",
    value=State.name,
    on_change=State.set_name,
)
```

### Submissão de formulário
```python
def submit_form():
    # lógica de validação e envio
    return rx.redirect("/obrigado")

rx.button("Enviar", on_click=State.submit_form)
```

### Validação básica
- Verifique campos obrigatórios
- Normalize entradas
- Exiba mensagens de erro no estado

```python
class FormState(State):
    email: str = ""
    error: str = ""

    def submit(self):
        if "@" not in self.email:
            self.error = "Email inválido"
        else:
            self.error = ""
```

### Eventos e callbacks
- `on_click`
- `on_change`
- `on_focus`
- `on_blur`

Use eventos para chamar ações do state e atualizar a interface.

### Notificações (Toasts)
Use Toasts para dar feedback instantâneo ao usuário sem mudar de página.

```python
class FormState(State):
    def handle_submit(self):
        # Lógica de processamento
        return rx.window_alert("Dados salvos com sucesso!") # Ou usar rx.toast se disponível no tema
```

---

## 🔐 Autenticação, Cookies e Segurança
### Persistência com Cookies (rx.Cookie)
Para manter o usuário logado entre sessões, use cookies ou `Local Storage`.

```python
class AuthState(State):
    auth_token: rx.Cookie = "" # Token persistente
    email: str = ""
    password: str = ""

    def login(self):
        # Em um app real, valide contra o banco
        if self.email == "user@test.com" and self.password == "1234":
            self.auth_token = "sessao_valida_123"
            return rx.redirect("/dashboard")
        return rx.window_alert("Credenciais inválidas")
```

### Rotas protegidas
Use o estado para impedir o acesso a componentes ou páginas.

```python
def dashboard_page():
    return rx.cond(
        AuthState.auth_token != "",
        rx.vstack(
            rx.heading("Dashboard"),
            rx.text("Bem-vindo à área logada!")
        ),
        rx.redirect("/login")
    )
```

### Autenticação via backend
- Use um endpoint FastAPI para verificar credenciais.
- Armazene tokens de sessão ou cookies seguros via `rx.Cookie`.
- Não coloque credenciais sensíveis no frontend; processe-as sempre no servidor.

---

## 🗄️ Banco de Dados e Models
### Modelo SQLModel / Pydantic
```python
from reflex import Model
from sqlmodel import Field

class Usuario(Model, table=True):
    id: int = Field(default=None, primary_key=True)
    nome: str
    email: str
```

### CRUD básico
```python
class DBState(State):
    def criar_usuario(self, nome: str, email: str):
        with rx.session() as session:
            session.add(Usuario(nome=nome, email=email))
            session.commit()

    def listar_usuarios(self):
        with rx.session() as session:
            return session.query(Usuario).all()
```

### Migrations
- Reflex não gerencia migrations automaticamente.
- Use Alembic ou SQLModel para gerar e aplicar migrations.
- Mantenha o schema versionado no repositório.

---

## 📡 Backend, API Routes e WebHooks
### Rotas FastAPI customizadas
```python
from fastapi import Request

async def meu_webhook(request: Request):
    dados = await request.json()
    print(dados)
    return {"status": "ok"}

app = rx.App()
app.api.add_api_route("/webhook", meu_webhook, methods=["POST"])
```

### Endpoints REST
```python
@app.api.get("/api/usuarios")
async def get_users():
    with rx.session() as session:
        return session.query(Usuario).all()
```

### Chamadas do frontend ao backend
- Use ações do State para invocar endpoints.
- Use `requests` ou `httpx` no backend para chamadas externas.

---

## 🕸️ WebSockets e Tempo Real
Para apps em tempo real, use WebSockets via FastAPI no backend.

```python
from fastapi import WebSocket

@app.api.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"Eco: {data}")
    except Exception:
        await websocket.close()
```

### Uso em Reflex
- O Reflex pode consumir WebSockets no frontend compilado.
- Use WebSockets para notificações, chats e atualizações em tempo real.

---

## 🖼️ Assets e Arquivos Estáticos
### Assets e public
- Coloque imagens e CSS em `assets/`.
- Coloque arquivos públicos em `public/`.

### Usar imagens
```python
rx.image(src="/logo.png", alt="Logo")
```

### Favicons e meta tags
- Coloque `favicon.ico` e `robots.txt` em `public/`.
- Configure meta tags no frontend se precisar de SEO.

---

## 🧪 Testes, Debug e Logs
### Testes unitários
- Teste ações do `State` isoladamente.
- Simule mudanças de estado e verificações de retorno.

### Debug no servidor
- Ative logs do Reflex e do FastAPI.
- Registre erros e payloads importantes.

### Inspeção no browser
- Use o console do navegador gerado pelo Reflex/React.
- Verifique requisições de API, estados e eventos.

---

## 🚀 Build, Export e Deploy
### Desenvolvimento
```bash
reflex run
```

### Build do frontend
```bash
reflex export --frontend-only
```

### Produção
```bash
reflex run --env prod
```

### Deploy
- **Vercel / Netlify**: exporte o frontend e hospede `public/`/build.
- **Render / Railway**: use o backend FastAPI com o frontend gerado.
- **Docker**: crie um container com `reflex run --env prod`.

---

## 🛡️ Produção e Boas Práticas
1. **Mantenha o state enxuto**: evite armazenar dados pesados no frontend.
2. **Use WSS para WebSockets** em produção.
3. **Proteja APIs** com autenticação e validação.
4. **Implemente monitoramento** de erros e métricas.
5. **Separe frontend e backend logicamente** em modules diferentes.
6. **Versione migrations** e mantenha histórico de schema.
7. **Teste rotas e ações** em ambiente de staging antes de produção.

---

## ✅ Checklist da Bíblia Reflex
- [ ] Projeto inicializado com `reflex init`
- [ ] `rxconfig.py` configurado corretamente
- [ ] `State` organizado com **Substates** e **Computed Vars (@rx.var)**
- [ ] UI dinâmica usando **rx.cond** e **rx.foreach**
- [ ] Layout global e navegação definidos
- [ ] Formulários com **Toasts** e validação
- [ ] Autenticação com persistência via **Cookies**
- [ ] **Dashboards** com gráficos (recharts)
- [ ] Modelos de banco de dados com SQLModel
- [ ] Rotas FastAPI customizadas e WebHooks
- [ ] WebSockets ou tempo real configurados
- [ ] Assets e public configurados
- [ ] Logs e testes de State implementados
- [ ] Build e deploy documentados
- [ ] Boas práticas de produção aplicadas

---

## 💡 Observações Finais
Este guia agora cobre Reflex como uma plataforma full-stack, do estado à UI, do backend ao deploy. Para ser a “bíblia” definitiva, mantenha-o atualizado com as evoluções do Reflex e com exemplos reais de projetos em produção.
