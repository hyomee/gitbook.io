# Langchain

LangChain은 LLM 기반 에이전트와 애플리케이션 구축을 시작하는 가장 쉬운 방법으로 10줄 미만의 코드로 OpenAI, Anthropic, Google 등에 연결할 수 있다.

* 철학: [https://docs.langchain.com/oss/python/langchain/philosophy](https://docs.langchain.com/oss/python/langchain/philosophy)
* Release Notes: [https://docs.langchain.com/oss/python/releases/langchain-v1](https://docs.langchain.com/oss/python/releases/langchain-v1)
* Migration Guide: [https://docs.langchain.com/oss/python/migrate/langchain-v1](https://docs.langchain.com/oss/python/migrate/langchain-v1)

## 1. 주요기능

* **다양한 LLM 통합:** OpenAI의 GPT, Anthropic의 Claude, Google의 PaLM 등 여러 LLM과 원활하게 연동할 수 있는 인터페이스를 제공한다.
* **외부 데이터 소스 연결:** 데이터베이스, API, 파일 시스템 등 다양한 데이터 소스에서 실시간으로 데이터를 가져와 모델의 응답에 활용할 수 있게 한다.
* **체인(Chains):** 여러 AI 구성 요소를 순차적으로 연결해 복잡한 비즈니스 로직을 처리하는 워크플로를 자동화한다.
* **에이전트(Agents):** 사용자 요청에 따라 필요한 도구를 스스로 결정하고 실행하는 기능을 제공합니다. 이를 통해 언어 모델이 인터넷 검색과 같은 외부 환경과 상호작용하며 최신 정보를 기반으로 답변을 생성할 수 있다.
* **메모리(Memory):** 대화형 애플리케이션에서 이전 상호작용의 정보를 기억하고 활용하여 연속적인 대화를 가능하게 한다.
* **검색 증강 생성(RAG):** 외부 데이터를 활용해 LLM의 응답을 보강하는 기술로, 모델의 '환각(hallucination)' 문제를 줄이고 더 정확한 답변을 생성하도록 돕는다.
* **LangGraph:** LangChain의 확장 기능으로, 그래프 기반의 구조를 사용해 복잡한 조건부 로직과 다중 에이전트 협업을 구현할 수 있다.

## 2. 구성요소

### 2-1. Models

LLM(대규모 언어 모델)이나 Embedding 모델을 추상화한 객체로. LangChain에서 모델과 상호작용하는 주요 방식은 **invoke**(단일 호출), **stream**(실시간 스트리밍), **batch**(배치 처리)가 있다

```python
from langchain_openai import ChatOpenAI

# 모델 초기화
model = ChatOpenAI(
    model="gpt-4o",
    temperature=0.7,
    max_tokens=1000,
    timeout=30
)

# 단일 호출
response = model.invoke("LangChain이 무엇인가요?")
print(response.content)

# 메시지 형식으로 호출
messages = [
    ("system", "당신은 친절한 AI 어시스턴트입니다."),
    ("human", "Python에 대해 설명해주세요.")
]
response = model.invoke(messages)
print(response.content)
```

**LangChain v1.0에서는 \*\*`init_chat_model`\*\*을 사용하여 통합된 방식으로 모델을 초기화 한다**

#### 2-1-1. 기본 모델 초기화

```python
from langchain.chat_models import init_chat_model

# OpenAI 모델
model = init_chat_model("gpt-4o", model_provider="openai", temperature=0)

# Ollama 모델 (로컬)
ollama_model = init_chat_model("llama3.2", model_provider="ollama", temperature=0)

# Anthropic 모델
claude = init_chat_model("claude-3-5-sonnet-latest", model_provider="anthropic")

# 모델 호출 - OpenAI
response = model.invoke("LangChain v1.0에 대해 설명해주세요")
print(response.content)

# 모델 호출 - Ollama
ollama_response = ollama_model.invoke("LangChain v1.0에 대해 설명해주세요")
print(ollama_response.content)

```

#### 2-1-2. 메시지 형식으로 호출

```python
from langchain.messages import HumanMessage, SystemMessage

messages = [
    SystemMessage(content="당신은 친절한 AI 어시스턴트입니다."),
    HumanMessage(content="Python에 대해 설명해주세요.")
]

# OpenAI로 호출
response_openai = model.invoke(messages)
print("OpenAI:", response_openai.content)

# Ollama로 호출
response_ollama = ollama_model.invoke(messages)
print("Ollama:", response_ollama.content)

```

#### 2-1-3. 런타임 구성 가능 모델

```python
# 실행 시점에 모델 변경 가능
configurable_model = init_chat_model(temperature=0)

# OpenAI 사용
result1 = configurable_model.invoke(
    "안녕하세요",
    config={"configurable": {"model": "gpt-4o"}}
)

# Ollama 사용
result2 = configurable_model.invoke(
    "안녕하세요",
    config={"configurable": {"model": "llama3.2", "model_provider": "ollama"}}
)

```

#### 2-1-4. 임베딩 모델

```python
from langchain.embeddings import init_embeddings

# OpenAI 임베딩 모델
embeddings_openai = init_embeddings("text-embedding-3-small", provider="openai")

# Ollama 임베딩 모델
embeddings_ollama = init_embeddings("nomic-embed-text", provider="ollama")

# 텍스트 임베딩 - OpenAI
text_embedding_openai = embeddings_openai.embed_query("LangChain v1.0")

# 텍스트 임베딩 - Ollama
text_embedding_ollama = embeddings_ollama.embed_query("LangChain v1.0")

# 문서 임베딩
docs_embeddings = embeddings_openai.embed_documents(["문서1", "문서2"])

```

### 2-2. Prompts

**Prompts**는 모델에 전달할 입력을 구조화하고 템플릿화하는 컴포넌트로 변수 삽입, 다중 메시지 구성, Few-shot 예제 포함 등이 가능합니다

#### 2-2-1.  기본 프롬프트 템플릿

```python
from langchain_core.prompts import PromptTemplate

# 단순 템플릿
prompt = PromptTemplate.from_template(
    "{subject}에 대해 {style} 스타일로 설명해주세요."
)
formatted = prompt.format(subject="AI", style="초보자가 이해할 수 있는")
print(formatted)
```

#### 2-2-2. Chat 프롬프트 템플릿

```python
from langchain_core.prompts import ChatPromptTemplate

chat_prompt = ChatPromptTemplate.from_messages([
    ("system", "당신은 {role} 전문가입니다."),
    ("human", "{question}"),
    ("ai", "네, 도와드리겠습니다."),
    ("human", "{follow_up}")
])

messages = chat_prompt.format_messages(
    role="Python 개발",
    question="FastAPI에 대해 알려주세요",
    follow_up="장점은 무엇인가요?"
)

```

#### 2-2-3. Few-Shot 프롬프트

```python
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate

# 예제 데이터
examples = [
    {"input": "happy", "output": "sad"},
    {"input": "tall", "output": "short"},
    {"input": "sunny", "output": "gloomy"}
]

# 예제 포맷 템플릿
example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="입력: {input}\n출력: {output}"
)

# Few-shot 프롬프트 생성
few_shot_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    suffix="입력: {word}\n출력:",
    input_variables=["word"]
)

print(few_shot_prompt.format(word="energetic"))

```

#### 2-2-4. LCEL을 사용한 프롬프트 체인

```python
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 프롬프트 정의
prompt = ChatPromptTemplate.from_messages([
    ("system", "당신은 {role} 전문가입니다."),
    ("human", "{question}")
])

# OpenAI 모델 체인
model_openai = init_chat_model("gpt-4o", temperature=0)
chain_openai = prompt | model_openai | StrOutputParser()

# Ollama 모델 체인
model_ollama = init_chat_model("llama3.2", model_provider="ollama", temperature=0)
chain_ollama = prompt | model_ollama | StrOutputParser()

# 실행
result_openai = chain_openai.invoke({
    "role": "Python 개발",
    "question": "비동기 프로그래밍을 설명해주세요"
})

result_ollama = chain_ollama.invoke({
    "role": "Python 개발",
    "question": "비동기 프로그래밍을 설명해주세요"
})

print("OpenAI:", result_openai)
print("Ollama:", result_ollama)

```

### 2-3. Output Parsers

**Output Parsers**는 LLM의 문자열 출력을 구조화된 데이터(JSON, Pydantic 객체 등)로 변환한다.

#### 2-3-1. PydanticOutputParser

```python
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import PromptTemplate
from langchain.chat_models import init_chat_model
from pydantic import BaseModel, Field

# Pydantic 모델 정의
class BookInfo(BaseModel):
    title: str = Field(description="책 제목")
    author: str = Field(description="저자 이름")
    year: int = Field(description="출판 연도")

# 파서 생성
parser = PydanticOutputParser(pydantic_object=BookInfo)

# 프롬프트에 포맷 지시사항 추가
prompt = PromptTemplate(
    template="다음 책 정보를 제공해주세요: {book}\n\n{format_instructions}",
    input_variables=["book"],
    partial_variables={"format_instructions": parser.get_format_instructions()}
)

# OpenAI 체인
model_openai = init_chat_model("gpt-4o", temperature=0)
chain_openai = prompt | model_openai | parser

# Ollama 체인
model_ollama = init_chat_model("llama3.2", model_provider="ollama", temperature=0)
chain_ollama = prompt | model_ollama | parser

# 실행 - OpenAI
result_openai = chain_openai.invoke({"book": "해리포터"})
print(f"OpenAI - 제목: {result_openai.title}, 저자: {result_openai.author}, 연도: {result_openai.year}")

# 실행 - Ollama
result_ollama = chain_ollama.invoke({"book": "해리포터"})
print(f"Ollama - 제목: {result_ollama.title}, 저자: {result_ollama.author}, 연도: {result_ollama.year}")

```

#### 2-3-2. JsonOutputParser

```python
from langchain_core.output_parsers import JsonOutputParser

parser = JsonOutputParser()

# JSON 문자열 파싱
json_string = '{"name": "철수", "age": 25, "city": "서울"}'
parsed = parser.parse(json_string)
print(parsed)  # {'name': '철수', 'age': 25, 'city': '서울'}



```

#### 2-3-3. StructuredOutputParser

```python
from langchain.output_parsers import StructuredOutputParser, ResponseSchema

# 응답 스키마 정의
response_schemas = [
    ResponseSchema(name="answer", description="질문에 대한 답변"),
    ResponseSchema(name="confidence", description="답변의 확신도 (0-100)")
]

parser = StructuredOutputParser.from_response_schemas(response_schemas)
format_instructions = parser.get_format_instructions()

# 프롬프트에 포함
prompt = PromptTemplate(
    template="질문: {query}\n\n{format_instructions}",
    input_variables=["query"],
    partial_variables={"format_instructions": format_instructions}
)
```

### 2-4. Document Loaders, TextSplites, Vector Stores

이 컴포넌트들은 **RAG(Retrieval-Augmented Generation)** 구현에 핵심적요소 이다.

#### 2-4-1. Document Loaders

```python
from langchain_community.document_loaders import PyPDFLoader

# PDF 로더
loader = PyPDFLoader("document.pdf")
pages = loader.load()

# 페이지별 내용 확인
for i, page in enumerate(pages):
    print(f"페이지 {i+1}: {page.page_content[:100]}...")
    print(f"메타데이터: {page.metadata}")

```

#### 2-4-2. TextSplitters

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

# RecursiveCharacterTextSplitter (권장)
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,        # 청크 최대 크기
    chunk_overlap=200,      # 청크 간 중복 문자 수
    length_function=len,
    separators=["\n\n", "\n", " ", ""]  # 분할 우선순위
)

# 문서 분할
documents = text_splitter.split_documents(pages)
print(f"총 {len(documents)}개 청크 생성")

```

#### 2-4-3. Vector Stores

```python
from langchain_chroma import Chroma
from langchain.embeddings import init_embeddings

# OpenAI 임베딩 사용
embeddings_openai = init_embeddings("text-embedding-3-small", provider="openai")
vectorstore_openai = Chroma.from_documents(
    documents=documents,
    embedding=embeddings_openai,
    persist_directory="./chroma_db_openai"
)

# Ollama 임베딩 사용
embeddings_ollama = init_embeddings("nomic-embed-text", provider="ollama")
vectorstore_ollama = Chroma.from_documents(
    documents=documents,
    embedding=embeddings_ollama,
    persist_directory="./chroma_db_ollama"
)

# 유사도 검색 - OpenAI
query = "LangChain이란?"
results_openai = vectorstore_openai.similarity_search(query, k=3)
print("OpenAI 검색 결과:")
for doc in results_openai:
    print(doc.page_content)

# 유사도 검색 - Ollama
results_ollama = vectorstore_ollama.similarity_search(query, k=3)
print("\nOllama 검색 결과:")
for doc in results_ollama:
    print(doc.page_content)

# Retriever로 사용
retriever_openai = vectorstore_openai.as_retriever(search_kwargs={"k": 3})
retriever_ollama = vectorstore_ollama.as_retriever(search_kwargs={"k": 3})

```

#### 2-4-4. FAISS 사용 예시

```python
from langchain_community.vectorstores import FAISS
from langchain.embeddings import init_embeddings

# OpenAI 임베딩
embeddings_openai = init_embeddings("text-embedding-3-small", provider="openai")
vectorstore_faiss_openai = FAISS.from_documents(documents, embeddings_openai)

# Ollama 임베딩
embeddings_ollama = init_embeddings("nomic-embed-text", provider="ollama")
vectorstore_faiss_ollama = FAISS.from_documents(documents, embeddings_ollama)

# 저장 및 로드
vectorstore_faiss_openai.save_local("faiss_index_openai")
vectorstore_faiss_ollama.save_local("faiss_index_ollama")

new_vectorstore_openai = FAISS.load_local(
    "faiss_index_openai", 
    embeddings_openai,
    allow_dangerous_deserialization=True
)

```

### 2-5. Tools

**Tools**는 에이전트가 외부 기능(검색, 계산, API 호출 등)을 실행할 수 있도록 연결하는 함수이다.

#### 2-5-1. @tool 데코레이터 사용

```python
from langchain.tools import tool

@tool
def search_database(query: str, limit: int = 10) -> str:
    """데이터베이스에서 쿼리와 일치하는 레코드를 검색합니다.
    
    Args:
        query: 검색어
        limit: 반환할 최대 결과 수
    """
    # 실제 검색 로직
    return f"{limit}개의 결과를 찾았습니다: {query}"

# Tool 정보 확인
print(search_database.name)
print(search_database.description)

# Tool 실행
result = search_database.invoke({"query": "Python 튜토리얼", "limit": 5})
print(result)

```

#### 2-5-2. 커스텀 Tool 이름과 스키마

```python
from langchain.tools import tool
from pydantic import BaseModel, Field

class CalculatorInput(BaseModel):
    expression: str = Field(description="계산할 수식")

@tool("calculator", args_schema=CalculatorInput)
def calculate(expression: str) -> str:
    """수학 표현식을 계산합니다."""
    try:
        result = eval(expression)
        return f"결과: {result}"
    except:
        return "계산 오류"

print(calculate.invoke({"expression": "2 + 3 * 4"}))
```

#### 2-5-3. 여러 Tool 정의

```python
from langchain.tools import tool

@tool
def get_current_weather(location: str) -> str:
    """특정 위치의 현재 날씨를 가져옵니다."""
    return f"{location}의 날씨: 맑음, 23도"

@tool
def get_word_length(word: str) -> int:
    """문자열의 길이를 반환합니다."""
    return len(word)

@tool
def multiply(a: int, b: int) -> int:
    """두 숫자를 곱합니다."""
    return a * b

@tool
def add(a: int, b: int) -> int:
    """두 숫자를 더합니다."""
    return a + b

tools = [search_database, 
         calculate, 
         get_current_weather, 
         get_word_length, 
         multiply, 
         add]

```

### 2-5. Agent

**Agent**는 LLM을 사용하여 어떤 행동(Tool 호출)을 취할지 결정하는 시스템으로  LangChain v1.0에서는 \*\*`langchain.agents.create_agent`\*\*를 사용한다.

#### 2-5-1. 기본 Agent 생성

```python
from langchain.agents import create_agent
from langchain.tools import tool

# Tool 정의
@tool
def multiply(a: int, b: int) -> int:
    """두 숫자를 곱합니다."""
    return a * b

@tool
def add(a: int, b: int) -> int:
    """두 숫자를 더합니다."""
    return a + b

tools = [multiply, add]

# OpenAI Agent
agent_openai = create_agent(
    model="openai:gpt-4o",
    tools=tools,
    system_prompt="당신은 수학 문제를 단계별로 해결하는 전문가입니다."
)

# Ollama Agent
agent_ollama = create_agent(
    model="ollama:llama3.2",
    tools=tools,
    system_prompt="당신은 수학 문제를 단계별로 해결하는 전문가입니다."
)

# Agent 실행 - OpenAI
result_openai = agent_openai.invoke({
    "messages": [{"role": "user", "content": "5와 3을 곱한 후 7을 더하면?"}]
})
print("OpenAI:", result_openai["messages"][-1].content)

# Agent 실행 - Ollama
result_ollama = agent_ollama.invoke({
    "messages": [{"role": "user", "content": "5와 3을 곱한 후 7을 더하면?"}]
})
print("Ollama:", result_ollama["messages"][-1].content)

```

#### 2-5-2. 메모리가 있는 Agent (Checkpointing)

```python
from langchain.agents import create_agent
from langgraph.checkpoint.memory import MemorySaver

# Checkpointer 생성
memory = MemorySaver()

# OpenAI Agent (메모리 포함)
agent_openai = create_agent(
    model="openai:gpt-4o",
    tools=tools,
    system_prompt="당신은 친절한 어시스턴트입니다.",
    checkpointer=memory
)

# Ollama Agent (메모리 포함)
agent_ollama = create_agent(
    model="ollama:llama3.2",
    tools=tools,
    system_prompt="당신은 친절한 어시스턴트입니다.",
    checkpointer=memory
)

# 대화 세션 설정
config_openai = {"configurable": {"thread_id": "openai-session-123"}}
config_ollama = {"configurable": {"thread_id": "ollama-session-456"}}

# 첫 번째 대화 - OpenAI
response1_openai = agent_openai.invoke(
    {"messages": [{"role": "user", "content": "안녕, 내 이름은 철수야"}]},
    config_openai
)

# 두 번째 대화 - OpenAI (이전 대화 기억)
response2_openai = agent_openai.invoke(
    {"messages": [{"role": "user", "content": "내 이름이 뭐였지?"}]},
    config_openai
)
print("OpenAI:", response2_openai["messages"][-1].content)

# 첫 번째 대화 - Ollama
response1_ollama = agent_ollama.invoke(
    {"messages": [{"role": "user", "content": "안녕, 내 이름은 영희야"}]},
    config_ollama
)

# 두 번째 대화 - Ollama (이전 대화 기억)
response2_ollama = agent_ollama.invoke(
    {"messages": [{"role": "user", "content": "내 이름이 뭐였지?"}]},
    config_ollama
)
print("Ollama:", response2_ollama["messages"][-1].content)

```

#### 2-5-3. Context를 활용한 Agent

```python
from dataclasses import dataclass
from langchain.agents import create_agent

# Context 스키마 정의
@dataclass
class Context:
    user_id: str
    session_id: str
    user_level: str = "beginner"

# OpenAI Agent
agent_openai = create_agent(
    model="openai:gpt-4o",
    tools=tools,
    system_prompt="당신은 사용자 수준에 맞춰 설명하는 튜터입니다.",
    context_schema=Context
)

# Ollama Agent
agent_ollama = create_agent(
    model="ollama:llama3.2",
    tools=tools,
    system_prompt="당신은 사용자 수준에 맞춰 설명하는 튜터입니다.",
    context_schema=Context
)

# Context와 함께 실행 - OpenAI
result_openai = agent_openai.invoke(
    {"messages": [{"role": "user", "content": "비동기 프로그래밍을 설명해줘"}]},
    context=Context(user_id="123", session_id="abc", user_level="expert")
)

# Context와 함께 실행 - Ollama
result_ollama = agent_ollama.invoke(
    {"messages": [{"role": "user", "content": "비동기 프로그래밍을 설명해줘"}]},
    context=Context(user_id="456", session_id="def", user_level="beginner")
)

```

#### 2-5-4. Agent 스트리밍

```python
from langchain.agents import create_agent

# OpenAI Agent 스트리밍
agent_openai = create_agent(
    model="openai:gpt-4o",
    tools=tools,
    system_prompt="당신은 실시간으로 응답하는 어시스턴트입니다."
)

print("OpenAI 스트리밍:")
for chunk in agent_openai.stream(
    {"messages": [{"role": "user", "content": "3 곱하기 5는?"}]},
    stream_mode="updates"
):
    print(chunk)
    print("---")

# Ollama Agent 스트리밍
agent_ollama = create_agent(
    model="ollama:llama3.2",
    tools=tools,
    system_prompt="당신은 실시간으로 응답하는 어시스턴트입니다."
)

print("\nOllama 스트리밍:")
for chunk in agent_ollama.stream(
    {"messages": [{"role": "user", "content": "3 곱하기 5는?"}]},
    stream_mode="updates"
):
    print(chunk)
    print("---")

```

### 2-6. Example Selects

**Example Selectors**는 프롬프트에 포함할 예제를 동적으로 선택하는 컴포넌트이다.

#### 2-6-1. SemanticSimilarityExampleSelector

```python
from langchain_chroma import Chroma
from langchain_core.example_selectors import SemanticSimilarityExampleSelector
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate
from langchain.embeddings import init_embeddings

# 예제 정의
examples = [
    {"input": "happy", "output": "sad"},
    {"input": "tall", "output": "short"},
    {"input": "energetic", "output": "lethargic"},
    {"input": "sunny", "output": "gloomy"},
    {"input": "windy", "output": "calm"}
]

# 예제 포맷 템플릿
example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="입력: {input}\n출력: {output}"
)

# OpenAI 임베딩 사용
embeddings_openai = init_embeddings("text-embedding-3-small", provider="openai")
example_selector_openai = SemanticSimilarityExampleSelector.from_examples(
    examples,
    embeddings_openai,
    Chroma,
    k=2
)

# Ollama 임베딩 사용
embeddings_ollama = init_embeddings("nomic-embed-text", provider="ollama")
example_selector_ollama = SemanticSimilarityExampleSelector.from_examples(
    examples,
    embeddings_ollama,
    Chroma,
    k=2
)

# Few-shot 프롬프트 - OpenAI
similar_prompt_openai = FewShotPromptTemplate(
    example_selector=example_selector_openai,
    example_prompt=example_prompt,
    prefix="다음 입력의 반대말을 제시하세요",
    suffix="입력: {adjective}\n출력:",
    input_variables=["adjective"]
)

# Few-shot 프롬프트 - Ollama
similar_prompt_ollama = FewShotPromptTemplate(
    example_selector=example_selector_ollama,
    example_prompt=example_prompt,
    prefix="다음 입력의 반대말을 제시하세요",
    suffix="입력: {adjective}\n출력:",
    input_variables=["adjective"]
)

# 실행
print("OpenAI 임베딩 기반 선택:")
print(similar_prompt_openai.format(adjective="excited"))

print("\nOllama 임베딩 기반 선택:")
print(similar_prompt_ollama.format(adjective="excited"))

```

#### 2-6-2. LengthBasedExampleSelector

```python
from langchain_core.example_selectors import LengthBasedExampleSelector

examples = [
    {"input": "안녕", "output": "반가워"},
    {"input": "좋은 아침입니다", "output": "좋은 아침이에요"},
    {"input": "오늘 날씨가 정말 좋네요", "output": "네, 화창하네요"}
]

example_selector = LengthBasedExampleSelector(
    examples=examples,
    example_prompt=example_prompt,
    max_length=25
)

dynamic_prompt = FewShotPromptTemplate(
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="대화 예시:",
    suffix="입력: {input}\n출력:",
    input_variables=["input"]
)

print(dynamic_prompt.format(input="안녕하세요"))

```
