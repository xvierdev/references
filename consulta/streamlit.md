# 🚀 Bíblia do Streamlit

Este guia transforma o arquivo Streamlit em uma referência completa para construção de aplicações interativas, visuais e de dados.

---

## 📑 Sumário
1. [📝 Texto e Markdown](#-texto-e-markdown)
2. [📊 Exibição de Dados](#-exibição-de-dados)
3. [🎛️ Widgets de Entrada](#-widgets-de-entrada)
4. [📐 Layout e Organização](#-layout-e-organização)
5. [🎥 Multimídia e Arquivos](#-multimídia-e-arquivos)
6. [🧠 Estado e Controle de Fluxo](#-estado-e-controle-de-fluxo)
7. [⚡ Caching e Performance](#-caching-e-performance)
8. [🎨 Configuração e Temas](#-configuração-e-temas)
9. [🧩 Recursos Avançados](#-recursos-avançados)
10. [💡 Boas Práticas e Dicas](#-boas-práticas-e-dicas)
11. [🚀 Deploy e CI](#-deploy-e-ci)

> Dica: este sumário é manual, mas você pode usar extensões de Markdown (por exemplo, Markdown All in One no VS Code) para gerar tabelas de conteúdo automaticamente a partir dos cabeçalhos do documento.

---

## 📝 Texto e Markdown

### Títulos e Texto
```python
import streamlit as st

st.title("Título Principal")
st.header("Cabeçalho de Seção")
st.subheader("Subcabeçalho")
st.write("Texto simples com **renderização automática**.")
st.markdown("Texto em **negrito**, *itálico* ou [link](https://streamlit.io)")
st.caption("Legenda pequena")
st.code("print('Hello World')", language='python')
```

### Texto Formatado
- `st.write`: renderiza texto, DataFrames, listas e objetos Python
- `st.markdown`: texto Markdown com suporte mais direto
- `st.latex`: para fórmulas matemáticas
- `st.json`: exibindo JSON formatado

```python
st.latex(r"e^{i\pi} + 1 = 0")
st.json({"nome": "Streamlit", "tipo": "app"})
```

### Mensagens de Status
```python
st.success("Sucesso!")
st.info("Informação útil.")
st.warning("Aviso.")
st.error("Erro!")
```

### Contêineres de Texto
```python
st.markdown("---")
with st.container():
    st.write("Conteúdo dentro de um container")
```

---

## 📊 Exibição de Dados

### DataFrames e Tabelas
```python
import pandas as pd

df = pd.DataFrame({"A": [1, 2, 3], "B": [4, 5, 6]})

st.dataframe(df)  # Interativo
st.table(df)      # Estático
```

### Data Editor
```python
st.data_editor(df)
```

### JSON e Dados Estruturados
```python
st.json({"nome": "Streamlit", "versao": 1.0})
```

---

## 📈 Gráficos e Visualizações

### Gráficos nativos
```python
st.line_chart([1, 5, 2, 6])
st.bar_chart([1, 5, 2, 6])
st.area_chart([1, 5, 2, 6])
```

### Gráficos com bibliotecas externas
```python
import altair as alt
import numpy as np

chart = alt.Chart(pd.DataFrame({
    "x": np.arange(10),
    "y": np.random.randn(10)
})).mark_line().encode(x="x", y="y")

st.altair_chart(chart, use_container_width=True)
```

### Outros gráficos populares
```python
st.pyplot(fig)
st.plotly_chart(fig)
st.vega_lite_chart(data)
st.map(df_map)
```

---

## 🎛️ Widgets de Entrada

### Texto e Campos Numéricos
```python
nome = st.text_input("Digite seu nome")
senha = st.text_input("Senha", type="password")
textarea = st.text_area("Descrição")
numero = st.number_input("Idade", min_value=0, max_value=120, step=1)
```

### Seleção de Opções
```python
ok = st.checkbox("Aceito os termos")
cor = st.color_picker("Escolha uma cor", "#00f900")
opcao = st.radio("Escolha uma opção", ["A", "B", "C"])
selecao = st.selectbox("Escolha uma cidade", ["São Paulo", "Rio", "BH"])
multi = st.multiselect("Escolha várias", ["A", "B", "C"])
```

### Sliders e Controle de Intervalo
```python
valor = st.slider("Faixa", 0, 100, 25)
intervalo = st.select_slider("Intervalo", options=[1, 2, 3, 4, 5])
data = st.date_input("Data de nascimento")
hora = st.time_input("Hora de início")
```

### Botões e Downloads
```python
if st.button("Clique aqui"):
    st.write("Botão foi clicado")

if st.download_button("Baixar CSV", data=df.to_csv(index=False), file_name="dados.csv"):
    st.success("Download iniciado")
```

### Upload de Arquivos e Câmera
```python
arquivo = st.file_uploader("Envie um arquivo")
if arquivo is not None:
    st.write(arquivo.name)

imagem = st.camera_input("Tire uma foto")
if imagem is not None:
    st.image(imagem)
```

---

## 📐 Layout e Organização

### Colunas e Containers
```python
col1, col2, col3 = st.columns([1, 2, 1])
col1.write("Coluna 1")
col2.write("Coluna 2 maior")
col3.write("Coluna 3")
```

### Abas (Tabs)
```python
aba1, aba2 = st.tabs(["Gráfico", "Dados"])
with aba1:
    st.line_chart([1, 2, 3])
with aba2:
    st.write(df)
```

### Barra Lateral
```python
st.sidebar.title("Configurações")
st.sidebar.checkbox("Exibir gráfico")
```

### Expanders e Placeholders
```python
with st.expander("Mais informações"):
    st.write("Detalhes adicionais")

placeholder = st.empty()
placeholder.write("Conteúdo inicial")
placeholder.text("Atualizado depois")
```

### Layout Responsivo
- `use_container_width=True` ajusta gráficos ao tamanho do container
- `st.columns` escolhe larguras relativas usando array de inteiros

---

## 🎥 Multimídia e Arquivos

### Imagens, Áudio e Vídeo
```python
st.image("https://www.python.org/static/community_logos/python-logo.png", caption="Logo Python")
st.audio("audio.mp3")
st.video("video.mp4")
```

### Arquivos e Downloads
```python
arquivo = st.file_uploader("Envie um arquivo", type=["csv", "txt"])
if arquivo:
    st.write(arquivo.name)

st.download_button("Baixar texto", "conteúdo", file_name="arquivo.txt")
```

### Modelos Interativos de Dados
```python
st.map(pd.DataFrame({"lat": [40.7128], "lon": [-74.0060]}))
```

---

## 🧠 Estado e Controle de Fluxo

### Session State
```python
if "contador" not in st.session_state:
    st.session_state.contador = 0


def incrementar():
    st.session_state.contador += 1

st.button("Incrementar", on_click=incrementar)
st.write(f"Contador: {st.session_state.contador}")
```

### Usando Session State com Widgets
```python
st.session_state.nome = st.session_state.get("nome", "")
nome = st.text_input("Nome", key="nome")
```

### Forms
```python
with st.form("formulario"):
    nome = st.text_input("Nome")
    email = st.text_input("Email")
    enviado = st.form_submit_button("Enviar")

if enviado:
    st.success(f"Mensagem enviada para {email}")
```

### Callbacks e Eventos
```python
def clique():
    st.session_state.contador += 1

st.button("Clique", on_click=clique)
```

### Condições de Renderização
```python
if st.checkbox("Mostrar quadro"):
    st.write("Aqui está o quadro")
```

---

## ⚡ Caching e Performance

### Cache de Dados
```python
@st.cache_data
def carregar_dados(path):
    return pd.read_csv(path)
```

### Cache de Recursos
```python
@st.cache_resource
def criar_conexao():
    return criar_banco_de_dados()
```

### Controle de Atualização
```python
with st.spinner("Carregando..."):
    time.sleep(2)
st.success("Concluído")

st.progress(50)
```

### Rerun e Otimização
- `st.experimental_rerun()` reinicia o script imediatamente
- Evite operações pesadas fora de funções que possam ser memoizadas

---

## 🎨 Configuração e Temas

### `st.set_page_config`
```python
st.set_page_config(
    page_title="Meu App",
    page_icon="🚀",
    layout="wide",
    initial_sidebar_state="expanded"
)
```

### Tema e Estilo
- Personalize ícone, nome, layout e tamanho de menu
- Use CSS limitado via componentes HTML ou `st.markdown` com `unsafe_allow_html=True`

### Query Parameters
```python
st.experimental_set_query_params(page="1")
params = st.experimental_get_query_params()
```

---

## 🧩 Recursos Avançados

### Componentes Personalizados
```python
import streamlit.components.v1 as components
components.html("<h1>Componente HTML</h1>")
```

### Integração com APIs e ML
- Consumir API com `requests`, exibir resultados com `st.json`
- Mostrar inferências de modelo usando `st.image`, `st.dataframe`, `st.metric`

### Mensagens e Alertas
```python
st.success("Tudo certo")
st.warning("Atenção")
st.error("Algo deu errado")
st.info("Informação")
```

### Métricas em Destaque
```python
st.metric("Temperatura", "25°C", delta="+2°C")
```

### Depuração e Logs
- Use `st.write` para inspecionar variáveis
- Use a barra lateral para filtros e parâmetros de depuração

---

## 💡 Boas Práticas e Dicas

- Separe lógica de dados da interface visual
- Compare `st.write` e `st.markdown` para escolher o melhor formato
- Cacheie dados caros com `st.cache_data`
- Use `st.session_state` para manter estado entre recargas
- Prefira layout `wide` para dashboards complexos
- Teste em diferentes tamanhos de tela e resoluções
- Documente botões e inputs com `help` ou texto explicativo

### Checklist para apps Streamlit
- [x] Configuração de página definida
- [x] Inputs com chaves únicas (`key=`)
- [x] Uso de sidebar para controles de filtro
- [x] Evitar loops pesados no corpo principal
- [x] Usar placeholder para atualizações dinâmicas

---

## 🚀 Deploy e CI

### Preparar o app para produção
- Confirme o arquivo `requirements.txt` ou `pyproject.toml` com todas as dependências.
- Use `st.set_page_config()` para definir título, ícone e layout.
- Separe configuração e credenciais usando variáveis de ambiente ou `st.secrets`.
- Mantenha `main.py` ou `app.py` com a lógica de inicialização limpa e testável.

### Deploy em Streamlit Cloud
1. Crie um repositório GitHub público ou privado.
2. Acesse `share.streamlit.io` e conecte sua conta GitHub.
3. Escolha o repositório e o branch para deploy.
4. Defina o comando de inicialização se necessário (`streamlit run app.py`).

### Deploy em outros provedores
- `Heroku`: use `Procfile` com `web: streamlit run app.py --server.port $PORT`
- `Render`: selecione o repositório e configure `streamlit run app.py`
- `Google Cloud Run`: crie imagem Docker com o app e publique no container registry
- `AWS Elastic Beanstalk`: configure plataforma Python e comando de execução

### Docker básico para Streamlit
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 8501
CMD ["streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

### Exemplos de CI
- GitHub Actions:
  - rodar `pip install -r requirements.txt`
  - executar testes com `pytest`
  - checar lint com `ruff`/`black`
  - opcional: criar artefato Docker ou build de imagem
- GitLab CI / Bitbucket Pipelines: mesma lógica de build, teste e deploy automatizado

### Boas práticas de CI para apps Streamlit
- Teste componentes de backend e funções de transformação de dados separadamente.
- Valide performance de cargas de dados e caching.
- Execute linting e formatação antes de merge.
- Use branches e PRs com revisão antes do deploy.

---

## 📚 Referências Rápidas
- `st.title`, `st.header`, `st.subheader`, `st.markdown`
- `st.write`, `st.code`, `st.latex`, `st.json`
- `st.dataframe`, `st.table`, `st.data_editor`
- `st.line_chart`, `st.bar_chart`, `st.area_chart`, `st.map`
- `st.button`, `st.checkbox`, `st.selectbox`, `st.file_uploader`
- `st.sidebar`, `st.columns`, `st.tabs`, `st.expander`
- `st.session_state`, `st.form`, `st.cache_data`, `st.set_page_config`
