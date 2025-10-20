# OpenAI RAG 시스템

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

# 3. OpenAI 벡터 스토어 생성
embeddings_openai = init_embeddings("text-embedding-3-small", provider="openai")
vectorstore_openai = Chroma.from_documents(
    documents=splits,
    embedding=embeddings_openai
)
retriever_openai = vectorstore_openai.as_retriever(search_kwargs={"k": 3})

# 4. 프롬프트 템플릿
prompt = ChatPromptTemplate.from_messages([
    ("system", "다음 컨텍스트를 기반으로 질문에 답변하세요:\n\n{context}"),
    ("human", "{question}")
])

# 5. OpenAI 모델 초기화
model_openai = init_chat_model("gpt-4o", temperature=0)

# 6. RAG Chain 구성
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

rag_chain_openai = (
    {"context": retriever_openai | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model_openai
    | StrOutputParser()
)

# 7. 실행
answer_openai = rag_chain_openai.invoke("LangChain v1.0의 주요 변경사항은 무엇인가요?")
print("OpenAI RAG 답변:", answer_openai)
```
