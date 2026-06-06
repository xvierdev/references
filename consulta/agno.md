# 🦅 Bíblia do Agno: Agentes de IA de Próxima Geração

O **Agno** (anteriormente Phidata) é o framework definitivo para construir sistemas de Agentes de IA com memória, conhecimento (RAG) e ferramentas (Tool Calling). Ele é focado em performance e integração full-stack.

---

## 📑 Sumário
1. [🛠️ Instalação](#-instalação)
2. [🤖 Agente Básico e Provedores (Groq/OpenAI)](#-agente-básico-e-provedores-groqopenai)
3. [🛠️ Tool Calling (Ferramentas Personalizadas)](#️-tool-calling-ferramentas-personalizadas)
4. [🧩 Saída Estruturada (Pydantic)](#-saída-estruturada-pydantic)
5. [🤝 Multi-Agent Teams (Orquestração)](#-multi-agent-teams-orquestração)
6. [📚 Knowledge Base (RAG e Vector DBs)](#-knowledge-base-rag-e-vector-dbs)
7. [🧠 Memória, Persistência e Sessões](#-memória-persistência-e-sessões)
8. [🚀 Agentes de Raciocínio (Reasoning)](#-agentes-de-raciocínio-reasoning)
9. [🛡️ Produção, Logs e Segurança](#️-produção-logs-e-segurança)
10. [✅ Checklist de Agentes Agno](#-checklist-de-agentes-agno)

---

## 🛠️ Instalação
```bash
pip install agno groq chromadb ollama pydantic
```

---

## 🤖 Agente Básico e Provedores
O Agno suporta múltiplos modelos. O Groq é recomendado pela velocidade extrema.

```python
from agno.agent import Agent
from agno.models.groq import Groq
# from agno.models.openai import OpenAIChat

agent = Agent(
    model=Groq(id="llama-3.3-70b-versatile"),
    description="Você é um assistente técnico especializado em Python.",
    markdown=True,
    instructions=["Seja conciso", "Use exemplos de código"]
)

agent.print_response("Como criar um decorador em Python?")
```

---

## 🛠️ Tool Calling (Ferramentas Personalizadas)
Agentes podem executar funções Python. Sempre documente as funções com *docstrings* claras para a LLM entender quando usar.

```python
def get_weather(city: str) -> str:
    """Busca o clima de uma cidade.
    Args:
        city (str): Nome da cidade.
    """
    return f"O clima em {city} está ensolarado com 25°C."

agent = Agent(
    model=Groq(id="llama-3.3-70b-versatile"),
    tools=[get_weather],
    show_tool_calls=True
)
```

---

## 🧩 Saída Estruturada (Pydantic)
Force o agente a responder em um formato JSON válido usando modelos Pydantic. Ideal para integrações de API.

```python
from pydantic import BaseModel, Field

class MovieReview(BaseModel):
    title: str = Field(..., description="Título do filme")
    rating: int = Field(..., description="Nota de 1 a 10")
    summary: str = Field(..., description="Resumo curto")

agent = Agent(
    model=Groq(id="llama-3.3-70b-versatile"),
    response_model=MovieReview,
    description="Você é um crítico de cinema."
)

# O retorno será um objeto MovieReview (instância de Pydantic)
review = agent.run("Fale sobre o filme Inception")
print(review.content.title)
```

---

## 🤝 Multi-Agent Teams (Orquestração)
Equipes de agentes permitem separar responsabilidades (Separação de Preocupações).

```python
finance_agent = Agent(name="Finance", role="Analista Financeiro", tools=[...])
news_agent = Agent(name="News", role="Analista de Notícias", tools=[...])

agent_team = Agent(
    team=[finance_agent, news_agent],
    instructions=["O Finance Agent busca preços", "O News Agent busca notícias", "Resuma ambos."],
    show_tool_calls=True,
)
```

---

## 📚 Knowledge Base (RAG e Vector DBs)
Conecte o agente a PDFs, Bancos de Dados ou Sites.

```python
from agno.knowledge.pdf import PDFKnowledgeBase, PDFReader
from agno.vectordb.chroma import ChromaDb

knowledge_base = PDFKnowledgeBase(
    path="data/pdfs",
    vector_db=ChromaDb(collection="docs", path="db/chroma"),
    reader=PDFReader(chunk_size=512)
)
knowledge_base.load(recreate=False)

agent = Agent(knowledge=knowledge_base, search_knowledge=True)
```

---

## 🧠 Memória, Persistência e Sessões
Para que o agente lembre do usuário em diferentes execuções.

```python
from agno.storage.agent.sqlite import SqlAgentStorage

storage = SqlAgentStorage(table_name="agent_sessions", db_file="agents.db")

agent = Agent(
    storage=storage,
    add_history_to_messages=True, # Adiciona contexto anterior ao prompt
    num_history_responses=5      # Quantidade de trocas que ele lembra
)

# Retomar uma sessão específica pelo session_id
# agent.print_response("Oi", session_id="user_123")
```

---

## 🚀 Agentes de Raciocínio (Reasoning)
Agentes que pensam antes de responder, ideal para problemas complexos ou lógica matemática.

```python
from agno.agent import Agent

reasoning_agent = Agent(
    model=Groq(id="deepseek-r1-distill-llama-70b"), # Modelo focado em raciocínio
    reasoning=True,
    instructions=["Mostre seu passo a passo de pensamento."]
)
```

---

## 🛡️ Produção, Logs e Segurança

### Logs e Debugging
Use `debug_mode=True` no desenvolvimento para ver o prompt exato. Para produção, integre com ferramentas de observabilidade.

```python
agent = Agent(debug_mode=True)
```

### Segurança de Ferramentas
Sempre valide as entradas das ferramentas. Nunca dê ferramentas que executem código arbitrário (como `eval`) sem extrema cautela.

### Limites de Contexto
Se o histórico ficar muito grande, o custo aumenta e a performance cai. Use `num_history_responses` para limitar.

---

## ✅ Checklist de Agentes Agno

- [ ] Modelo (Model) escolhido adequadamente (Groq para velocidade, OpenAI para complexidade).
- [ ] Instruções (Instructions) claras e específicas.
- [ ] Ferramentas (Tools) documentadas com Docstrings.
- [ ] Memória e Storage configurados se precisar persistir sessões.
- [ ] Base de Conhecimento (Knowledge) carregada e testada.
- [ ] Resposta estruturada com Pydantic (se for integração de sistema).
- [ ] `debug_mode` desligado em produção.
- [ ] Tratamento de erros para falhas de API ou Limites de Rate.
- [ ] Logs monitorados (ex: integrando com Loguru).

---

## 💡 Notas Finais
O Agno é poderoso porque é agnóstico ao modelo. Você pode trocar de Groq para OpenAI mudando uma única linha. Para sistemas complexos, priorize a divisão em múltiplos agentes pequenos em vez de um único "super agente".
