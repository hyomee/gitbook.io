# PGVector(Ollama RAG)

### 1. 패키지 설치

```bash
# LangChain v1.0 핵심 패키지
pip install -U langchain langchain-core langchain-community

# Ollama 통합 패키지
pip install -U langchain-ollama

# PGVector 벡터 스토어
pip install -U langchain-postgres psycopg[binary]

# 문서 처리
pip install -U langchain-text-splitters

# PDF 로더 (선택사항)
pip install -U pypdf

# 기타 유틸리티
pip install python-dotenv

```

### 2. Ollama 설치 및 모델 준비

```bash
# macOS
brew install ollama
brew services start ollama

# Linux
curl -fsSL https://ollama.com/install.sh | sh

# Ollama 모델 다운로드
ollama pull llama3.2        # 채팅 모델
ollama pull nomic-embed-text  # 임베딩 모델 (768차원)
ollama pull mxbai-embed-large # 대안 임베딩 모델 (1024차원)

# 모델 확인
ollama list

# Ollama 서버 실행 확인
curl http://localhost:11434/api/tags

```

## 3. PostgreSQL + PGVector 설치 (Docker)

```bash
# Docker로 PostgreSQL + pgvector 실행
docker run -d \
  --name pgvector-container \
  -e POSTGRES_USER=langchain \
  -e POSTGRES_PASSWORD=langchain \
  -e POSTGRES_DB=langchain \
  -p 5432:5432 \
  pgvector/pgvector:pg16

# PostgreSQL 연결 테스트
psql -h localhost -U langchain -d langchain


```

## 4. PGVector를 사용한 Ollama RAG

### 4-1. 기본 구현

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_postgres import PGVector
from langchain_ollama import OllamaEmbeddings, ChatOllama
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# ====================
# 1. 문서 로드
# ====================
print("📄 문서 로드 중...")

loader = PyPDFLoader("sample_document.pdf")
documents = loader.load()
print(f"✅ {len(documents)}개 페이지 로드 완료")

# ====================
# 2. 텍스트 분할
# ====================
print("\n✂️  텍스트 분할 중...")

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    length_function=len
)

splits = text_splitter.split_documents(documents)
print(f"✅ {len(splits)}개 청크로 분할 완료")

# ====================
# 3. Ollama 임베딩 초기화
# ====================
print("\n🔢 임베딩 모델 초기화 중...")

embeddings = OllamaEmbeddings(
    model="nomic-embed-text",
    base_url="http://localhost:11434"
)

test_embedding = embeddings.embed_query("테스트")
print(f"✅ 임베딩 차원: {len(test_embedding)}차원")

# ====================
# 4. PostgreSQL 연결 문자열
# ====================
# 주의: LangChain v1.0에서는 postgresql+psycopg:// 사용
CONNECTION_STRING = "postgresql+psycopg://langchain:langchain@localhost:5432/langchain"

# ====================
# 5. PGVector 벡터 스토어 생성
# ====================
print("\n🐘 PGVector 벡터 스토어 생성 중...")

vectorstore = PGVector.from_documents(
    documents=splits,
    embedding=embeddings,
    collection_name="ollama_documents",
    connection=CONNECTION_STRING,
    use_jsonb=True,  # 메타데이터를 JSONB로 저장
    pre_delete_collection=False  # 기존 컬렉션 유지
)

print("✅ PGVector 벡터 스토어 생성 완료")

# ====================
# 6. Retriever 생성
# ====================
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

# ====================
# 7. Ollama LLM 초기화
# ====================
print("\n🤖 Ollama LLM 초기화 중...")

llm = ChatOllama(
    model="llama3.2",
    base_url="http://localhost:11434",
    temperature=0.0,
    num_predict=512
)

print("✅ Ollama LLM 초기화 완료")

# ====================
# 8. RAG 프롬프트 템플릿
# ====================
template = """당신은 문서 기반 질의응답 전문가입니다.
주어진 컨텍스트를 기반으로 정확하고 상세하게 답변하세요.
컨텍스트에 답이 없으면 "주어진 문서에서 해당 정보를 찾을 수 없습니다"라고 답변하세요.

컨텍스트:
{context}

질문: {question}

답변:"""

prompt = ChatPromptTemplate.from_template(template)

# ====================
# 9. RAG Chain 구성
# ====================
def format_docs(docs):
    """검색된 문서를 문자열로 포맷팅"""
    return "\n\n".join(doc.page_content for doc in docs)

rag_chain = (
    {
        "context": retriever | format_docs,
        "question": RunnablePassthrough()
    }
    | prompt
    | llm
    | StrOutputParser()
)

# ====================
# 10. 질의 실행
# ====================
print("\n" + "="*50)
print("💬 RAG 시스템 준비 완료!")
print("="*50)

# 예제 질문
questions = [
    "문서의 주요 내용은 무엇인가요?",
    "문서에서 다루는 핵심 개념을 설명해주세요"
]

for question in questions:
    print(f"\n❓ 질문: {question}")
    print("-" * 50)
    
    # 답변 생성
    answer = rag_chain.invoke(question)
    print(f"💡 답변: {answer}")
    
    # 검색된 문서와 유사도 점수
    results_with_scores = vectorstore.similarity_search_with_score(
        query=question,
        k=3
    )
    
    print(f"\n📚 참조 문서:")
    for i, (doc, score) in enumerate(results_with_scores, 1):
        print(f"\n[문서 {i}] (유사도: {score:.4f})")
        print(f"출처: {doc.metadata.get('source', 'Unknown')}")
        print(f"내용: {doc.page_content[:150]}...")
    
    print("\n" + "="*50)


```

### 4-2. PGVector 고급 기능

```python
from langchain_postgres import PGVector
from langchain_ollama import OllamaEmbeddings
from langchain_core.documents import Document

embeddings = OllamaEmbeddings(model="nomic-embed-text")
CONNECTION_STRING = "postgresql+psycopg://langchain:langchain@localhost:5432/langchain"

# ====================
# 1. 기존 컬렉션 연결
# ====================

# 새 벡터 스토어 인스턴스 
vectorstore = PGVector(
    embeddings=embeddings,
    collection_name="ollama_documents",
    connection=CONNECTION_STRING,
    use_jsonb=True
)

# ====================
# 2. 문서 ID 지정하여 추가
# ====================

docs_with_ids = [
    Document(
        page_content="새로운 문서 1",
        metadata={"category": "tech", "author": "홍길동", "id": 1}
    ),
    Document(
        page_content="새로운 문서 2",
        metadata={"category": "science", "author": "김철수", "id": 2}
    )
]

# ID 지정하여 추가 (업데이트 가능)
ids = vectorstore.add_documents(docs_with_ids, ids=["doc_1", "doc_2"])
print(f"추가된 문서 IDs: {ids}")

# ====================
# 3. 메타데이터 필터링 검색
# ====================

# 같음 (Equal)
results = vectorstore.similarity_search(
    query="검색 쿼리",
    k=5,
    filter={"category": {"$eq": "tech"}}
)

# 포함 (In)
results = vectorstore.similarity_search(
    query="검색 쿼리",
    k=5,
    filter={"id": {"$in": [1, 2, 3, 5]}}
)

# 크거나 같음 (Greater Than or Equal)
results = vectorstore.similarity_search(
    query="검색 쿼리",
    k=5,
    filter={"id": {"$gte": 10}}
)

# 작거나 같음 (Less Than or Equal)
results = vectorstore.similarity_search(
    query="검색 쿼리",
    k=5,
    filter={"id": {"$lte": 100}}
)

# 논리 AND
results = vectorstore.similarity_search(
    query="검색 쿼리",
    k=5,
    filter={
        "$and": [
            {"category": {"$eq": "tech"}},
            {"id": {"$gte": 10}},
            {"id": {"$lte": 100}}
        ]
    }
)

# 논리 OR
results = vectorstore.similarity_search(
    query="검색 쿼리",
    k=5,
    filter={
        "$or": [
            {"category": {"$eq": "tech"}},
            {"category": {"$eq": "science"}},
            {"author": {"$eq": "홍길동"}}
        ]
    }
)

# LIKE 검색 (부분 일치)
results = vectorstore.similarity_search(
    query="검색 쿼리",
    k=5,
    filter={"author": {"$like": "%홍%"}}
)

# 복합 조건
results = vectorstore.similarity_search(
    query="검색 쿼리",
    k=5,
    filter={
        "$and": [
            {
                "$or": [
                    {"category": {"$eq": "tech"}},
                    {"category": {"$eq": "science"}}
                ]
            },
            {"id": {"$gte": 10}}
        ]
    }
)

# ====================
# 4. MMR 검색 (다양성 보장)
# ====================

retriever_mmr = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 3,           # 최종 반환 문서 수
        "fetch_k": 10,    # 초기 검색 문서 수
        "lambda_mult": 0.5  # 0: 다양성 우선, 1: 유사도 우선
    }
)

mmr_results = retriever_mmr.invoke("검색 쿼리")

# ====================
# 5. 유사도 임계값 설정
# ====================

retriever_threshold = vectorstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={
        "score_threshold": 0.8,  # 최소 유사도 점수
        "k": 5
    }
)

threshold_results = retriever_threshold.invoke("검색 쿼리")

# ====================
# 6. 문서 삭제
# ====================

# ID로 삭제
vectorstore.delete(ids=["doc_1", "doc_2"])

# 전체 컬렉션 삭제 (주의!)
# vectorstore.delete_collection()

# ====================
# 7. 통계 정보 확인
# ====================

# PostgreSQL에서 직접 쿼리
import psycopg

conn = psycopg.connect(
    "host=localhost dbname=langchain user=langchain password=langchain"
)

with conn.cursor() as cur:
    # 컬렉션별 문서 수
    cur.execute("""
        SELECT name, COUNT(*) as doc_count
        FROM langchain_pg_collection c
        JOIN langchain_pg_embedding e ON c.uuid = e.collection_id
        GROUP BY c.name
    """)
    
    results = cur.fetchall()
    for name, count in results:
        print(f"컬렉션 '{name}': {count}개 문서")

conn.close()

```

### 4-3. RAG 시스템

```python
from typing import List, Literal
from langchain_community.document_loaders import PyPDFLoader, TextLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_postgres import PGVector 
from langchain_ollama import OllamaEmbeddings, ChatOllama
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_core.documents import Document
import os

class OllamaRAGSystem:
    """
    Ollama 기반 RAG 시스템
    - OpenSearch 또는 PGVector 선택 가능
    - 문서 로드, 분할, 임베딩, 검색, 질의응답 통합
    """
    
    def __init__(
        self,
        vector_db_type: Literal["pgvector"] = "pgvector",
        embedding_model: str = "nomic-embed-text",
        llm_model: str = "llama3.2",
        ollama_base_url: str = "http://localhost:11434",
        chunk_size: int = 1000,
        chunk_overlap: int = 200
    ):
        """
        Args:
            vector_db_type: 벡터 DB 유형 ( "pgvector")
            embedding_model: Ollama 임베딩 모델
            llm_model: Ollama LLM 모델
            ollama_base_url: Ollama 서버 URL
            chunk_size: 텍스트 분할 크기
            chunk_overlap: 청크 간 중복 크기
        """
        self.vector_db_type = vector_db_type
        self.embedding_model = embedding_model
        self.llm_model = llm_model
        self.ollama_base_url = ollama_base_url
        self.chunk_size = chunk_size
        self.chunk_overlap = chunk_overlap
        
        # 임베딩 초기화
        self.embeddings = OllamaEmbeddings(
            model=self.embedding_model,
            base_url=self.ollama_base_url
        )
        
        # LLM 초기화
        self.llm = ChatOllama(
            model=self.llm_model,
            base_url=self.ollama_base_url,
            temperature=0.0
        )
        
        # 텍스트 분할기 초기화
        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=self.chunk_size,
            chunk_overlap=self.chunk_overlap,
            length_function=len,
            separators=["\n\n", "\n", ". ", " ", ""]
        )
        
        self.vectorstore = None
        self.retriever = None
        self.rag_chain = None
    
    def load_documents(self, file_paths: List[str]) -> List[Document]:
        """다양한 형식의 문서 로드"""
        all_docs = []
        
        for file_path in file_paths:
            if not os.path.exists(file_path):
                print(f"⚠️  파일 없음: {file_path}")
                continue
            
            ext = os.path.splitext(file_path)[1].lower()
            
            try:
                if ext == ".pdf":
                    loader = PyPDFLoader(file_path)
                elif ext in [".txt", ".md"]:
                    loader = TextLoader(file_path, encoding="utf-8")
                else:
                    print(f"⚠️  지원하지 않는 파일 형식: {file_path}")
                    continue
                
                docs = loader.load()
                all_docs.extend(docs)
                print(f"✅ 로드 완료: {file_path} ({len(docs)}개 문서)")
                
            except Exception as e:
                print(f"❌ 로드 실패: {file_path} - {str(e)}")
        
        return all_docs
    
    def split_documents(self, documents: List[Document]) -> List[Document]:
        """문서 분할"""
        splits = self.text_splitter.split_documents(documents)
        print(f"✂️  총 {len(splits)}개 청크로 분할 완료")
        return splits
    
    def create_vectorstore(self, documents: List[Document], **kwargs):
        """벡터 스토어 생성"""
        print(f"\n🔍 {self.vector_db_type.upper()} 벡터 스토어 생성 중...")
        
        connection_string = kwargs.get(
            "connection_string",
            "postgresql+psycopg://langchain:langchain@localhost:5432/langchain"
        )
        
        self.vectorstore = PGVector.from_documents(
            documents=documents,
            embedding=self.embeddings,
            collection_name=kwargs.get("collection_name", "ollama_documents"),
            connection=connection_string,
            use_jsonb=True
        )
        
        print(f"✅ {self.vector_db_type.upper()} 벡터 스토어 생성 완료")
        
        # Retriever 생성
        self.retriever = self.vectorstore.as_retriever(
            search_type=kwargs.get("search_type", "similarity"),
            search_kwargs=kwargs.get("search_kwargs", {"k": 3})
        )
        
        # RAG Chain 구성
        self._setup_rag_chain()
    
    def _setup_rag_chain(self):
        """RAG Chain 설정"""
        template = """당신은 문서 기반 질의응답 전문가입니다.
주어진 컨텍스트를 기반으로 정확하고 상세하게 답변하세요.
컨텍스트에 답이 없으면 "주어진 문서에서 해당 정보를 찾을 수 없습니다"라고 답변하세요.

컨텍스트:
{context}

질문: {question}

답변:"""
        
        prompt = ChatPromptTemplate.from_template(template)
        
        def format_docs(docs):
            return "\n\n".join(doc.page_content for doc in docs)
        
        self.rag_chain = (
            {
                "context": self.retriever | format_docs,
                "question": RunnablePassthrough()
            }
            | prompt
            | self.llm
            | StrOutputParser()
        )
    
    def query(self, question: str, return_sources: bool = False):
        """RAG 질의"""
        if not self.rag_chain:
            raise ValueError("벡터 스토어가 초기화되지 않았습니다. create_vectorstore()를 먼저 호출하세요.")
        
        # 답변 생성
        answer = self.rag_chain.invoke(question)
        
        if return_sources:
            # 유사도 점수와 함께 검색
            if self.vector_db_type == "pgvector":
                sources = self.vectorstore.similarity_search_with_score(question, k=3)
            else:
                # OpenSearch는 similarity_search만 지원
                sources = self.retriever.invoke(question)
            
            return answer, sources
        
        return answer
    
    def add_documents(self, documents: List[Document]):
        """새 문서 추가"""
        if not self.vectorstore:
            raise ValueError("벡터 스토어가 초기화되지 않았습니다.")
        
        splits = self.split_documents(documents)
        self.vectorstore.add_documents(splits)
        print(f"✅ {len(splits)}개 청크 추가 완료")


# ====================
# 사용 예제
# ====================

if __name__ == "__main__":
    # PGVector 사용
    print("="*60)
    print("PGVector RAG 시스템")
    print("="*60)
    
    rag_pg = OllamaRAGSystem(
        vector_db_type="pgvector",
        embedding_model="nomic-embed-text",
        llm_model="llama3.2"
    )
    
    # 문서 로드
    docs = rag_pg.load_documents([
        "document1.pdf",
        "document2.txt"
    ])
    
    # 문서 분할
    splits = rag_pg.split_documents(docs)
    
    # 벡터 스토어 생성
    rag_pg.create_vectorstore(
        splits,
        connection_string="postgresql+psycopg://langchain:langchain@localhost:5432/langchain",
        collection_name="my_documents",
        search_kwargs={"k": 3}
    )
    
    # 질의
    answer, sources = rag_pg.query(
        "문서의 주요 내용은 무엇인가요?",
        return_sources=True
    )
    
    print(f"\n💡 답변:\n{answer}")
    print(f"\n📚 참조 문서 수: {len(sources)}")
    
    for i, (doc, score) in enumerate(sources, 1):
        print(f"\n[문서 {i}] 유사도: {score:.4f}")
        print(f"내용: {doc.page_content[:200]}...")
    
  

```
