---
title: "LangChain: Memória em Conversas"
date: 2026-09-07
description: "Como manter contexto entre mensagens usando memória no LangChain"
tags: ["python", "ia", "langchain", "llm"]
draft: false
---

## Por que memória?

LLMs são stateless - cada chamada é independente. Sem memória, o modelo não lembra o que foi dito antes:

```python
# Sem memória
llm.invoke("Meu nome é Eduardo")  # "Prazer, Eduardo!"
llm.invoke("Qual meu nome?")      # "Não sei seu nome"
```

## Memória com mensagens

A forma moderna de implementar memória no LangChain é gerenciar o histórico de mensagens:

```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage

llm = ChatOpenAI(model="gpt-4o-mini")

# Histórico de mensagens
historico = []

def chat(mensagem: str) -> str:
    historico.append(HumanMessage(content=mensagem))
    resposta = llm.invoke(historico)
    historico.append(resposta)
    return resposta.content

chat("Meu nome é Eduardo")
chat("Qual meu nome?")  # "Seu nome é Eduardo!"
```

## ChatMessageHistory

Classe utilitária para gerenciar histórico:

```python
from langchain_community.chat_message_histories import ChatMessageHistory

historia = ChatMessageHistory()

historia.add_user_message("Olá!")
historia.add_ai_message("Oi! Como posso ajudar?")
historia.add_user_message("Qual a capital do Brasil?")

# Acessa as mensagens
print(historia.messages)
```

## RunnableWithMessageHistory

Integra memória com cadeias LCEL:

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_community.chat_message_histories import ChatMessageHistory

# Store de sessões
store = {}

def get_session_history(session_id: str):
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]

# Prompt com placeholder para histórico
prompt = ChatPromptTemplate.from_messages([
    ("system", "Você é um assistente útil."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])

cadeia = prompt | llm

# Adiciona memória à cadeia
cadeia_com_memoria = RunnableWithMessageHistory(
    cadeia,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history"
)

# Usa com session_id
config = {"configurable": {"session_id": "usuario_123"}}

cadeia_com_memoria.invoke({"input": "Meu nome é Eduardo"}, config=config)
cadeia_com_memoria.invoke({"input": "Qual meu nome?"}, config=config)
# "Seu nome é Eduardo!"
```

## Limitando o histórico

Conversas longas estouram o limite de tokens. Soluções:

### Janela de mensagens

```python
from langchain_core.messages import trim_messages

# Mantém apenas as últimas N mensagens
mensagens_trimadas = trim_messages(
    historico.messages,
    max_tokens=1000,
    token_counter=llm,
    strategy="last"
)
```

### Resumo do histórico

```python
from langchain_core.prompts import ChatPromptTemplate

prompt_resumo = ChatPromptTemplate.from_template(
    "Resuma esta conversa em 2-3 frases:\n\n{conversa}"
)

def resumir_historico(mensagens):
    conversa = "\n".join([f"{m.type}: {m.content}" for m in mensagens])
    resumo = (prompt_resumo | llm).invoke({"conversa": conversa})
    return resumo.content
```

## Persistência com Redis

Para manter memória entre reinicializações:

```python
from langchain_community.chat_message_histories import RedisChatMessageHistory

def get_session_history(session_id: str):
    return RedisChatMessageHistory(
        session_id=session_id,
        url="redis://localhost:6379"
    )

# Histórico persiste no Redis
cadeia_com_memoria = RunnableWithMessageHistory(
    cadeia,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history"
)
```

## Persistência com PostgreSQL

```python
from langchain_community.chat_message_histories import PostgresChatMessageHistory

def get_session_history(session_id: str):
    return PostgresChatMessageHistory(
        session_id=session_id,
        connection_string="postgresql://user:pass@localhost/db"
    )
```

## Persistência com arquivo

Para desenvolvimento local:

```python
from langchain_community.chat_message_histories import FileChatMessageHistory

def get_session_history(session_id: str):
    return FileChatMessageHistory(f"./historico_{session_id}.json")
```

## Exemplo completo: Chatbot com memória

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_community.chat_message_histories import ChatMessageHistory

# Setup
llm = ChatOpenAI(model="gpt-4o-mini")
store = {}

def get_history(session_id: str):
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]

prompt = ChatPromptTemplate.from_messages([
    ("system", "Você é um assistente amigável chamado Bot."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])

chatbot = RunnableWithMessageHistory(
    prompt | llm,
    get_history,
    input_messages_key="input",
    history_messages_key="history"
)

# Loop de conversa
session = {"configurable": {"session_id": "user_1"}}

while True:
    user_input = input("Você: ")
    if user_input.lower() == "sair":
        break
    response = chatbot.invoke({"input": user_input}, config=session)
    print(f"Bot: {response.content}")
```

## Resumo

| Tipo | Uso |
|------|-----|
| `ChatMessageHistory` | Memória em RAM |
| `RedisChatMessageHistory` | Persistência Redis |
| `PostgresChatMessageHistory` | Persistência PostgreSQL |
| `FileChatMessageHistory` | Persistência arquivo |
| `trim_messages` | Limitar tokens |
| `RunnableWithMessageHistory` | Integrar com LCEL |

Memória transforma um LLM stateless em um chatbot que lembra do contexto.
