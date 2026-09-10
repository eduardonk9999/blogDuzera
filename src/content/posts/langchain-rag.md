---
title: "LangChain: RAG (Retrieval Augmented Generation)"
date: 2026-09-10
description: "Como criar aplicações que consultam documentos usando RAG"
tags: ["python", "ia", "langchain", "llm", "rag"]
draft: false
---

## O que é RAG?

RAG (Retrieval Augmented Generation) combina busca de documentos com geração de texto. Em vez de confiar apenas no conhecimento do modelo, você fornece contexto relevante.

```
Pergunta → Busca documentos → Passa contexto + pergunta → LLM responde
```

## Por que usar RAG?

- **Dados atualizados** - Informações além do corte de conhecimento do modelo
- **Dados privados** - Documentos internos da empresa
- **Precisão** - Respostas baseadas em fontes específicas
- **Citações** - Pode referenciar de onde veio a informação

## Arquitetura básica

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Documentos │ →  │  Embeddings │ →  │ Vector Store│
└─────────────┘    └─────────────┘    └─────────────┘
                                            ↓
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Resposta  │ ←  │     LLM     │ ←  │  Retriever  │
└─────────────┘    └─────────────┘    └─────────────┘
```

## Instalação

```bash
pip install langchain langchain-openai langchain-chroma
```

## Passo 1: Carregar documentos

```python
from langchain_community.document_loaders import (
    TextLoader,
    PyPDFLoader,
    WebBaseLoader
)

# Texto simples
loader = TextLoader("documento.txt")
docs = loader.load()

# PDF
loader = PyPDFLoader("relatorio.pdf")
docs = loader.load()

# Página web
loader = WebBaseLoader("https://exemplo.com/artigo")
docs = loader.load()
```

## Passo 2: Dividir em chunks

Documentos grandes precisam ser divididos:

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)

chunks = splitter.split_documents(docs)
print(f"{len(chunks)} chunks criados")
```

## Passo 3: Criar embeddings

Embeddings convertem texto em vetores numéricos:

```python
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# Teste
vetor = embeddings.embed_query("O que é Python?")
print(f"Dimensões: {len(vetor)}")  # 1536
```

## Passo 4: Armazenar em vector store

```python
from langchain_chroma import Chroma

# Cria vector store com os chunks
vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# Busca por similaridade
resultados = vectorstore.similarity_search("sua pergunta", k=3)
```

## Passo 5: Criar retriever

```python
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

# Busca documentos relevantes
docs = retriever.invoke("O que é machine learning?")
```

## Passo 6: Montar a cadeia RAG

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

llm = ChatOpenAI(model="gpt-4o-mini")

# Prompt com contexto
prompt = ChatPromptTemplate.from_template("""
Responda a pergunta baseado apenas no contexto fornecido.

Contexto:
{context}

Pergunta: {question}

Resposta:
""")

def formatar_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

# Cadeia RAG
rag_chain = (
    {"context": retriever | formatar_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

# Pergunta
resposta = rag_chain.invoke("Qual o tema principal do documento?")
print(resposta)
```

## Exemplo completo

```python
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.document_loaders import WebBaseLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# 1. Carrega
loader = WebBaseLoader("https://pt.wikipedia.org/wiki/Python")
docs = loader.load()

# 2. Divide
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = splitter.split_documents(docs)

# 3. Embeddings + Vector Store
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(chunks, embeddings)
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

# 4. LLM + Prompt
llm = ChatOpenAI(model="gpt-4o-mini")
prompt = ChatPromptTemplate.from_template("""
Contexto: {context}

Pergunta: {question}

Responda de forma clara e concisa:
""")

def formatar_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

# 5. Cadeia RAG
rag = (
    {"context": retriever | formatar_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

# 6. Usa
print(rag.invoke("Quem criou o Python?"))
```

## Vector Stores alternativos

```python
# FAISS (local, rápido)
from langchain_community.vectorstores import FAISS
vectorstore = FAISS.from_documents(chunks, embeddings)

# Pinecone (cloud, escalável)
from langchain_pinecone import PineconeVectorStore
vectorstore = PineconeVectorStore.from_documents(
    chunks, embeddings, index_name="meu-indice"
)

# Weaviate
from langchain_weaviate import WeaviateVectorStore
vectorstore = WeaviateVectorStore.from_documents(
    chunks, embeddings, client=client
)
```

## Estratégias de retrieval

### Similarity Search (padrão)

```python
retriever = vectorstore.as_retriever(search_type="similarity")
```

### MMR (Maximum Marginal Relevance)

Diversifica resultados:

```python
retriever = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={"k": 5, "fetch_k": 10}
)
```

### Threshold

Filtra por score mínimo:

```python
retriever = vectorstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={"score_threshold": 0.7}
)
```

## Multi-Query Retriever

Gera múltiplas versões da pergunta para melhor busca:

```python
from langchain.retrievers.multi_query import MultiQueryRetriever

retriever_mq = MultiQueryRetriever.from_llm(
    retriever=vectorstore.as_retriever(),
    llm=llm
)
```

## Parent Document Retriever

Busca chunks, retorna documento completo:

```python
from langchain.retrievers import ParentDocumentRetriever
from langchain.storage import InMemoryStore

store = InMemoryStore()

retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=store,
    child_splitter=RecursiveCharacterTextSplitter(chunk_size=200),
    parent_splitter=RecursiveCharacterTextSplitter(chunk_size=1000)
)
```

## RAG com citações

```python
from langchain_core.prompts import ChatPromptTemplate

prompt_com_citacao = ChatPromptTemplate.from_template("""
Responda a pergunta baseado no contexto.
Inclua citações entre [1], [2], etc.

Contexto:
{context}

Pergunta: {question}

Resposta (com citações):
""")

def formatar_com_indice(docs):
    return "\n\n".join(
        f"[{i+1}] {doc.page_content}"
        for i, doc in enumerate(docs)
    )

rag_citacoes = (
    {"context": retriever | formatar_com_indice, "question": RunnablePassthrough()}
    | prompt_com_citacao
    | llm
    | StrOutputParser()
)
```

## Reranking

Reordena resultados para maior precisão:

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain_cohere import CohereRerank

compressor = CohereRerank(model="rerank-english-v3.0")

retriever_rerank = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=vectorstore.as_retriever(search_kwargs={"k": 10})
)
```

## Resumo

| Componente | Função |
|------------|--------|
| Document Loader | Carrega documentos |
| Text Splitter | Divide em chunks |
| Embeddings | Texto → vetores |
| Vector Store | Armazena e busca |
| Retriever | Interface de busca |
| RAG Chain | Junta tudo |

RAG é a forma mais comum de dar conhecimento específico a LLMs sem fine-tuning.
