# Retrieval

\*Retrieval-Augmented Generation (RAG)\*\*은 외부 지식 베이스에서 관련 정보를 검색하여 LLM의 응답을 향상시키는 기술로 LangChain v1.0은 다양한 검색 패턴과 벡터 스토어를 지원하여 강력한 RAG 시스템을 구축할 수 있다.​&#x20;

***

## 1. RAG 핵심 개념 <a href="#rag" id="rag"></a>

### RAG 파이프라인

RAG는 두 가지 주요 단계로 구성됩니다:​

**1. Indexing (오프라인)**

* **Load**: 문서 로더로 데이터 수집
* **Split**: 텍스트 분할기로 청크 생성
* **Store**: 임베딩 모델로 벡터화 후 벡터 스토어 저장

**2. Retrieval & Generation (런타임)**

* **Retrieve**: 쿼리 기반으로 관련 문서 검색
* **Generate**: 검색된 컨텍스트로 LLM 응답 생성

***

### 1. 설치 및 기본 설정 <a href="#id-1" id="id-1"></a>

```python
# 필수 패키지 설치
pip install --pre -U langchain
pip install -U langchain-openai langchain-ollama
pip install -U langchain-community langchain-text-splitters
pip install -U faiss-cpu  # 또는 faiss-gpu
pip install -U chromadb
pip install -U rank-bm25  # Hybrid search용

# 환경 변수 설정
import os
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"
# Ollama는 로컬에서 실행 (http://localhost:11434)
```

***

## 2. 기본 패턴: Simple RAG (OpenAI) <a href="#id-2---simple-rag-openai" id="id-2---simple-rag-openai"></a>

### 2.1 문서 로드 및 분할

```python
from langchain_community.document_loaders import WebBaseLoader, PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
import bs4

# ===== 방법 1: 웹 문서 로드 =====
loader = WebBaseLoader(
    web_paths=("https://lilianweng.github.io/posts/2023-06-23-agent/",),
    bs_kwargs=dict(
        parse_only=bs4.SoupStrainer(
            class_=("post-content", "post-title", "post-header")
        )
    ),
)
web_docs = loader.load()

# ===== 방법 2: PDF 로드 =====
pdf_loader = PyPDFLoader("document.pdf")
pdf_docs = pdf_loader.load()

# 문서 분할
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    length_function=len,
    is_separator_regex=False,
)

splits = text_splitter.split_documents(web_docs)
print(f"Total {len(splits)} chunks created")
print(f"First chunk: {splits[0].page_content[:200]}...")
```

### 2.2 Vector Store 생성 및 검색

```python
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings

# 임베딩 모델 초기화
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# FAISS Vector Store 생성
vectorstore = FAISS.from_documents(
    documents=splits,
    embedding=embeddings
)

# Retriever 생성
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 4}  # 상위 4개 문서 반환
)

# 검색 테스트
query = "What is agent memory?"
retrieved_docs = retriever.invoke(query)

for i, doc in enumerate(retrieved_docs):
    print(f"\n[Document {i+1}]")
    print(doc.page_content[:200])
```

### 2.3 RAG Chain 구축 (OpenAI)

```python
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# init_chat_model로 OpenAI 모델 초기화
model = init_chat_model("openai:gpt-4o", temperature=0)

# RAG 프롬프트 템플릿
prompt = ChatPromptTemplate.from_messages([
    ("system", """You are an assistant for question-answering tasks. 
Use the following pieces of retrieved context to answer the question. 
If you don't know the answer, just say that you don't know. 
Use three sentences maximum and keep the answer concise.

Context: {context}"""),
    ("human", "{question}")
])

# Helper 함수
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

# RAG Chain 구축
rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# 실행
question = "What are the components of an agent system?"
response = rag_chain.invoke(question)

print(f"Question: {question}")
print(f"\nAnswer: {response}")
```

***

## 3. 기본 패턴: Simple RAG (Ollama) <a href="#id-3---simple-rag-ollama" id="id-3---simple-rag-ollama"></a>

```python
from langchain.chat_models import init_chat_model
from langchain_community.embeddings import OllamaEmbeddings
from langchain_community.vectorstores import Chroma

# init_chat_model로 Ollama 모델 초기화
model = init_chat_model("ollama:llama3.1", temperature=0.7)

# Ollama 임베딩
embeddings = OllamaEmbeddings(model="nomic-embed-text")

# Chroma Vector Store
vectorstore = Chroma.from_documents(
    documents=splits,
    embedding=embeddings,
    collection_name="ollama_rag"
)

retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

# RAG Chain
rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# 실행
response = rag_chain.invoke("Explain task decomposition in agent systems")
print(response)
```

***

## 4. 고급 패턴 1: Hybrid Search (OpenAI) <a href="#id-4---1-hybrid-search-openai" id="id-4---1-hybrid-search-openai"></a>

### 4.1 BM25 + Semantic Search

```python
from langchain.retrievers import BM25Retriever, EnsembleRetriever
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings

# ===== Semantic Retriever (Dense Vector) =====
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(splits, embeddings)
semantic_retriever = vectorstore.as_retriever(search_kwargs={"k": 5})

# ===== BM25 Retriever (Sparse Keyword) =====
bm25_retriever = BM25Retriever.from_documents(splits)
bm25_retriever.k = 5

# ===== Ensemble Retriever (Reciprocal Rank Fusion) =====
ensemble_retriever = EnsembleRetriever(
    retrievers=[semantic_retriever, bm25_retriever],
    weights=[0.5, 0.5],  # 동등한 가중치
    c=60  # RRF 상수
)

# Hybrid RAG Chain
model = init_chat_model("openai:gpt-4o-mini")

hybrid_rag_chain = (
    {"context": ensemble_retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# 테스트
query = "What is self-reflection in agents?"
response = hybrid_rag_chain.invoke(query)
print(response)
```

### 4.2 Hybrid Search with Milvus (Ollama)

```python
from langchain_community.vectorstores import Milvus
from langchain_milvus.utils.sparse import BM25SparseEmbedding

# Ollama 모델
model = init_chat_model("ollama:llama3.1")

# Dense embedding
dense_embeddings = OllamaEmbeddings(model="nomic-embed-text")

# Sparse embedding (BM25)
sparse_embeddings = BM25SparseEmbedding(corpus=splits)

# Milvus with Hybrid Search
vectorstore = Milvus.from_documents(
    documents=splits,
    embedding=dense_embeddings,
    builtin_function=sparse_embeddings,
    vector_field=["dense", "sparse"],
    connection_args={"uri": "./milvus_hybrid.db"}
)

# Hybrid retriever
hybrid_retriever = vectorstore.as_retriever(
    search_type="hybrid",  # dense + sparse
    search_kwargs={"k": 4}
)

# RAG Chain
hybrid_rag_ollama = (
    {"context": hybrid_retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

response = hybrid_rag_ollama.invoke("Explain planning in LLM agents")
print(response)
```

***

## 5. 고급 패턴 2: Parent Document Retriever (OpenAI) <a href="#id-5---2-parent-document-retriever-openai" id="id-5---2-parent-document-retriever-openai"></a>

### 5.1 작은 청크 검색 → 큰 문서 반환

```python
from langchain.retrievers import ParentDocumentRetriever
from langchain.storage import InMemoryStore
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

# 임베딩
embeddings = OpenAIEmbeddings()

# Vector Store
vectorstore = Chroma(
    collection_name="parent_docs",
    embedding_function=embeddings
)

# Storage for parent documents
store = InMemoryStore()

# Child Splitter (작은 청크 - 검색용)
child_splitter = RecursiveCharacterTextSplitter(chunk_size=400, chunk_overlap=50)

# Parent Splitter (큰 청크 - 반환용)
parent_splitter = RecursiveCharacterTextSplitter(chunk_size=2000, chunk_overlap=200)

# Parent Document Retriever
parent_retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=store,
    child_splitter=child_splitter,
    parent_splitter=parent_splitter,  # None이면 전체 문서 반환
)

# 문서 추가
parent_retriever.add_documents(splits)

# RAG Chain
model = init_chat_model("openai:gpt-4o")

parent_rag_chain = (
    {"context": parent_retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# 실행
response = parent_rag_chain.invoke("What is Chain of Thought?")
print(response)
```

***

## 6. 고급 패턴 3: Multi-Vector Retriever (Ollama) <a href="#id-6---3-multi-vector-retriever-ollama" id="id-6---3-multi-vector-retriever-ollama"></a>

### 6.1 요약 기반 검색

```python
import uuid
from langchain.retrievers.multi_vector import MultiVectorRetriever
from langchain.storage import InMemoryByteStore
from langchain_community.vectorstores import FAISS
from langchain.chat_models import init_chat_model
from langchain_core.documents import Document

# Ollama 모델
model = init_chat_model("ollama:llama3.1")
embeddings = OllamaEmbeddings(model="nomic-embed-text")

# Vector Store (요약 저장)
vectorstore = FAISS.from_documents([], embeddings)

# Docstore (원본 문서 저장)
docstore = InMemoryByteStore()

# Multi-Vector Retriever
id_key = "doc_id"
multi_retriever = MultiVectorRetriever(
    vectorstore=vectorstore,
    byte_store=docstore,
    id_key=id_key,
)

# 문서별 요약 생성
doc_ids = [str(uuid.uuid4()) for _ in splits]
summaries = []

summary_prompt = """Summarize the following document in one concise sentence:

{doc}

Summary:"""

for doc in splits:
    summary = model.invoke(summary_prompt.format(doc=doc.page_content))
    summaries.append(summary.content)

# 요약을 Vector Store에, 원본을 Docstore에 저장
summary_docs = [
    Document(page_content=s, metadata={id_key: doc_ids[i]})
    for i, s in enumerate(summaries)
]

multi_retriever.vectorstore.add_documents(summary_docs)
multi_retriever.docstore.mset(list(zip(doc_ids, splits)))

# RAG Chain
multi_rag_chain = (
    {"context": multi_retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# 실행
response = multi_rag_chain.invoke("What is task decomposition?")
print(response)
```

***

## 7. 고급 패턴 4: Self-Query Retriever (OpenAI) <a href="#id-7---4-self-query-retriever-openai" id="id-7---4-self-query-retriever-openai"></a>

### 7.1 메타데이터 기반 필터링

```python
from langchain.chains.query_constructor.schema import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.vectorstores import Chroma
from langchain_core.documents import Document

# 메타데이터가 있는 문서
docs = [
    Document(
        page_content="LangChain v1.0 was released in October 2025",
        metadata={"year": 2025, "month": 10, "category": "release"}
    ),
    Document(
        page_content="Python 3.12 introduced new features",
        metadata={"year": 2023, "month": 10, "category": "programming"}
    ),
    Document(
        page_content="OpenAI GPT-4 launched in March 2023",
        metadata={"year": 2023, "month": 3, "category": "ai"}
    ),
    Document(
        page_content="LangChain supports multiple LLMs",
        metadata={"year": 2024, "month": 5, "category": "framework"}
    ),
]

# Vector Store 생성
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(docs, embeddings)

# 메타데이터 필드 정의
metadata_field_info = [
    AttributeInfo(
        name="year",
        description="The year of the event",
        type="integer",
    ),
    AttributeInfo(
        name="month",
        description="The month of the event (1-12)",
        type="integer",
    ),
    AttributeInfo(
        name="category",
        description="The category: 'release', 'programming', 'ai', 'framework'",
        type="string",
    ),
]

# Self-Query Retriever
document_content_description = "Technical events and releases"
llm = ChatOpenAI(temperature=0, model="gpt-4")

self_query_retriever = SelfQueryRetriever.from_llm(
    llm,
    vectorstore,
    document_content_description,
    metadata_field_info,
    verbose=True
)

# RAG Chain
model = init_chat_model("openai:gpt-4o")

self_query_rag = (
    {"context": self_query_retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# 복잡한 쿼리 테스트
query = "What AI events happened in 2023 between January and June?"
response = self_query_rag.invoke(query)
print(response)
```

***

## 8. 고급 패턴 5: Contextual Compression (Ollama) <a href="#id-8---5-contextual-compression-ollama" id="id-8---5-contextual-compression-ollama"></a>

### 8.1 LLM 기반 문서 압축

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor
from langchain.chat_models import init_chat_model

# Ollama 모델
model = init_chat_model("ollama:llama3.1", temperature=0)

# Base Retriever
embeddings = OllamaEmbeddings(model="nomic-embed-text")
vectorstore = Chroma.from_documents(splits, embeddings)
base_retriever = vectorstore.as_retriever(search_kwargs={"k": 6})

# LLM Compressor
compressor = LLMChainExtractor.from_llm(model)

# Contextual Compression Retriever
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=base_retriever
)

# 압축 전후 비교
query = "What is agent memory?"

print("="*70)
print("Before Compression:")
print("="*70)
base_docs = base_retriever.invoke(query)
for i, doc in enumerate(base_docs[:2]):
    print(f"\n[Document {i+1}] {len(doc.page_content)} chars")
    print(doc.page_content[:300])

print("\n" + "="*70)
print("After Compression:")
print("="*70)
compressed_docs = compression_retriever.invoke(query)
for i, doc in enumerate(compressed_docs[:2]):
    print(f"\n[Document {i+1}] {len(doc.page_content)} chars")
    print(doc.page_content)

# RAG Chain with Compression
compression_rag = (
    {"context": compression_retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

response = compression_rag.invoke(query)
print(f"\n{'='*70}")
print("Final Answer:")
print(response)
```

***

## 9. 고급 패턴 6: Reranking (OpenAI) <a href="#id-9---6-reranking-openai" id="id-9---6-reranking-openai"></a>

### 9.1 Cross-Encoder Reranker

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import CrossEncoderReranker
from langchain_community.cross_encoders import HuggingFaceCrossEncoder

# Base Retriever (많은 문서 검색)
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(splits, embeddings)
base_retriever = vectorstore.as_retriever(search_kwargs={"k": 20})

# Cross-Encoder Reranker
cross_encoder_model = HuggingFaceCrossEncoder(model_name="cross-encoder/ms-marco-MiniLM-L-6-v2")
compressor = CrossEncoderReranker(model=cross_encoder_model, top_n=5)

# Reranking Retriever
rerank_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=base_retriever
)

# RAG Chain
model = init_chat_model("openai:gpt-4o-mini")

rerank_rag = (
    {"context": rerank_retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

response = rerank_rag.invoke("Explain self-reflection in AI agents")
print(response)
```

### 9.2 LLM-based Reranker (Ollama)

```python
from langchain.retrievers.document_compressors import LLMChainFilter

# Ollama 모델
model = init_chat_model("ollama:llama3.1", temperature=0)

# Base Retriever
embeddings = OllamaEmbeddings(model="nomic-embed-text")
vectorstore = Chroma.from_documents(splits, embeddings)
base_retriever = vectorstore.as_retriever(search_kwargs={"k": 15})

# LLM Filter (관련성 낮은 문서 제거)
llm_filter = LLMChainFilter.from_llm(model)

# Filtering Retriever
filter_retriever = ContextualCompressionRetriever(
    base_compressor=llm_filter,
    base_retriever=base_retriever
)

# RAG Chain
filter_rag = (
    {"context": filter_retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

response = filter_rag.invoke("What are the components of an agent?")
print(response)
```

***

## 10. 실전 예제: Complete RAG System <a href="#id-10---complete-rag-system" id="id-10---complete-rag-system"></a>

### 10.1 모든 패턴 통합 (OpenAI + Ollama)

```python
from typing import TypedDict, Literal
from langchain.chat_models import init_chat_model
from langchain_openai import OpenAIEmbeddings
from langchain_community.embeddings import OllamaEmbeddings
from langchain_community.vectorstores import FAISS, Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain.retrievers import BM25Retriever, EnsembleRetriever, ContextualCompressionRetriever
from langchain.retrievers.document_compressors import CrossEncoderReranker
from langchain_community.cross_encoders import HuggingFaceCrossEncoder
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
import bs4

# ===== 1. 문서 로드 및 전처리 =====
print("="*70)
print("Step 1: Loading and Processing Documents")
print("="*70)

loader = WebBaseLoader(
    web_paths=(
        "https://lilianweng.github.io/posts/2023-06-23-agent/",
        "https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/",
    ),
    bs_kwargs=dict(
        parse_only=bs4.SoupStrainer(
            class_=("post-content", "post-title", "post-header")
        )
    ),
)

docs = loader.load()

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)

splits = text_splitter.split_documents(docs)
print(f"Created {len(splits)} chunks")

# ===== 2. Hybrid Vector Stores =====
print("\n" + "="*70)
print("Step 2: Creating Hybrid Vector Stores")
print("="*70)

# OpenAI Embeddings (Semantic)
openai_embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
openai_vectorstore = FAISS.from_documents(splits, openai_embeddings)
openai_semantic_retriever = openai_vectorstore.as_retriever(search_kwargs={"k": 10})

# Ollama Embeddings (Semantic)
ollama_embeddings = OllamaEmbeddings(model="nomic-embed-text")
ollama_vectorstore = Chroma.from_documents(splits, ollama_embeddings, collection_name="complete_rag")
ollama_semantic_retriever = ollama_vectorstore.as_retriever(search_kwargs={"k": 10})

# BM25 (Keyword)
bm25_retriever = BM25Retriever.from_documents(splits)
bm25_retriever.k = 10

# Ensemble Retriever (OpenAI)
openai_ensemble = EnsembleRetriever(
    retrievers=[openai_semantic_retriever, bm25_retriever],
    weights=[0.6, 0.4]  # Semantic에 더 높은 가중치
)

# Ensemble Retriever (Ollama)
ollama_ensemble = EnsembleRetriever(
    retrievers=[ollama_semantic_retriever, bm25_retriever],
    weights=[0.6, 0.4]
)

print("Hybrid retrievers created successfully")

# ===== 3. Reranking =====
print("\n" + "="*70)
print("Step 3: Adding Reranking")
print("="*70)

# Cross-Encoder Reranker
cross_encoder = HuggingFaceCrossEncoder(model_name="cross-encoder/ms-marco-MiniLM-L-6-v2")
reranker = CrossEncoderReranker(model=cross_encoder, top_n=5)

# OpenAI Retriever with Reranking
openai_final_retriever = ContextualCompressionRetriever(
    base_compressor=reranker,
    base_retriever=openai_ensemble
)

# Ollama Retriever with Reranking
ollama_final_retriever = ContextualCompressionRetriever(
    base_compressor=reranker,
    base_retriever=ollama_ensemble
)

print("Reranking enabled")

# ===== 4. LLM 모델 초기화 =====
openai_model = init_chat_model("openai:gpt-4o", temperature=0)
ollama_model = init_chat_model("ollama:llama3.1", temperature=0.7)

# ===== 5. RAG Prompt =====
rag_prompt = ChatPromptTemplate.from_messages([
    ("system", """You are an expert AI assistant specializing in LLM agents and prompt engineering.
Use the provided context to answer questions accurately and concisely.
If the context doesn't contain enough information, acknowledge that and provide your best answer based on general knowledge.

Context:
{context}"""),
    ("human", "{question}")
])

def format_docs(docs):
    return "\n\n".join([f"[Doc {i+1}]: {doc.page_content}" for i, doc in enumerate(docs)])

# ===== 6. RAG Chains =====
openai_rag_chain = (
    {"context": openai_final_retriever | format_docs, "question": RunnablePassthrough()}
    | rag_prompt
    | openai_model
    | StrOutputParser()
)

ollama_rag_chain = (
    {"context": ollama_final_retriever | format_docs, "question": RunnablePassthrough()}
    | rag_prompt
    | ollama_model
    | StrOutputParser()
)

# ===== 7. 테스트 =====
test_queries = [
    "What are the key components of an LLM-powered agent system?",
    "Explain the difference between Chain-of-Thought and Tree-of-Thoughts prompting",
    "How does self-reflection work in agent systems?"
]

print("\n" + "="*70)
print("Step 4: Testing Complete RAG System")
print("="*70)

for i, query in enumerate(test_queries):
    print(f"\n{'='*70}")
    print(f"Query {i+1}: {query}")
    print(f"{'='*70}")
    
    # OpenAI
    print("\n[OpenAI GPT-4o Response]")
    openai_response = openai_rag_chain.invoke(query)
    print(openai_response)
    
    # Ollama
    print("\n[Ollama Llama3.1 Response]")
    ollama_response = ollama_rag_chain.invoke(query)
    print(ollama_response)
```

***

## 11. Best Practices <a href="#id-11-best-practices" id="id-11-best-practices"></a>

### RAG 시스템 최적화

1. **청크 크기 조정**: 도메인에 따라 `chunk_size`와 `chunk_overlap` 최적화​
2. **Hybrid Search 사용**: Semantic + Keyword 검색으로 정확도 향상​
3. **Reranking 적용**: 검색 결과를 재정렬하여 relevance 개선​
4. **메타데이터 활용**: 시간, 카테고리 등으로 필터링​
5. **Contextual Compression**: 토큰 비용 절감 및 컨텍스트 품질 향상​

### Vector Store 선택

| 용도            | 추천 Vector Store          | 특징                |
| ------------- | ------------------------ | ----------------- |
| 로컬 개발         | FAISS, Chroma            | 빠른 프로토타이핑         |
| 프로덕션          | Milvus, Pinecone, Qdrant | 확장성, 성능           |
| Hybrid Search | Milvus, Elasticsearch    | Dense + Sparse 지원 |
| 관계 데이터        | Neo4j                    | 그래프 기반 검색         |

### Embedding 모델 선택

| 모델                              | 용도     | 특징          |
| ------------------------------- | ------ | ----------- |
| OpenAI `text-embedding-3-small` | 범용, 빠름 | 저렴, 높은 품질   |
| OpenAI `text-embedding-3-large` | 고품질 필요 | 더 정확, 비용 높음 |
| Ollama `nomic-embed-text`       | 로컬, 무료 | 완전 로컬 실행    |
| HuggingFace SBERT               | 다국어 지원 | 다양한 언어      |
