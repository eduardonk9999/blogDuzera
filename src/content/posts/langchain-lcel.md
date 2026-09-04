---
title: "LangChain: LCEL (LangChain Expression Language)"
date: 2026-09-03
description: "Domine a sintaxe declarativa do LangChain para compor cadeias de forma elegante"
tags: ["python", "ia", "langchain", "llm"]
draft: true
---

## O que é LCEL?

LCEL (LangChain Expression Language) é a sintaxe declarativa do LangChain para compor cadeias. Usa o operador `|` (pipe) para conectar componentes, similar ao pipe do Unix.

```python
cadeia = componente1 | componente2 | componente3
```

## Por que usar LCEL?

- **Streaming** - Tokens são enviados assim que disponíveis
- **Async** - Suporte nativo a operações assíncronas
- **Batch** - Processa múltiplos inputs em paralelo
- **Retry** - Retentativas automáticas com backoff
- **Fallbacks** - Alternativas quando algo falha

## Runnables: Os blocos de construção

Todo componente LCEL implementa a interface `Runnable`:

```python
# Métodos principais de um Runnable
runnable.invoke(input)        # Síncrono
await runnable.ainvoke(input) # Assíncrono
runnable.batch([inputs])      # Múltiplos inputs
runnable.stream(input)        # Streaming
```

## Exemplo completo

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template(
    "Você é um poeta. Escreva um haiku sobre {tema}."
)
llm = ChatOpenAI(model="gpt-4o-mini")
parser = StrOutputParser()

# Composição com LCEL
cadeia = prompt | llm | parser

# Invoke
resultado = cadeia.invoke({"tema": "programação"})
print(resultado)
```

## RunnablePassthrough

Passa o input adiante sem modificação. Útil para construir dicionários:

```python
from langchain_core.runnables import RunnablePassthrough

cadeia = (
    {"tema": RunnablePassthrough(), "estilo": lambda _: "minimalista"}
    | prompt
    | llm
    | parser
)

resultado = cadeia.invoke("café")
```

## RunnableLambda

Transforma qualquer função em um Runnable:

```python
from langchain_core.runnables import RunnableLambda

def processar(texto: str) -> str:
    return texto.strip().lower()

cadeia = (
    prompt
    | llm
    | parser
    | RunnableLambda(processar)
)
```

## RunnableParallel

Executa múltiplos runnables em paralelo:

```python
from langchain_core.runnables import RunnableParallel

analise = RunnableParallel(
    resumo=prompt_resumo | llm | parser,
    keywords=prompt_keywords | llm | parser,
    sentimento=prompt_sentimento | llm | parser
)

# Retorna dict com todas as chaves
resultado = analise.invoke({"texto": "..."})
```

## Branching com RunnableBranch

Lógica condicional baseada no input:

```python
from langchain_core.runnables import RunnableBranch

roteador = RunnableBranch(
    (lambda x: "código" in x["pergunta"], cadeia_tecnica),
    (lambda x: "receita" in x["pergunta"], cadeia_culinaria),
    cadeia_geral  # fallback
)

resultado = roteador.invoke({"pergunta": "Como fazer bolo?"})
```

## Configuração dinâmica com configurable

```python
from langchain_core.runnables import ConfigurableField

llm_configuravel = ChatOpenAI(model="gpt-4o-mini").configurable_fields(
    temperature=ConfigurableField(id="temp")
)

cadeia = prompt | llm_configuravel | parser

# Usa temperatura padrão
cadeia.invoke({"tema": "sol"})

# Configura temperatura diferente
cadeia.with_config(configurable={"temp": 0.9}).invoke({"tema": "sol"})
```

## Bind: Passando argumentos extras

```python
from langchain_core.tools import tool

@tool
def pesquisar(query: str) -> str:
    """Pesquisa na web."""
    return f"Resultados para: {query}"

# Bind anexa tools ao modelo
llm_com_tools = llm.bind_tools([pesquisar])

cadeia = prompt | llm_com_tools
```

## Assign: Adicionando chaves ao contexto

```python
from langchain_core.runnables import RunnablePassthrough

cadeia = (
    RunnablePassthrough.assign(
        tamanho=lambda x: len(x["texto"]),
        maiusculo=lambda x: x["texto"].upper()
    )
    | prompt
    | llm
)

# Input: {"texto": "hello"}
# Passa: {"texto": "hello", "tamanho": 5, "maiusculo": "HELLO"}
```

## Debugging com callbacks

```python
from langchain_core.callbacks import StdOutCallbackHandler

cadeia.invoke(
    {"tema": "python"},
    config={"callbacks": [StdOutCallbackHandler()]}
)
```

## Resumo dos Runnables

| Runnable | Uso |
|----------|-----|
| `RunnablePassthrough` | Passa input sem modificar |
| `RunnableLambda` | Função → Runnable |
| `RunnableParallel` | Execução paralela |
| `RunnableBranch` | Lógica condicional |
| `.bind()` | Anexar argumentos |
| `.assign()` | Adicionar chaves |
| `.configurable_fields()` | Config dinâmica |

LCEL é a base de tudo no LangChain moderno. Dominar essa sintaxe abre portas para RAG, Agents e aplicações complexas.
