# Model

LangChain v1.0에서 `init_chat_model`을 사용하여  다양한 모델 제공자를 단일 인터페이스로 초기화할 수 있게 해주는 핵심 기능.​

### 주요 패턴 정리 <a href="#undefined" id="undefined"></a>

1. **init\_chat\_model**: 모든 모델 제공자를 통일된 인터페이스로 초기화​
2. **invoke/stream/batch**: 동기/비동기 실행 메서드​
3. **LCEL 체인**: `|` 연산자로 간단한 체인 구성​
4. **Prompt Templates**: 재사용 가능한 프롬프트​
5. **Structured Output**: Pydantic으로 스키마 정의​
6. **Tool Calling**: 함수를 도구로 바인딩​
7. **RAG**: 검색 증강 생성 구현​
8. **Agent**: ReAct 패턴으로 자율 행동​
9. **Configurable Model**: 런타임에 모델 전환​
10. **Memory**: 대화 히스토리 관리​

## 1. 기본 초기화 및 설정 <a href="#id-1" id="id-1"></a>

### 1-1. 환경 설정

먼저 필요한 패키지를 설치합니다:​

```bash
pip install -qU langchain langchain-openai langchain-ollama
```

API 키 설정 (OpenAI):​

```python
import os
from dotenv import load_dotenv

load_dotenv()

# 환경 변수로 API 키 설정
os.environ["OPENAI_API_KEY"] = "your_api_key_here"
```

### 1-2. init\_chat\_model 기본 사용법

**OpenAI 모델 초기화**:​

```python
from langchain.chat_models import init_chat_model

# 방법 1: model_provider 명시
llm_openai = init_chat_model(
    "gpt-4o", 
    model_provider="openai", 
    temperature=0
)

# 방법 2: 자동 추론 (gpt-로 시작하면 openai 자동 인식)
llm_openai = init_chat_model("gpt-4o", temperature=0)

# 방법 3: provider:model 형식
llm_openai = init_chat_model("openai:gpt-4o", temperature=0)
```

**Ollama 모델 초기화**:​

```python
# Ollama 로컬 모델 (Ollama 서버가 실행 중이어야 함)
llm_ollama = init_chat_model(
    "llama3.1:8b",
    model_provider="ollama",
    temperature=0.7
)

# base_url 커스터마이징 (기본값: http://localhost:11434)
llm_ollama = init_chat_model(
    "llama3.1:8b",
    model_provider="ollama",
    base_url="http://localhost:11434",
    temperature=0.7
)
```

### 1-3. 지원 파라미터:​

```python
llm = init_chat_model(
    model="gpt-4o",                    # 모델 이름
    model_provider="openai",           # 제공자
    temperature=0.7,                   # 창의성 조절 (0.0-1.0)
    max_tokens=1000,                   # 최대 출력 토큰
    timeout=30,                        # 타임아웃 (초)
    max_retries=3,                     # 재시도 횟수
    # base_url="custom_url",           # API 엔드포인트
    # api_key="your_key"               # API 키
)
```

## 2. 핵심 실행 패턴 <a href="#id-2" id="id-2"></a>

### invoke - 기본 호출:​

```python
from langchain.chat_models import init_chat_model

llm = init_chat_model("gpt-4o", temperature=0)

# 단순 문자열 입력
response = llm.invoke("Python의 장점을 설명해주세요")
print(response.content)

# 메시지 객체로 입력
from langchain_core.messages import HumanMessage, SystemMessage

messages = [
    SystemMessage(content="당신은 전문 개발자입니다."),
    HumanMessage(content="비동기 프로그래밍을 설명해주세요")
]
response = llm.invoke(messages)
print(response.content)
```

### stream - 실시간 스트리밍:​

```python
# 동기 스트리밍
llm = init_chat_model("gpt-4o", model_provider="openai")

for chunk in llm.stream("LangChain에 대해 설명해주세요"):
    print(chunk.content, end="", flush=True)

# 비동기 스트리밍
import asyncio

async def async_stream():
    llm = init_chat_model("gpt-4o", model_provider="openai")
    async for chunk in llm.astream("머신러닝이란 무엇인가요?"):
        print(chunk.content, end="", flush=True)

asyncio.run(async_stream())
```

### batch - 배치 처리:​

```python
llm = init_chat_model("gpt-4o", model_provider="openai")

# 여러 질문을 동시에 처리
questions = [
    "Python의 특징은?",
    "JavaScript의 특징은?",
    "Java의 특징은?"
]

# 동기 배치
responses = llm.batch(questions)
for i, response in enumerate(responses):
    print(f"Q{i+1}: {questions[i]}")
    print(f"A{i+1}: {response.content}\n")

# 비동기 배치
async def async_batch():
    responses = await llm.abatch(questions)
    for response in responses:
        print(response.content)

asyncio.run(async_batch())
```

## 3. 프롬프트 템플릿과 체인 (LCEL) <a href="#id-3----lcel" id="id-3----lcel"></a>

### 기본 체인 구성:​

```python
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

llm = init_chat_model("gpt-4o", temperature=0)

# 프롬프트 템플릿
prompt = ChatPromptTemplate.from_messages([
    ("system", "당신은 {role} 전문가입니다."),
    ("human", "{question}")
])

# LCEL 체인: prompt | llm | parser
chain = prompt | llm | StrOutputParser()

# 실행
result = chain.invoke({
    "role": "Python 프로그래밍",
    "question": "리스트 컴프리헨션을 설명해주세요"
})
print(result)

# 스트리밍
for chunk in chain.stream({
    "role": "데이터 과학",
    "question": "판다스의 주요 기능은?"
}):
    print(chunk, end="", flush=True)
```

### 복잡한 프롬프트 구성:​

```python
from langchain_core.prompts import (
    ChatPromptTemplate,
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate
)

system_template = "당신은 {language} 전문가입니다."
human_template = "{topic}에 대해 {style} 스타일로 설명해주세요"

prompt = ChatPromptTemplate.from_messages([
    SystemMessagePromptTemplate.from_template(system_template),
    HumanMessagePromptTemplate.from_template(human_template)
])

chain = prompt | llm | StrOutputParser()

result = chain.invoke({
    "language": "Python",
    "topic": "제너레이터",
    "style": "초보자도 이해할 수 있게"
})
```

## 4. 대화 메모리 (Conversation Memory) <a href="#id-4---conversation-memory" id="id-4---conversation-memory"></a>

### MessagesPlaceholder 사용:​

```python
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.messages import HumanMessage, AIMessage

llm = init_chat_model("gpt-4o", temperature=0)

# 대화 히스토리를 포함하는 프롬프트
prompt = ChatPromptTemplate.from_messages([
    ("system", "당신은 친절한 AI 어시스턴트입니다."),
    MessagesPlaceholder("chat_history"),
    ("human", "{question}")
])

chain = prompt | llm

# 대화 히스토리 관리
chat_history = []

# 첫 번째 대화
response1 = chain.invoke({
    "chat_history": chat_history,
    "question": "내 이름은 김철수야"
})
print(f"AI: {response1.content}")

# 히스토리에 추가
chat_history.extend([
    HumanMessage(content="내 이름은 김철수야"),
    AIMessage(content=response1.content)
])

# 두 번째 대화 (이전 대화 기억)
response2 = chain.invoke({
    "chat_history": chat_history,
    "question": "내 이름이 뭐였지?"
})
print(f"AI: {response2.content}")
```

### ConversationBufferMemory:​

```python
from langchain.memory import ConversationBufferMemory
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

memory = ConversationBufferMemory(return_messages=True, memory_key="history")

prompt = ChatPromptTemplate.from_messages([
    ("system", "당신은 도움이 되는 AI입니다."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])

chain = prompt | llm

# 메모리를 통한 대화
def chat_with_memory(user_input):
    # 메모리에서 히스토리 가져오기
    history = memory.load_memory_variables({})
    
    # 체인 실행
    response = chain.invoke({
        "history": history.get("history", []),
        "input": user_input
    })
    
    # 메모리에 저장
    memory.save_context(
        {"input": user_input},
        {"output": response.content}
    )
    
    return response.content

print(chat_with_memory("안녕하세요!"))
print(chat_with_memory("제 이름은 홍길동입니다."))
print(chat_with_memory("제 이름이 뭐였죠?"))
```

## 5. 구조화된 출력 (Structured Output) <a href="#id-5---structured-output" id="id-5---structured-output"></a>

### Pydantic을 이용한 스키마 정의:​

```python
from langchain.chat_models import init_chat_model
from pydantic import BaseModel, Field
from typing import List, Optional

# 출력 스키마 정의
class Person(BaseModel):
    """사람 정보"""
    name: str = Field(description="사람의 이름")
    age: int = Field(description="나이")
    occupation: str = Field(description="직업")
    skills: List[str] = Field(description="보유 기술 목록")

class ExtractedPeople(BaseModel):
    """추출된 사람들의 정보"""
    people: List[Person] = Field(description="사람들 목록")
    summary: Optional[str] = Field(description="전체 요약")

# 구조화된 출력 설정
llm = init_chat_model("gpt-4o", model_provider="openai")
structured_llm = llm.with_structured_output(ExtractedPeople)

# 정보 추출
text = """
김영희는 35세의 데이터 과학자입니다. Python, SQL, TensorFlow를 다룹니다.
이철수는 28세 웹 개발자로 JavaScript, React, Node.js에 능숙합니다.
"""

result = structured_llm.invoke(
    f"다음 텍스트에서 사람들의 정보를 추출하세요:\n{text}"
)

# Pydantic 객체로 반환
print(f"추출된 사람 수: {len(result.people)}")
for person in result.people:
    print(f"이름: {person.name}, 나이: {person.age}, 직업: {person.occupation}")
    print(f"기술: {', '.join(person.skills)}")

# JSON 변환
import json
print(json.dumps(result.model_dump(), ensure_ascii=False, indent=2))
```

### TypedDict 사용:​

```python
from typing import TypedDict
from typing_extensions import Annotated

class ProductInfo(TypedDict):
    """제품 정보"""
    name: Annotated[str, ..., "제품명"]
    price: Annotated[float, ..., "가격"]
    category: Annotated[str, ..., "카테고리"]
    in_stock: Annotated[bool, ..., "재고 여부"]

llm = init_chat_model("gpt-4o")
structured_llm = llm.with_structured_output(ProductInfo)

result = structured_llm.invoke(
    "MacBook Pro M3는 2,890,000원이며 노트북 카테고리이고 재고가 있습니다."
)
print(result)  # dict 형태로 반환
```

## 6. 도구 호출 (Tool Calling) <a href="#id-6---tool-calling" id="id-6---tool-calling"></a>

### 함수를 도구로 바인딩:​

```python
from langchain.chat_models import init_chat_model
from langchain_core.tools import tool

# 도구 정의
@tool
def get_weather(city: str) -> str:
    """특정 도시의 현재 날씨를 조회합니다"""
    # 실제로는 API 호출
    weather_data = {
        "서울": "맑음, 20도",
        "부산": "흐림, 18도",
        "제주": "비, 16도"
    }
    return weather_data.get(city, "정보 없음")

@tool
def calculate(expression: str) -> float:
    """수학 계산을 수행합니다. 예: '25 * 4' 또는 '100 / 5'"""
    try:
        return eval(expression)
    except:
        return "계산 오류"

@tool
def search_database(query: str) -> str:
    """데이터베이스에서 정보를 검색합니다"""
    return f"{query}에 대한 검색 결과입니다."

# 모델에 도구 바인딩
llm = init_chat_model("gpt-4o", model_provider="openai")
llm_with_tools = llm.bind_tools([get_weather, calculate, search_database])

# 도구 호출 요청
response = llm_with_tools.invoke(
    "서울의 날씨를 알려주고, 25 곱하기 4를 계산해줘"
)

print("모델 응답:", response.content)
print("\n호출된 도구:")
for tool_call in response.tool_calls:
    print(f"- 도구명: {tool_call['name']}")
    print(f"  인자: {tool_call['args']}")
```

### 도구 강제 실행 및 옵션:​

```python
# 특정 도구 강제 실행
llm_with_forced_tool = llm.bind_tools(
    [get_weather],
    tool_choice={"type": "tool", "name": "get_weather"}
)

# 병렬 도구 호출 비활성화
llm_with_sequential_tools = llm.bind_tools(
    [get_weather, calculate],
    parallel_tool_calls=False
)
```

## 7. RAG (Retrieval-Augmented Generation) <a href="#id-7-rag-retrieval-augmented-generation" id="id-7-rag-retrieval-augmented-generation"></a>

### 기본 RAG 체인:​

```python
from langchain.chat_models import init_chat_model
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_core.documents import Document

# 1. 문서 준비 및 벡터 스토어 생성
documents = [
    Document(page_content="LangChain은 LLM 애플리케이션 개발 프레임워크입니다.", 
             metadata={"source": "docs"}),
    Document(page_content="RAG는 검색 증강 생성 기법입니다.", 
             metadata={"source": "docs"}),
    Document(page_content="벡터 데이터베이스는 임베딩을 저장합니다.", 
             metadata={"source": "docs"})
]

embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(documents, embeddings)
retriever = vectorstore.as_retriever(search_kwargs={"k": 2})

# 2. RAG 프롬프트
prompt = ChatPromptTemplate.from_template("""
다음 문맥을 기반으로 질문에 답변하세요.

문맥: {context}

질문: {question}

답변:
""")

# 3. 문서 포맷팅 함수
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

# 4. RAG 체인 구성 (LCEL)
llm = init_chat_model("gpt-4o", temperature=0)

rag_chain = (
    {
        "context": retriever | format_docs,
        "question": RunnablePassthrough()
    }
    | prompt
    | llm
    | StrOutputParser()
)

# 5. 실행
response = rag_chain.invoke("LangChain이 무엇인가요?")
print(response)

# 스트리밍 실행
for chunk in rag_chain.stream("RAG에 대해 설명해주세요"):
    print(chunk, end="", flush=True)
```

### RunnableParallel을 이용한 고급 RAG:​

```python
from langchain_core.runnables import RunnableParallel, RunnablePassthrough

# 병렬 처리를 명시적으로 표현
rag_chain_explicit = (
    RunnableParallel({
        "context": retriever | format_docs,
        "question": RunnablePassthrough()
    })
    | prompt
    | llm
    | StrOutputParser()
)

# 다중 체인 병렬 실행
multi_chain = RunnableParallel({
    "summary": retriever | format_docs | llm | StrOutputParser(),
    "keywords": retriever | format_docs | 
                (lambda x: "키워드 추출: " + x) | 
                llm | StrOutputParser(),
    "original_question": RunnablePassthrough()
})

result = multi_chain.invoke("LangChain 설명")
print(result)
```

## 8. Agent와 ReAct 패턴 <a href="#id-8-agent-react" id="id-8-agent-react"></a>

### create\_react\_agent 사용:​

```python
from langchain.chat_models import init_chat_model
from langchain_core.tools import tool
from langchain.agents import create_react_agent, AgentExecutor
from langchain import hub

# 도구 정의
@tool
def get_word_length(word: str) -> int:
    """단어의 길이를 반환합니다"""
    return len(word)

@tool
def multiply(a: int, b: int) -> int:
    """두 숫자를 곱합니다"""
    return a * b

tools = [get_word_length, multiply]

# 프롬프트 가져오기 (LangChain Hub)
prompt = hub.pull("hwchase17/react")

# LLM 초기화
llm = init_chat_model("gpt-4o", temperature=0)

# Agent 생성
agent = create_react_agent(llm, tools, prompt)

# Agent Executor
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    handle_parsing_errors=True,
    max_iterations=5
)

# 실행
result = agent_executor.invoke({
    "input": "'LangChain' 단어의 길이에 3을 곱하면?"
})
print(result["output"])
```

### LangGraph의 prebuilt agent:​

```python
from langgraph.prebuilt import create_react_agent

# 더 간단한 방식
llm = init_chat_model("gpt-4o")
tools = [get_word_length, multiply]

agent = create_react_agent(llm, tools)

# 실행
result = agent.invoke({
    "messages": [("human", "'Python' 단어 길이에 5를 곱하면?")]
})

for message in result["messages"]:
    print(f"{message.type}: {message.content}")
```

## 9. 런타임 설정 가능 모델 (Configurable Model) <a href="#id-9-----configurable-model" id="id-9-----configurable-model"></a>

### 기본 설정 없는 모델:​

```python
from langchain.chat_models import init_chat_model

# 모델을 지정하지 않으면 런타임에 선택 가능
configurable_llm = init_chat_model(temperature=0)

# OpenAI로 실행
response1 = configurable_llm.invoke(
    "Python의 장점은?",
    config={"configurable": {"model": "gpt-4o"}}
)

# Ollama로 실행
response2 = configurable_llm.invoke(
    "Python의 장점은?",
    config={"configurable": {
        "model": "llama3.1:8b",
        "model_provider": "ollama"
    }}
)

# Anthropic으로 실행
response3 = configurable_llm.invoke(
    "Python의 장점은?",
    config={"configurable": {"model": "claude-3-5-sonnet-latest"}}
)
```

### 기본값이 있는 설정 가능 모델:​

```python
# 기본 모델 설정 + 런타임 변경 가능
configurable_with_default = init_chat_model(
    "gpt-4o",
    model_provider="openai",
    temperature=0,
    configurable_fields=("model", "model_provider", "temperature", "max_tokens"),
    config_prefix="llm"  # 접두사로 구분
)

# 기본 모델로 실행
response1 = configurable_with_default.invoke("안녕하세요")

# 런타임에 다른 모델로 변경
response2 = configurable_with_default.invoke(
    "안녕하세요",
    config={
        "configurable": {
            "llm_model": "claude-3-5-sonnet-latest",
            "llm_model_provider": "anthropic",
            "llm_temperature": 0.7,
            "llm_max_tokens": 500
        }
    }
)
```

### 도구와 함께 사용:​

```python
from pydantic import BaseModel, Field

class GetWeather(BaseModel):
    """날씨 조회"""
    location: str = Field(description="도시명 (예: 서울, 부산)")

class GetPopulation(BaseModel):
    """인구 조회"""
    location: str = Field(description="도시명")

# 설정 가능한 모델에 도구 바인딩
configurable_llm = init_chat_model(
    "gpt-4o",
    configurable_fields=("model", "model_provider"),
    temperature=0
)

llm_with_tools = configurable_llm.bind_tools([GetWeather, GetPopulation])

# GPT-4o로 실행
result1 = llm_with_tools.invoke(
    "서울과 부산의 날씨를 알려주세요",
    config={"configurable": {"model": "gpt-4o"}}
)

# Claude로 실행
result2 = llm_with_tools.invoke(
    "서울과 부산의 날씨를 알려주세요",
    config={"configurable": {"model": "claude-3-5-sonnet-latest"}}
)
```

## 10. 토큰 사용량 추적 및 콜백 <a href="#id-10" id="id-10"></a>

### usage\_metadata로 토큰 추적:​

```python
from langchain.chat_models import init_chat_model

llm = init_chat_model("gpt-4o-mini")

response = llm.invoke("안녕하세요!")

# 토큰 사용량 확인
if response.usage_metadata:
    print(f"입력 토큰: {response.usage_metadata['input_tokens']}")
    print(f"출력 토큰: {response.usage_metadata['output_tokens']}")
    print(f"총 토큰: {response.usage_metadata['total_tokens']}")
```

### get\_openai\_callback 사용:​

```python
from langchain_community.callbacks import get_openai_callback
from langchain.chat_models import init_chat_model

llm = init_chat_model("gpt-4o")

# 단일 호출 추적
with get_openai_callback() as cb:
    result = llm.invoke("Python에 대해 설명해주세요")
    print(f"총 토큰: {cb.total_tokens}")
    print(f"프롬프트 토큰: {cb.prompt_tokens}")
    print(f"완료 토큰: {cb.completion_tokens}")
    print(f"총 비용 (USD): ${cb.total_cost}")

# 여러 호출 추적
with get_openai_callback() as cb:
    result1 = llm.invoke("첫 번째 질문")
    result2 = llm.invoke("두 번째 질문")
    result3 = llm.invoke("세 번째 질문")
    
    print(f"\n총 요청 수: {cb.successful_requests}")
    print(f"총 토큰: {cb.total_tokens}")
    print(f"총 비용: ${cb.total_cost}")
```

## 11. OpenAI와 Ollama 통합 예제 <a href="#id-11-openai-ollama" id="id-11-openai-ollama"></a>

### OpenAI와 Ollama 모델 전환:​

```python
from langchain.chat_models import init_chat_model

# Ollama는 OpenAI 호환 API 제공
# OpenAI 패키지로 Ollama 접근 가능

# 방법 1: init_chat_model 사용 (권장)
llm_ollama = init_chat_model(
    "llama3.1:8b",
    model_provider="ollama"
)

# 방법 2: OpenAI 호환 인터페이스 사용
from langchain_openai import ChatOpenAI

llm_ollama_openai_compatible = ChatOpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama",  # 필수지만 사용되지 않음
    model="llama3.1:8b"
)

# 동일한 인터페이스로 사용
response1 = llm_ollama.invoke("안녕하세요")
response2 = llm_ollama_openai_compatible.invoke("안녕하세요")
```

### 멀티 모델 애플리케이션:​

```python
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate

# 런타임에 모델 선택 가능한 설정
def create_flexible_chain(system_message: str):
    prompt = ChatPromptTemplate.from_messages([
        ("system", system_message),
        ("human", "{input}")
    ])
    
    llm = init_chat_model(temperature=0)
    return prompt | llm

chain = create_flexible_chain("당신은 도움이 되는 AI입니다.")

# 사용자가 모델 선택
def ask(question: str, model_choice: str = "gpt-4o"):
    model_configs = {
        "gpt-4o": {"model": "gpt-4o"},
        "gpt-3.5": {"model": "gpt-3.5-turbo"},
        "llama": {"model": "llama3.1:8b", "model_provider": "ollama"},
        "claude": {"model": "claude-3-5-sonnet-latest"}
    }
    
    config = {"configurable": model_configs[model_choice]}
    return chain.invoke({"input": question}, config=config)

# 다양한 모델로 실행
print("GPT-4o:", ask("Python이란?", "gpt-4o").content)
print("Llama:", ask("Python이란?", "llama").content)
print("Claude:", ask("Python이란?", "claude").content)
```

## 12. 고급 패턴: 체인 조합 <a href="#id-12" id="id-12"></a>

### RunnableSequence와 RunnableParallel:​

```python
from langchain_core.runnables import RunnableSequence, RunnableParallel
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate

llm = init_chat_model("gpt-4o", temperature=0)

# 순차 체인
prompt1 = ChatPromptTemplate.from_template("{topic}에 대해 간단히 설명해")
prompt2 = ChatPromptTemplate.from_template("다음 내용을 3줄로 요약해: {text}")

sequential_chain = (
    {"topic": RunnablePassthrough()}
    | prompt1
    | llm
    | (lambda x: {"text": x.content})
    | prompt2
    | llm
)

result = sequential_chain.invoke("머신러닝")

# 병렬 체인
parallel_chain = RunnableParallel({
    "summary": prompt1 | llm,
    "keywords": ChatPromptTemplate.from_template(
        "{topic}의 주요 키워드 5개만 나열해"
    ) | llm,
    "examples": ChatPromptTemplate.from_template(
        "{topic}의 실제 응용 사례 3가지"
    ) | llm
})

results = parallel_chain.invoke({"topic": "자연어처리"})
print("요약:", results["summary"].content)
print("키워드:", results["keywords"].content)
print("사례:", results["examples"].content)
```

## 13. 에러 처리 및 폴백 <a href="#id-13" id="id-13"></a>

```python
from langchain.chat_models import init_chat_model
from langchain_core.runnables import RunnableLambda

def safe_invoke(model, question, fallback_message="처리 중 오류가 발생했습니다."):
    try:
        return model.invoke(question).content
    except Exception as e:
        print(f"오류 발생: {e}")
        return fallback_message

# 폴백 체인
primary_llm = init_chat_model("gpt-4o", timeout=5)
fallback_llm = init_chat_model("gpt-3.5-turbo")

def invoke_with_fallback(question):
    try:
        return primary_llm.invoke(question)
    except:
        print("Primary 모델 실패, fallback 모델 사용")
        return fallback_llm.invoke(question)
```

### &#x20;<a href="#undefined" id="undefined"></a>
