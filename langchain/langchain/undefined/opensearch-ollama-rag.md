# OpenSearch(Ollama RAG)

### 1. 패키지 설치

```bash
# LangChain v1.0 핵심 패키지
pip install -U langchain langchain-core langchain-community

# Ollama 통합 패키지
pip install -U langchain-ollama

# OpenSearch 벡터 스토어
pip install -U opensearch-py

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

## 3. OpenSearch 설치 (Docker)

```bash
# Docker로 OpenSearch 실행
docker run -d \
  --name opensearch-node \
  -p 9200:9200 \
  -p 9600:9600 \
  -e "discovery.type=single-node" \
  -e "OPENSEARCH_INITIAL_ADMIN_PASSWORD=Admin@123" \
  -e "plugins.security.disabled=true" \
  opensearchproject/opensearch:latest

# OpenSearch 연결 테스트
curl http://localhost:9200

```

## 4. OpenSearch를 사용한 Ollama RAG

### 4-1. 기본 구현

```python
from langchain_community.document_loaders import PyPDFLoader, TextLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import OpenSearchVectorSearch
from langchain_ollama import OllamaEmbeddings, ChatOllama
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# ====================
# 1. 문서 로드
# ====================
print("📄 문서 로드 중...")

# PDF 파일 로드
loader = PyPDFLoader("sample_document.pdf")
# 또는 텍스트 파일
# loader = TextLoader("sample_document.txt", encoding="utf-8")

documents = loader.load()
print(f"✅ {len(documents)}개 페이지 로드 완료")

# ====================
# 2. 텍스트 분할
# ====================
print("\n✂️  텍스트 분할 중...")

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    length_function=len,
    separators=["\n\n", "\n", ". ", " ", ""]
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

# 임베딩 테스트
test_embedding = embeddings.embed_query("테스트")
print(f"✅ 임베딩 차원: {len(test_embedding)}차원")

# ====================
# 4. OpenSearch 벡터 스토어 생성
# ====================
print("\n🔍 OpenSearch 벡터 스토어 생성 중...")

vectorstore = OpenSearchVectorSearch.from_documents(
    documents=splits,
    embedding=embeddings,
    opensearch_url="http://localhost:9200",
    http_auth=("admin", "Admin@123"),
    use_ssl=False,
    verify_certs=False,
    ssl_assert_hostname=False,
    ssl_show_warn=False,
    index_name="ollama_rag_documents",
    engine="nmslib",  # "nmslib", "faiss", "lucene"
    space_type="l2",  # "l2", "cosinesimil", "innerproduct"
    ef_construction=256,
    m=48
)

print("✅ OpenSearch 벡터 스토어 생성 완료")

# ====================
# 5. Retriever 생성
# ====================
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

# ====================
# 6. Ollama LLM 초기화
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
# 7. RAG 프롬프트 템플릿
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
# 8. RAG Chain 구성
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
# 9. 질의 실행
# ====================
print("\n" + "="*50)
print("💬 RAG 시스템 준비 완료!")
print("="*50)

# 예제 질문
questions = [
    "문서의 주요 내용은 무엇인가요?",
    "문서에서 다루는 핵심 개념을 설명해주세요",
    "문서에 언급된 예시나 사례가 있나요?"
]

for question in questions:
    print(f"\n❓ 질문: {question}")
    print("-" * 50)
    
    # 답변 생성
    answer = rag_chain.invoke(question)
    print(f"💡 답변: {answer}")
    
    # 검색된 문서 확인
    retrieved_docs = retriever.invoke(question)
    print(f"\n📚 참조 문서 수: {len(retrieved_docs)}")
    
    for i, doc in enumerate(retrieved_docs, 1):
        print(f"\n[문서 {i}]")
        print(f"출처: {doc.metadata.get('source', 'Unknown')}")
        print(f"내용: {doc.page_content[:150]}...")
    
    print("\n" + "="*50)

```

### 4-2. OpenSearch 고급 기능

```python
from langchain_community.vectorstores import OpenSearchVectorSearch
from langchain_ollama import OllamaEmbeddings

# ====================
# 1. 다양한 검색 엔진 설정
# ====================

embeddings = OllamaEmbeddings(model="nomic-embed-text")

# FAISS 엔진 (더 빠른 검색)
vectorstore_faiss = OpenSearchVectorSearch.from_documents(
    documents=splits,
    embedding=embeddings,
    opensearch_url="http://localhost:9200",
    http_auth=("admin", "Admin@123"),
    use_ssl=False,
    verify_certs=False,
    index_name="faiss_index",
    engine="faiss",
    space_type="innerproduct",  # 내적 유사도
    ef_construction=256,
    m=48
)

# Lucene 엔진 (표준 검색)
vectorstore_lucene = OpenSearchVectorSearch.from_documents(
    documents=splits,
    embedding=embeddings,
    opensearch_url="http://localhost:9200",
    http_auth=("admin", "Admin@123"),
    use_ssl=False,
    verify_certs=False,
    index_name="lucene_index",
    engine="lucene"
)

# ====================
# 2. 하이브리드 검색 (벡터 + 키워드)
# ====================

# 벡터 검색
vector_results = vectorstore.similarity_search(
    query="LangChain RAG",
    k=5
)

# MMR 검색 (다양성 보장)
mmr_results = vectorstore.max_marginal_relevance_search(
    query="LangChain RAG",
    k=3,
    fetch_k=10,
    lambda_mult=0.5  # 0: 다양성 우선, 1: 유사도 우선
)

# ====================
# 3. 메타데이터 필터링
# ====================

# 특정 카테고리만 검색
filter_query = {
    "bool": {
        "filter": [
            {"term": {"metadata.category.keyword": "technical"}},
            {"range": {"metadata.page": {"gte": 10, "lte": 20}}}
        ]
    }
}

filtered_results = vectorstore.similarity_search(
    query="검색 쿼리",
    k=5,
    pre_filter=filter_query
)

# ====================
# 4. 유사도 점수와 함께 검색
# ====================

results_with_scores = vectorstore.similarity_search_with_score(
    query="검색 쿼리",
    k=5
)

for doc, score in results_with_scores:
    print(f"점수: {score:.4f}")
    print(f"내용: {doc.page_content[:100]}...")
    print("-" * 50)

# ====================
# 5. 벡터 직접 검색
# ====================

# 쿼리 임베딩 생성
query_vector = embeddings.embed_query("검색 쿼리")

# 벡터로 직접 검색
results = vectorstore.similarity_search_by_vector(
    embedding=query_vector,
    k=5
)

# ====================
# 6. 문서 추가 및 삭제
# ====================

# 새 문서 추가
new_docs = [
    Document(
        page_content="새로운 문서 내용",
        metadata={"category": "technical", "page": 1}
    )
]

vectorstore.add_documents(new_docs)

# 인덱스에서 모든 문서 삭제 (주의!)
# vectorstore.delete(index_name="ollama_rag_documents")

```

### 4-3. RAG 시스템

```python
from typing import List, Literal
from langchain_community.document_loaders import PyPDFLoader, TextLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import OpenSearchVectorSearch
from langchain_ollama import OllamaEmbeddings, ChatOllama
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_core.documents import Document
import os

class OllamaRAGSystem:
    """
    Ollama 기반 RAG 시스템
    - OpenSearch  
    - 문서 로드, 분할, 임베딩, 검색, 질의응답 통합
    """
    
    def __init__(
        self,
        vector_db_type: Literal["opensearch" ] = "opensearch",
        embedding_model: str = "nomic-embed-text",
        llm_model: str = "llama3.2",
        ollama_base_url: str = "http://localhost:11434",
        chunk_size: int = 1000,
        chunk_overlap: int = 200
    ):
        """
        Args:
            vector_db_type: 벡터 DB 유형 ("opensearch"  )
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
        
        self.vectorstore = OpenSearchVectorSearch.from_documents(
                documents=documents,
                embedding=self.embeddings,
                opensearch_url=kwargs.get("opensearch_url", "http://localhost:9200"),
                http_auth=kwargs.get("http_auth", ("admin", "Admin@123")),
                use_ssl=False,
                verify_certs=False,
                ssl_assert_hostname=False,
                ssl_show_warn=False,
                index_name=kwargs.get("index_name", "ollama_rag_index"),
                engine=kwargs.get("engine", "nmslib"),
                space_type=kwargs.get("space_type", "l2")
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
    
    # OpenSearch 사용
    print("\n" + "="*60)
    print("OpenSearch RAG 시스템")
    print("="*60)
    
    rag_os = OllamaRAGSystem(
        vector_db_type="opensearch",
        embedding_model="nomic-embed-text",
        llm_model="llama3.2"
    )
    
    rag_os.create_vectorstore(
        splits,
        opensearch_url="http://localhost:9200",
        index_name="my_rag_index",
        engine="faiss",
        search_kwargs={"k": 3}
    )
    
    answer = rag_os.query("LangChain v1.0의 변경사항은?")
    print(f"\n💡 답변:\n{answer}")

```
