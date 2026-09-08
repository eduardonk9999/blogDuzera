---
title: "LangChain: Orquestração com LangGraph"
date: 2026-09-08
description: "Como criar fluxos complexos e agentes com LangGraph"
tags: ["python", "ia", "langchain", "llm", "langgraph"]
draft: false
---

## O que é orquestração?

Orquestração é coordenar múltiplos componentes em fluxos complexos. Enquanto cadeias LCEL são lineares, orquestração permite:

- Loops e ciclos
- Decisões condicionais
- Estados persistentes
- Execução paralela coordenada

## LangGraph

LangGraph é a biblioteca oficial para orquestração no ecossistema LangChain. Modela fluxos como grafos com nós e arestas.

```bash
pip install langgraph
```

## Conceitos básicos

```python
from langgraph.graph import StateGraph, START, END

# 1. Define o estado
class Estado(TypedDict):
    mensagens: list
    proximo_passo: str

# 2. Cria o grafo
grafo = StateGraph(Estado)

# 3. Adiciona nós (funções)
grafo.add_node("processar", funcao_processar)
grafo.add_node("responder", funcao_responder)

# 4. Adiciona arestas (conexões)
grafo.add_edge(START, "processar")
grafo.add_edge("processar", "responder")
grafo.add_edge("responder", END)

# 5. Compila e executa
app = grafo.compile()
resultado = app.invoke({"mensagens": [], "proximo_passo": ""})
```

## Exemplo: Chatbot simples

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")

class Estado(TypedDict):
    mensagens: Annotated[list, add_messages]

def chatbot(estado: Estado) -> Estado:
    resposta = llm.invoke(estado["mensagens"])
    return {"mensagens": [resposta]}

# Monta o grafo
grafo = StateGraph(Estado)
grafo.add_node("chatbot", chatbot)
grafo.add_edge(START, "chatbot")
grafo.add_edge("chatbot", END)

app = grafo.compile()

# Executa
resultado = app.invoke({
    "mensagens": [("user", "O que é Python?")]
})
print(resultado["mensagens"][-1].content)
```

## Arestas condicionais

Decisões baseadas no estado:

```python
from langgraph.graph import StateGraph, START, END

def roteador(estado: Estado) -> str:
    ultima_msg = estado["mensagens"][-1].content.lower()
    if "código" in ultima_msg:
        return "programador"
    elif "receita" in ultima_msg:
        return "chef"
    return "geral"

grafo = StateGraph(Estado)
grafo.add_node("programador", node_programador)
grafo.add_node("chef", node_chef)
grafo.add_node("geral", node_geral)

# Aresta condicional
grafo.add_conditional_edges(
    START,
    roteador,
    {
        "programador": "programador",
        "chef": "chef",
        "geral": "geral"
    }
)

grafo.add_edge("programador", END)
grafo.add_edge("chef", END)
grafo.add_edge("geral", END)
```

## Loops: Human-in-the-loop

```python
from langgraph.checkpoint.memory import MemorySaver

def precisa_aprovacao(estado: Estado) -> str:
    if estado.get("aprovado"):
        return "executar"
    return "aguardar"

grafo = StateGraph(Estado)
grafo.add_node("propor", node_propor)
grafo.add_node("aguardar", node_aguardar)
grafo.add_node("executar", node_executar)

grafo.add_edge(START, "propor")
grafo.add_conditional_edges("propor", precisa_aprovacao)
grafo.add_edge("aguardar", "propor")  # Loop!
grafo.add_edge("executar", END)

# Checkpoint para persistir estado
checkpointer = MemorySaver()
app = grafo.compile(checkpointer=checkpointer)
```

## Agentes com tools

Agentes decidem quais ferramentas usar:

```python
from langchain_core.tools import tool
from langgraph.prebuilt import create_react_agent

@tool
def calcular(expressao: str) -> str:
    """Calcula expressões matemáticas."""
    return str(eval(expressao))

@tool
def buscar(query: str) -> str:
    """Busca informações na web."""
    return f"Resultado para: {query}"

# Cria agente ReAct
agente = create_react_agent(
    model=ChatOpenAI(model="gpt-4o"),
    tools=[calcular, buscar]
)

resultado = agente.invoke({
    "messages": [("user", "Quanto é 25 * 4 + 10?")]
})
```

## Subgrafos

Componha grafos dentro de grafos:

```python
# Grafo interno
subgrafo = StateGraph(Estado)
subgrafo.add_node("etapa1", funcao1)
subgrafo.add_node("etapa2", funcao2)
subgrafo.add_edge(START, "etapa1")
subgrafo.add_edge("etapa1", "etapa2")
subgrafo.add_edge("etapa2", END)

# Grafo principal
grafo_principal = StateGraph(Estado)
grafo_principal.add_node("inicio", funcao_inicio)
grafo_principal.add_node("processamento", subgrafo.compile())
grafo_principal.add_node("fim", funcao_fim)

grafo_principal.add_edge(START, "inicio")
grafo_principal.add_edge("inicio", "processamento")
grafo_principal.add_edge("processamento", "fim")
grafo_principal.add_edge("fim", END)
```

## Execução paralela

```python
from langgraph.graph import StateGraph, START, END

grafo = StateGraph(Estado)

grafo.add_node("analise_a", funcao_a)
grafo.add_node("analise_b", funcao_b)
grafo.add_node("combinar", funcao_combinar)

# Fan-out: START vai para ambos
grafo.add_edge(START, "analise_a")
grafo.add_edge(START, "analise_b")

# Fan-in: ambos vão para combinar
grafo.add_edge("analise_a", "combinar")
grafo.add_edge("analise_b", "combinar")
grafo.add_edge("combinar", END)
```

## Persistência de estado

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.checkpoint.postgres import PostgresSaver

# Memória (dev)
checkpointer = MemorySaver()

# PostgreSQL (prod)
checkpointer = PostgresSaver.from_conn_string(
    "postgresql://user:pass@localhost/db"
)

app = grafo.compile(checkpointer=checkpointer)

# Executa com thread_id para persistir
config = {"configurable": {"thread_id": "conversa_123"}}
resultado = app.invoke({"mensagens": [...]}, config=config)

# Retoma depois
resultado = app.invoke({"mensagens": [...]}, config=config)
```

## Streaming

```python
app = grafo.compile()

for evento in app.stream({"mensagens": [("user", "Olá!")]}):
    print(evento)
```

## Exemplo completo: Agente pesquisador

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool

class Estado(TypedDict):
    mensagens: Annotated[list, add_messages]

llm = ChatOpenAI(model="gpt-4o")

@tool
def pesquisar_web(query: str) -> str:
    """Pesquisa informações na web."""
    # Integre com API de busca real
    return f"Resultados para '{query}': ..."

@tool
def analisar_dados(dados: str) -> str:
    """Analisa dados e extrai insights."""
    return f"Análise: {dados[:100]}..."

tools = [pesquisar_web, analisar_dados]
llm_com_tools = llm.bind_tools(tools)

def agente(estado: Estado) -> Estado:
    resposta = llm_com_tools.invoke(estado["mensagens"])
    return {"mensagens": [resposta]}

def executar_tools(estado: Estado) -> Estado:
    ultima = estado["mensagens"][-1]
    resultados = []
    for tool_call in ultima.tool_calls:
        tool_fn = {"pesquisar_web": pesquisar_web, "analisar_dados": analisar_dados}
        resultado = tool_fn[tool_call["name"]].invoke(tool_call["args"])
        resultados.append({"tool_call_id": tool_call["id"], "content": resultado})
    return {"mensagens": resultados}

def deve_continuar(estado: Estado) -> str:
    ultima = estado["mensagens"][-1]
    if hasattr(ultima, "tool_calls") and ultima.tool_calls:
        return "tools"
    return END

grafo = StateGraph(Estado)
grafo.add_node("agente", agente)
grafo.add_node("tools", executar_tools)

grafo.add_edge(START, "agente")
grafo.add_conditional_edges("agente", deve_continuar, {"tools": "tools", END: END})
grafo.add_edge("tools", "agente")

app = grafo.compile()
```

## Resumo

| Conceito | Uso |
|----------|-----|
| `StateGraph` | Define o grafo |
| `add_node` | Adiciona nós |
| `add_edge` | Conexão direta |
| `add_conditional_edges` | Decisões |
| `compile` | Prepara execução |
| `checkpointer` | Persistência |
| `create_react_agent` | Agente pronto |

LangGraph transforma LLMs em sistemas complexos com estado, loops e decisões.
