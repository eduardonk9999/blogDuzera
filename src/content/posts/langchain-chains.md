---
title: "LangChain: Entendendo Cadeias (Chains)"
date: 2026-09-04
description: "Como criar cadeias (chains) poderosas combinando componentes no LangChain"
tags: ["python", "ia", "langchain", "llm"]
draft: false
---

## O que são Cadeias?

Cadeias são o conceito central do LangChain. A ideia é simples: encadear componentes onde a saída de um vira entrada do próximo, usando o operador `|` (pipe).

```python
chain = componente1 | componente2 | componente3
```

## Cadeia básica: Prompt + LLM + Parser

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Componentes
prompt = ChatPromptTemplate.from_template(
    "Explique {conceito} em uma frase simples."
)
llm = ChatOpenAI(model="gpt-4o-mini")
parser = StrOutputParser()

# Monta a cadeia
chain = prompt | llm | parser

# Executa
resultado = chain.invoke({"conceito": "recursão"})
print(resultado)
# "Recursão é quando uma função chama a si mesma..."
```

## Cadeias sequenciais

Você pode encadear múltiplas cadeias para criar fluxos complexos:

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Chain 1: Gera uma explicação
chain_explicar = (
    ChatPromptTemplate.from_template("Explique {tema} de forma técnica.")
    | llm
    | StrOutputParser()
)

# Chain 2: Simplifica a explicação
chain_simplificar = (
    ChatPromptTemplate.from_template(
        "Simplifique este texto para uma criança de 10 anos:\n\n{texto}"
    )
    | llm
    | StrOutputParser()
)

# Combinando as cadeias
def pipeline(tema: str) -> str:
    explicacao = chain_explicar.invoke({"tema": tema})
    simples = chain_simplificar.invoke({"texto": explicacao})
    return simples

resultado = pipeline("API REST")
print(resultado)
```

## RunnablePassthrough e RunnableLambda

Para manipular dados entre componentes:

```python
from langchain_core.runnables import RunnablePassthrough, RunnableLambda

# Passthrough mantém o input original
chain = (
    {"conceito": RunnablePassthrough()}
    | prompt
    | llm
    | parser
)

resultado = chain.invoke("polimorfismo")

# Lambda para transformações customizadas
def formatar(texto: str) -> str:
    return texto.upper()

chain_com_formato = (
    prompt
    | llm
    | parser
    | RunnableLambda(formatar)
)
```

## Parallel: Executando em paralelo

Execute múltiplas cadeias ao mesmo tempo:

```python
from langchain_core.runnables import RunnableParallel

# Define cadeias paralelas
chain_paralela = RunnableParallel(
    resumo=ChatPromptTemplate.from_template("Resuma: {texto}") | llm | parser,
    sentimento=ChatPromptTemplate.from_template("Qual o sentimento: {texto}") | llm | parser,
    keywords=ChatPromptTemplate.from_template("Liste 3 palavras-chave: {texto}") | llm | parser
)

resultado = chain_paralela.invoke({
    "texto": "Python é uma linguagem incrível para iniciantes..."
})

print(resultado["resumo"])
print(resultado["sentimento"])
print(resultado["keywords"])
```

## Tratando erros com fallbacks

```python
# Se o modelo principal falhar, usa o fallback
modelo_principal = ChatOpenAI(model="gpt-4o")
modelo_backup = ChatOpenAI(model="gpt-4o-mini")

chain_resiliente = (
    prompt
    | modelo_principal.with_fallbacks([modelo_backup])
    | parser
)
```

## Batch e Stream

Processe múltiplos inputs ou receba respostas em tempo real:

```python
# Batch: processa lista de inputs
conceitos = [
    {"conceito": "herança"},
    {"conceito": "encapsulamento"},
    {"conceito": "abstração"}
]

resultados = chain.batch(conceitos)

# Stream: recebe tokens conforme são gerados
for chunk in chain.stream({"conceito": "microserviços"}):
    print(chunk, end="", flush=True)
```

## Resumo

| Componente | Uso |
|------------|-----|
| `\|` | Encadear componentes |
| `RunnablePassthrough` | Manter input original |
| `RunnableLambda` | Transformações customizadas |
| `RunnableParallel` | Executar em paralelo |
| `.with_fallbacks()` | Tratamento de erros |
| `.batch()` | Processar múltiplos inputs |
| `.stream()` | Resposta em tempo real |

Cadeias são a base para construir aplicações mais complexas como RAG e Agents.
