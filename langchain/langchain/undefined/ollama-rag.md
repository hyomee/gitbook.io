# Ollama RAG 시스템

## 1. Ollama 설치 및 설정

```bash
# macOS
brew install ollama
brew services start ollama

# Linux
curl -fsSL https://ollama.com/install.sh | sh

# Windows (WSL 사용)
curl -fsSL https://ollama.com/install.sh | sh

```

## 2. 모델 다운로드

```bash
# 채팅 모델
ollama pull llama3.2
ollama pull llama3.1
ollama pull qwen2.5

# 임베딩 모델
ollama pull nomic-embed-text
ollama pull mxbai-embed-large
ollama pull all-minilm

# 모델 목록 확인
ollama list

# 모델 실행 테스트
ollama run llama3.2

```

## 3. Ollama RAG 시스템

```python
from langchain.chat_models import init_chat_model
from langchain.embeddings import init_embeddings
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# 1. 문서 로드
loader = PyPDFLoader("document.pdf")
docs = loader.load()

# 2. 텍스트 분할
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
splits = text_splitter.split_documents(docs)

# 3. Ollama 벡터 스토어 생성
embeddings_ollama = init_embeddings("nomic-embed-text", provider="ollama")
vectorstore_ollama = Chroma.from_documents(
    documents=splits,
    embedding=embeddings_ollama
)
retriever_ollama = vectorstore_ollama.as_retriever(search_kwargs={"k": 3})

# 5. Ollama 모델 초기화
model_ollama = init_chat_model("llama3.2", model_provider="ollama", temperature=0)

# 6. RAG Chain 구성
rag_chain_ollama = (
    {"context": retriever_ollama | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model_ollama
    | StrOutputParser()
)

# 7. 실행
answer_ollama = rag_chain_ollama.invoke("LangChain v1.0의 주요 변경사항은 무엇인가요?")
print("Ollama RAG 답변:", answer_ollama)

```


