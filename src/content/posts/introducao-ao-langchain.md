---
title: "Introdução ao LangChain"
date: 2026-09-03
description: "Aprenda o básico do LangChain com exemplos práticos em Python"
tags: ["python", "ia", "langchain", "llm"]
draft: false
---

## O que é LangChain?

LangChain é um framework open-source para construir aplicações com LLMs (Large Language Models). Ele facilita a criação de pipelines que combinam modelos de linguagem com outras fontes de dados e ferramentas.

## Instalação

```bash
pip install langchain langchain-openai python-dotenv
```

## Exemplo básico: Chat simples

```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, SystemMessage

# Inicializa o modelo
llm = ChatOpenAI(model="gpt-4o-mini")

# Cria as mensagens
messages = [
    SystemMessage(content="Você é um assistente útil."),
    HumanMessage(content="O que é Python?")
]

# Faz a chamada
response = llm.invoke(messages)
print(response.content)
```

## Usando Prompt Templates

Templates permitem criar prompts dinâmicos e reutilizáveis:

```python
from langchain_core.prompts import ChatPromptTemplate

# Define o template
template = ChatPromptTemplate.from_messages([
    ("system", "Você é um especialista em {area}."),
    ("human", "{pergunta}")
])

# Cria a chain
chain = template | llm

# Executa
response = chain.invoke({
    "area": "programação",
    "pergunta": "Qual a diferença entre lista e tupla em Python?"
})

print(response.content)
```

## Chains: Combinando componentes

O poder do LangChain está em encadear componentes:

```python
from langchain_core.output_parsers import StrOutputParser

# Chain com parser de output
chain = template | llm | StrOutputParser()

# Agora retorna string diretamente
resultado = chain.invoke({
    "area": "DevOps",
    "pergunta": "O que é Docker?"
})

print(resultado)
```

## Configurando a API Key

Crie um arquivo `.env`:

```bash
OPENAI_API_KEY=sua-chave-aqui
```

E carregue no código:

```python
from dotenv import load_dotenv
load_dotenv()
```

## Próximos passos

- **RAG** - Retrieval Augmented Generation para consultar documentos
- **Agents** - LLMs que podem usar ferramentas externas
- **Memory** - Manter contexto entre conversas

A [documentação oficial](https://python.langchain.com/docs/) é excelente para se aprofundar.
