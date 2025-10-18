# Memory

## 1. LangChain v1.0 주요 변경사항 <a href="#id-1-langchain-v10" id="id-1-langchain-v10"></a>

LangChain v1.0에서는 메모리 시스템이 근본적으로 재설계되었습니다.​

### 1-1. Deprecated (제거됨)

* `ConversationBufferMemory`​
* `ConversationBufferWindowMemory`​
* `ConversationSummaryMemory`​
* `ConversationSummaryBufferMemory`​
* `ConversationTokenBufferMemory`​
* `ConversationEntityMemory`​
* `ConversationKGMemory`​
* `VectorStoreRetrieverMemory`​
* `LLMChain`​
* `ConversationChain`​

### 1-2. 새로운 표준 (v1.0)

* **LangGraph persistence** (권장)​
* `RunnableWithMessageHistory` (간단한 체인용)​
* `init_chat_model` (통합 모델 초기화)​
* `BaseChatMessageHistory` 구현체​

\<a name="short-term-memory">\</a>

## 2. Short-term Memory  <a href="#id-2-short-term-memory" id="id-2-short-term-memory"></a>

Short-term Memory는 단일 대화 스레드 내에서 메시지를 유지.​

### 2.1 기본 패턴: LangGraph + MemorySaver (OpenAI)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import MemorySaver
from langchain.chat_models import init_chat_model

# OpenAI 모델 초기화
model = init_chat_model("gpt-4o-mini", model_provider="openai", temperature=0)

# 워크플로우 정의
workflow = StateGraph(state_schema=MessagesState)

def call_model(state: MessagesState):
    """모델 호출 노드"""
    response = model.invoke(state["messages"])
    return {"messages": [response]}

# 그래프 구성
workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

# 메모리 체크포인터 추가
memory = MemorySaver()
app = workflow.compile(checkpointer=memory)

# 대화 실행
config = {"configurable": {"thread_id": "conversation-001"}}

# 첫 번째 턴
response1 = app.invoke(
    {"messages": [{"role": "user", "content": "안녕하세요! 제 이름은 김철수입니다."}]},
    config
)
print(response1["messages"][-1].content)

# 두 번째 턴 (이름 기억)
response2 = app.invoke(
    {"messages": [{"role": "user", "content": "제 이름이 뭐였죠?"}]},
    config
)
print(response2["messages"][-1].content)
# 출력: "김철수님이시죠!"
```

### 2.2 기본 패턴: LangGraph + MemorySaver (Ollama)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import MemorySaver
from langchain.chat_models import init_chat_model

# Ollama 모델 초기화
model = init_chat_model(
    "llama3.1",
    model_provider="ollama",
    temperature=0,
    base_url="http://localhost:11434"  # Ollama 서버 주소
)

workflow = StateGraph(state_schema=MessagesState)

def call_model(state: MessagesState):
    response = model.invoke(state["messages"])
    return {"messages": [response]}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

memory = MemorySaver()
app = workflow.compile(checkpointer=memory)

# 대화 실행
config = {"configurable": {"thread_id": "ollama-thread-001"}}

conversations = [
    "저는 Python 개발자입니다.",
    "주로 FastAPI와 Django를 사용해요.",
    "제 직업이 뭐였죠?"
]

for msg in conversations:
    response = app.invoke(
        {"messages": [{"role": "user", "content": msg}]},
        config
    )
    print(f"User: {msg}")
    print(f"AI: {response['messages'][-1].content}\n")
```

### 2.3 RunnableWithMessageHistory 패턴 (OpenAI)

레거시 메모리 클래스를 사용하던 간단한 체인을 마이그레이션할 때 유용.​

```python
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

# 세션별 메시지 히스토리 저장소
store = {}

def get_session_history(session_id: str) -> InMemoryChatMessageHistory:
    """세션 ID별로 메시지 히스토리 관리"""
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]

# OpenAI 모델
model = init_chat_model("gpt-4o-mini", model_provider="openai")

# 프롬프트 템플릿
prompt = ChatPromptTemplate.from_messages([
    ("system", "당신은 친절한 AI 어시스턴트입니다."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])

# 체인 구성
chain = prompt | model

# 메시지 히스토리 래퍼
chain_with_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history"
)

# 대화 실행
config = {"configurable": {"session_id": "user-123"}}

response1 = chain_with_history.invoke(
    {"input": "제 이름은 이영희입니다."},
    config
)
print(response1.content)

response2 = chain_with_history.invoke(
    {"input": "제 이름이 뭐였죠?"},
    config
)
print(response2.content)
```

### 2.4 RunnableWithMessageHistory 패턴 (Ollama)

```python
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

store = {}

def get_session_history(session_id: str) -> InMemoryChatMessageHistory:
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]

# Ollama 모델
model = init_chat_model(
    "llama3.1",
    model_provider="ollama",
    temperature=0
)

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful AI assistant. Answer in Korean."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])

chain = prompt | model

chain_with_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history"
)

config = {"configurable": {"session_id": "ollama-user-001"}}

# 여러 턴의 대화
inputs = [
    "저는 등산을 좋아합니다.",
    "주말마다 북한산에 갑니다.",
    "제가 자주 가는 곳이 어디였죠?"
]

for user_input in inputs:
    response = chain_with_history.invoke({"input": user_input}, config)
    print(f"User: {user_input}")
    print(f"AI: {response.content}\n")
```

### 2.5 메시지 트리밍 (토큰 관리)

LangChain v1.0에서는 `trim_messages`를 사용하여 컨텍스트 윈도우를 관리.​

```python
from langchain_core.messages import trim_messages
from langchain.chat_models import init_chat_model
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import MemorySaver

# OpenAI 모델
model = init_chat_model("gpt-4o-mini", model_provider="openai")

workflow = StateGraph(state_schema=MessagesState)

def call_model_with_trimming(state: MessagesState):
    """메시지 트리밍 후 모델 호출"""
    # 최근 1000 토큰만 유지
    trimmed_messages = trim_messages(
        state["messages"],
        max_tokens=1000,
        token_counter=model,  # 모델 기반 토큰 카운팅
        strategy="last",  # 최근 메시지 우선
        include_system=True,  # 시스템 메시지는 항상 유지
        start_on="human",  # human 메시지로 시작
        allow_partial=False  # 메시지 부분 포함 방지
    )
    
    response = model.invoke(trimmed_messages)
    return {"messages": [response]}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model_with_trimming)

memory = MemorySaver()
app = workflow.compile(checkpointer=memory)

# 긴 대화 시뮬레이션
config = {"configurable": {"thread_id": "long-conversation-001"}}

# 여러 턴의 대화
for i in range(20):
    response = app.invoke(
        {"messages": [{"role": "user", "content": f"메시지 번호 {i+1}입니다."}]},
        config
    )
    print(f"Turn {i+1}: {response['messages'][-1].content}")
```

### 2.6 Ollama 모델에서 메시지 트리밍

```python
from langchain_core.messages import trim_messages, count_tokens_approximately
from langchain.chat_models import init_chat_model
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import MemorySaver

# Ollama 모델
model = init_chat_model("llama3.1", model_provider="ollama", temperature=0)

workflow = StateGraph(state_schema=MessagesState)

def call_model_with_trimming(state: MessagesState):
    """근사 토큰 카운팅을 사용한 트리밍"""
    trimmed_messages = trim_messages(
        state["messages"],
        max_tokens=2000,
        token_counter=count_tokens_approximately,  # 빠른 근사 카운팅
        strategy="last",
        include_system=True,
        start_on="human"
    )
    
    response = model.invoke(trimmed_messages)
    return {"messages": [response]}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model_with_trimming)

memory = MemorySaver()
app = workflow.compile(checkpointer=memory)

config = {"configurable": {"thread_id": "ollama-trim-001"}}

# 긴 대화 테스트
for i in range(15):
    long_message = f"이것은 긴 메시지 {i+1}입니다. " * 10
    response = app.invoke(
        {"messages": [{"role": "user", "content": long_message}]},
        config
    )
    print(f"Turn {i+1} completed")
```

### 2.7 PostgreSQL Checkpointer (영구 Short-term Memory - OpenAI)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.postgres import PostgresSaver
from langchain.chat_models import init_chat_model

# PostgreSQL 연결 문자열
DB_URI = "postgresql://username:password@localhost:5432/chatbot_db?sslmode=disable"

# OpenAI 모델
model = init_chat_model("gpt-4o-mini", model_provider="openai")

workflow = StateGraph(state_schema=MessagesState)

def call_model(state: MessagesState):
    response = model.invoke(state["messages"])
    return {"messages": [response]}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

# PostgreSQL 체크포인터 사용
with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    # 최초 1회만 실행 (테이블 생성)
    checkpointer.setup()
    
    app = workflow.compile(checkpointer=checkpointer)
    
    config = {"configurable": {"thread_id": "persistent-openai-001"}}
    
    # 첫 번째 실행
    response1 = app.invoke(
        {"messages": [{"role": "user", "content": "제 이름은 박민수입니다."}]},
        config
    )
    print(response1["messages"][-1].content)
    
    # 애플리케이션 재시작 후에도 메모리 유지
    response2 = app.invoke(
        {"messages": [{"role": "user", "content": "제 이름이 뭐였죠?"}]},
        config
    )
    print(response2["messages"][-1].content)
```

### 2.8 PostgreSQL Checkpointer (Ollama)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.postgres import PostgresSaver
from langchain.chat_models import init_chat_model

DB_URI = "postgresql://username:password@localhost:5432/chatbot_db?sslmode=disable"

# Ollama 모델
model = init_chat_model("llama3.1", model_provider="ollama", temperature=0)

workflow = StateGraph(state_schema=MessagesState)

def call_model(state: MessagesState):
    response = model.invoke(state["messages"])
    return {"messages": [response]}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    # checkpointer.setup()  # 최초 1회만
    
    app = workflow.compile(checkpointer=checkpointer)
    
    config = {"configurable": {"thread_id": "persistent-ollama-001"}}
    
    conversations = [
        "저는 서울에 살고 있습니다.",
        "삼성동 코엑스 근처입니다.",
        "제가 어디에 사나요?"
    ]
    
    for msg in conversations:
        response = app.invoke(
            {"messages": [{"role": "user", "content": msg}]},
            config
        )
        print(f"User: {msg}")
        print(f"AI: {response['messages'][-1].content}\n")
```

### 2.9 Redis Checkpointer (OpenAI)from langgraph.graph import StateGraph, MessagesState, START

```python
from langgraph.checkpoint.redis import RedisSaver
from langchain.chat_models import init_chat_model

# Redis 연결
REDIS_URI = "redis://localhost:6379"

model = init_chat_model("gpt-4o-mini", model_provider="openai")

workflow = StateGraph(state_schema=MessagesState)

def call_model(state: MessagesState):
    response = model.invoke(state["messages"])
    return {"messages": [response]}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

with RedisSaver.from_conn_string(REDIS_URI) as checkpointer:
    # checkpointer.setup()  # 최초 1회만
    
    app = workflow.compile(checkpointer=checkpointer)
    
    config = {"configurable": {"thread_id": "redis-openai-001"}}
    
    response1 = app.invoke(
        {"messages": [{"role": "user", "content": "제가 좋아하는 색은 파란색입니다."}]},
        config
    )
    print(response1["messages"][-1].content)
    
    response2 = app.invoke(
        {"messages": [{"role": "user", "content": "제가 좋아하는 색이 뭐였죠?"}]},
        config
    )
    print(response2["messages"][-1].content)
```

### 2.10 Redis Checkpointer (Ollama)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.redis import RedisSaver
from langchain.chat_models import init_chat_model

REDIS_URI = "redis://localhost:6379"

model = init_chat_model("llama3.1", model_provider="ollama")

workflow = StateGraph(state_schema=MessagesState)

def call_model(state: MessagesState):
    response = model.invoke(state["messages"])
    return {"messages": [response]}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

with RedisSaver.from_conn_string(REDIS_URI) as checkpointer:
    # checkpointer.setup()  # 최초 1회만
    
    app = workflow.compile(checkpointer=checkpointer)
    
    config = {"configurable": {"thread_id": "redis-ollama-001"}}
    
    # TTL 설정 (1시간)
    for i in range(3):
        response = app.invoke(
            {"messages": [{"role": "user", "content": f"메시지 {i+1}"}]},
            config
        )
        print(f"Response {i+1}: {response['messages'][-1].content}")
```

\<a name="long-term-memory">\</a>

## 3. Long-term Memory  <a href="#id-3-long-term-memory" id="id-3-long-term-memory"></a>

Long-term Memory는 여러 스레드와 세션을 걸쳐 정보를 유지합니다.​

### 3.1 기본 InMemoryStore 패턴 (OpenAI)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import MemorySaver
from langgraph.store.memory import InMemoryStore
from langchain.chat_models import init_chat_model
from langchain_core.runnables import RunnableConfig
from langgraph.store.base import BaseStore
import uuid

# OpenAI 모델
model = init_chat_model("gpt-4o-mini", model_provider="openai")

# 체크포인터와 스토어
checkpointer = MemorySaver()
store = InMemoryStore()

workflow = StateGraph(state_schema=MessagesState)

def call_model(state: MessagesState, config: RunnableConfig, *, store: BaseStore):
    """장기 메모리를 활용한 응답 생성"""
    user_id = config["configurable"].get("user_id")
    namespace = ("user_facts", user_id)
    
    # 장기 메모리 검색
    last_message = state["messages"][-1].content
    memories = store.search(namespace)
    
    # 메모리 컨텍스트 구성
    if memories:
        memory_context = "\n".join([
            f"- {mem.value.get('fact', '')}" for mem in memories
        ])
        system_message = {
            "role": "system",
            "content": f"사용자 정보:\n{memory_context}\n\n위 정보를 참고하여 응답하세요."
        }
        messages = [system_message] + state["messages"]
    else:
        messages = state["messages"]
    
    response = model.invoke(messages)
    
    # 중요 정보 저장
    if "좋아합니다" in last_message or "좋아해요" in last_message:
        store.put(
            namespace,
            str(uuid.uuid4()),
            {"fact": last_message, "type": "preference"}
        )
    
    return {"messages": [response]}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

app = workflow.compile(checkpointer=checkpointer, store=store)

# 사용 예제
user_id = "user-openai-001"

# 첫 번째 스레드
config1 = {
    "configurable": {
        "thread_id": "thread-1",
        "user_id": user_id
    }
}

response1 = app.invoke(
    {"messages": [{"role": "user", "content": "저는 커피를 좋아합니다."}]},
    config1
)
print(response1["messages"][-1].content)

response2 = app.invoke(
    {"messages": [{"role": "user", "content": "등산도 좋아해요."}]},
    config1
)
print(response2["messages"][-1].content)

# 다른 스레드에서 같은 사용자 정보 활용
config2 = {
    "configurable": {
        "thread_id": "thread-2",
        "user_id": user_id
    }
}

response3 = app.invoke(
    {"messages": [{"role": "user", "content": "제가 좋아하는 것들을 알려주세요."}]},
    config2
)
print(response3["messages"][-1].content)
```

### 3.2 기본 InMemoryStore 패턴 (Ollama)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import MemorySaver
from langgraph.store.memory import InMemoryStore
from langchain.chat_models import init_chat_model
from langchain_core.runnables import RunnableConfig
from langgraph.store.base import BaseStore
import uuid

# Ollama 모델
model = init_chat_model("llama3.1", model_provider="ollama", temperature=0)

checkpointer = MemorySaver()
store = InMemoryStore()

workflow = StateGraph(state_schema=MessagesState)

def call_model(state: MessagesState, config: RunnableConfig, *, store: BaseStore):
    user_id = config["configurable"].get("user_id")
    namespace = ("user_profile", user_id)
    
    # 장기 메모리 검색
    memories = store.search(namespace)
    
    # 시스템 프롬프트에 메모리 추가
    memory_text = "\n".join([
        f"- {mem.value.get('info', '')}" for mem in memories
    ])
    
    if memory_text:
        system_msg = {
            "role": "system",
            "content": f"User profile:\n{memory_text}\n\nProvide personalized responses based on this information. Answer in Korean."
        }
        messages = [system_msg] + state["messages"]
    else:
        messages = state["messages"]
    
    response = model.invoke(messages)
    
    # 새로운 정보 저장
    last_message = state["messages"][-1].content
    
    if "이름은" in last_message and "입니다" in last_message:
        name_info = last_message.split("이름은")[1].split("입니다")[0].strip()
        store.put(namespace, "name", {"info": f"이름: {name_info}", "type": "identity"})
    
    if "살아요" in last_message or "살입니다" in last_message:
        store.put(namespace, str(uuid.uuid4()), {"info": last_message, "type": "location"})
    
    return {"messages": [response]}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

app = workflow.compile(checkpointer=checkpointer, store=store)

# 실행 예제
user_id = "user-ollama-001"

config1 = {
    "configurable": {
        "thread_id": "session-1",
        "user_id": user_id
    }
}

conversations = [
    "안녕하세요! 제 이름은 정수민입니다.",
    "저는 부산에 살아요.",
    "데이터 분석가로 일하고 있습니다."
]

for msg in conversations:
    response = app.invoke(
        {"messages": [{"role": "user", "content": msg}]},
        config1
    )
    print(f"User: {msg}")
    print(f"AI: {response['messages'][-1].content}\n")

# 새로운 세션에서 저장된 정보 활용
config2 = {
    "configurable": {
        "thread_id": "session-2",
        "user_id": user_id
    }
}

response = app.invoke(
    {"messages": [{"role": "user", "content": "저에 대해 알고 있는 것을 말해주세요."}]},
    config2
)
print(f"New Session - AI: {response['messages'][-1].content}")
```

### 3.3 Semantic Search with OpenAI Embeddings

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import MemorySaver
from langgraph.store.memory import InMemoryStore
from langchain.chat_models import init_chat_model
from langchain.embeddings import init_embeddings
from langchain_core.runnables import RunnableConfig
from langgraph.store.base import BaseStore
import uuid

# OpenAI 모델과 임베딩
model = init_chat_model("gpt-4o-mini", model_provider="openai")
embeddings = init_embeddings("openai:text-embedding-3-small")

checkpointer = MemorySaver()

# 벡터 검색 활성화
store = InMemoryStore(
    index={
        "dims": 1536,  # text-embedding-3-small 차원
        "embed": embeddings,
        "fields": ["content"]  # 임베딩할 필드
    }
)

workflow = StateGraph(state_schema=MessagesState)

def call_model(state: MessagesState, config: RunnableConfig, *, store: BaseStore):
    """의미 기반 메모리 검색 및 응답"""
    user_id = config["configurable"].get("user_id")
    namespace = ("memories", user_id)
    
    last_message = state["messages"][-1].content
    
    # 의미 기반 검색
    relevant_memories = store.search(
        namespace,
        query=last_message,  # 현재 메시지와 유사한 메모리 검색
        limit=3
    )
    
    # 메모리 컨텍스트 구성
    if relevant_memories:
        memory_context = "\n".join([
            f"- {mem.value.get('content', '')} (유사도: {mem.score:.2f})"
            for mem in relevant_memories
        ])
        system_msg = {
            "role": "system",
            "content": f"관련 기억:\n{memory_context}\n\n위 정보를 참고하여 응답하세요."
        }
        messages = [system_msg] + state["messages"]
    else:
        messages = state["messages"]
    
    response = model.invoke(messages)
    
    # 중요한 정보를 장기 메모리에 저장
    if "기억" in last_message or "저장" in last_message:
        store.put(
            namespace,
            str(uuid.uuid4()),
            {"content": last_message},
            index=["content"]  # content 필드를 벡터 인덱싱
        )
    
    return {"messages": [response]}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

app = workflow.compile(checkpointer=checkpointer, store=store)

# 사용 예제
user_id = "user-semantic-001"

config = {
    "configurable": {
        "thread_id": "thread-1",
        "user_id": user_id
    }
}

# 여러 정보 저장
memories_to_store = [
    "기억해줘: 저는 매일 아침 6시에 조깅합니다.",
    "기억해줘: 제가 좋아하는 음식은 파스타입니다.",
    "기억해줘: Python과 JavaScript를 주로 사용합니다.",
    "기억해줘: 주말에는 독서를 즐깁니다."
]

for memory in memories_to_store:
    response = app.invoke(
        {"messages": [{"role": "user", "content": memory}]},
        config
    )
    print(f"Stored: {memory}")

# 의미 기반 검색 테스트
questions = [
    "아침 루틴이 어떻게 되나요?",  # "조깅" 메모리와 유사
    "음식 취향은 어떤가요?",  # "파스타" 메모리와 유사
    "개발 관련 일 하시나요?"  # "Python, JavaScript" 메모리와 유사
]

for question in questions:
    config_new = {
        "configurable": {
            "thread_id": f"test-{uuid.uuid4()}",
            "user_id": user_id
        }
    }
    
    response = app.invoke(
        {"messages": [{"role": "user", "content": question}]},
        config_new
    )
    print(f"\nQ: {question}")
    print(f"A: {response['messages'][-1].content}")
```

### 3.4 Semantic Search with Ollama Embeddings

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import MemorySaver
from langgraph.store.memory import InMemoryStore
from langchain.chat_models import init_chat_model
from langchain_ollama import OllamaEmbeddings
from langchain_core.runnables import RunnableConfig
from langgraph.store.base import BaseStore
import uuid

# Ollama 모델과 임베딩
model = init_chat_model("llama3.1", model_provider="ollama", temperature=0)
embeddings = OllamaEmbeddings(
    model="mxbai-embed-large",  # Ollama 임베딩 모델
    base_url="http://localhost:11434"
)

checkpointer = MemorySaver()

# 벡터 검색 활성화
store = InMemoryStore(
    index={
        "dims": 1024,  # mxbai-embed-large 차원
        "embed": embeddings,
        "fields": ["text"]
    }
)

workflow = StateGraph(state_schema=MessagesState)

def call_model(state: MessagesState, config: RunnableConfig, *, store: BaseStore):
    """Ollama 임베딩을 사용한 의미 기반 검색"""
    user_id = config["configurable"].get("user_id")
    namespace = ("memories", user_id)
    
    last_message = state["messages"][-1].content
    
    # 의미 기반 검색
    relevant_memories = store.search(
        namespace,
        query=last_message,
        limit=3
    )
    
    # 메모리 컨텍스트 구성
    if relevant_memories:
        memory_context = "\n".join([
            f"- {mem.value.get('text', '')} (score: {mem.score:.2f})"
            for mem in relevant_memories
        ])
        system_msg = {
            "role": "system",
            "content": f"Relevant memories:\n{memory_context}\n\nUse this information to provide context-aware responses. Answer in Korean."
        }
        messages = [system_msg] + state["messages"]
    else:
        messages = state["messages"]
    
    response = model.invoke(messages)
    
    # 정보 저장
    if "기억" in last_message or "remember" in last_message.lower():
        store.put(
            namespace,
            str(uuid.uuid4()),
            {"text": last_message},
            index=["text"]
        )
    
    return {"messages": [response]}

workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

app = workflow.compile(checkpointer=checkpointer, store=store)

# 실행 예제
user_id = "user-ollama-semantic-001"

config = {
    "configurable": {
        "thread_id": "thread-1",
        "user_id": user_id
    }
}

# 정보 저장
memories = [
    "기억해줘: 저는 프론트엔드 개발자입니다.",
    "기억해줘: React와 Vue.js를 사용합니다.",
    "기억해줘: 카페에서 일하는 것을 좋아합니다.",
    "기억해줘: 재즈 음악을 즐겨 듣습니다."
]

for memory in memories:
    response = app.invoke(
        {"messages": [{"role": "user", "content": memory}]},
        config
    )
    print(f"Stored: {memory}")

# 의미 검색 테스트
questions = [
    "개발 업무에 대해 알려주세요",
    "일할 때 선호하는 환경이 있나요?",
    "어떤 음악을 좋아하시나요?"
]

for question in questions:
    config_new = {
        "configurable": {
            "thread_id": f"test-{uuid.uuid4()}",
            "user_id": user_id
        }
    }
    
    response = app.invoke(
        {"messages": [{"role": "user", "content": question}]},
        config_new
    )
    print(f"\nQ: {question}")
    print(f"A: {response['messages'][-1].content}")
```

### 3.5 PostgreSQL Store with Vector Search (OpenAI)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.postgres import PostgresSaver
from langgraph.store.postgres import PostgresStore
from langchain.chat_models import init_chat_model
from langchain.embeddings import init_embeddings
from langchain_core.runnables import RunnableConfig
from langgraph.store.base import BaseStore
import uuid

# PostgreSQL 연결
DB_URI = "postgresql://username:password@localhost:5432/chatbot_db?sslmode=disable"

# OpenAI 모델과 임베딩
model = init_chat_model("gpt-4o-mini", model_provider="openai")
embeddings = init_embeddings("openai:text-embedding-3-small")

with (
    PostgresStore.from_conn_string(
        DB_URI,
        index={
            "dims": 1536,
            "embed": embeddings,
            "fields": ["content", "summary"]
        }
    ) as store,
    PostgresSaver.from_conn_string(DB_URI) as checkpointer,
):
    # 최초 1회만 실행
    # store.setup()
    # checkpointer.setup()
    
    workflow = StateGraph(state_schema=MessagesState)
    
    def call_model(state: MessagesState, config: RunnableConfig, *, store: BaseStore):
        """PostgreSQL 벡터 검색을 사용한 장기 메모리"""
        user_id = config["configurable"].get("user_id")
        namespace = ("user_knowledge", user_id)
        
        last_message = state["messages"][-1].content
        
        # 벡터 유사도 검색
        relevant_memories = store.search(
            namespace,
            query=last_message,
            limit=5
        )
        
        # 메모리 컨텍스트
        if relevant_memories:
            memory_context = "\n".join([
                f"- {mem.value.get('content', '')} (유사도: {mem.score:.2f})"
                for mem in relevant_memories
            ])
            system_msg = {
                "role": "system",
                "content": f"사용자 정보:\n{memory_context}\n\n위 정보를 활용하여 응답하세요."
            }
            messages = [system_msg] + state["messages"]
        else:
            messages = state["messages"]
        
        response = model.invoke(messages)
        
        # 중요 정보 저장
        keywords = ["좋아", "싫어", "선호", "기억", "이름", "직업", "취미"]
        if any(keyword in last_message for keyword in keywords):
            store.put(
                namespace,
                str(uuid.uuid4()),
                {
                    "content": last_message,
                    "summary": last_message[:50],
                    "timestamp": str(uuid.uuid1().time)
                },
                index=["content", "summary"]
            )
        
        return {"messages": [response]}
    
    workflow.add_edge(START, "model")
    workflow.add_node("model", call_model)
    
    app = workflow.compile(checkpointer=checkpointer, store=store)
    
    # 사용 예제
    user_id = "user-postgres-001"
    
    config = {
        "configurable": {
            "thread_id": "thread-1",
            "user_id": user_id
        }
    }
    
    # 정보 저장
    conversations = [
        "제 이름은 최윤아이고 그래픽 디자이너입니다.",
        "Adobe Illustrator와 Figma를 주로 사용해요.",
        "미니멀한 디자인을 선호합니다.",
        "커피를 정말 좋아합니다."
    ]
    
    for msg in conversations:
        response = app.invoke(
            {"messages": [{"role": "user", "content": msg}]},
            config
        )
        print(f"User: {msg}")
        print(f"AI: {response['messages'][-1].content}\n")
    
    # 새로운 세션에서 검색
    config2 = {
        "configurable": {
            "thread_id": "thread-2",
            "user_id": user_id
        }
    }
    
    questions = [
        "제 직업이 뭐였죠?",
        "어떤 도구를 사용하나요?",
        "디자인 스타일은 어떤가요?"
    ]
    
    for question in questions:
        response = app.invoke(
            {"messages": [{"role": "user", "content": question}]},
            config2
        )
        print(f"Q: {question}")
        print(f"A: {response['messages'][-1].content}\n")
```

### 3.6 PostgreSQL Store with Vector Search (Ollama)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.postgres import PostgresSaver
from langgraph.store.postgres import PostgresStore
from langchain.chat_models import init_chat_model
from langchain_ollama import OllamaEmbeddings
from langchain_core.runnables import RunnableConfig
from langgraph.store.base import BaseStore
import uuid

# PostgreSQL 연결
DB_URI = "postgresql://username:password@localhost:5432/chatbot_db?sslmode=disable"

# Ollama 모델과 임베딩
model = init_chat_model("llama3.1", model_provider="ollama", temperature=0)
embeddings = OllamaEmbeddings(model="mxbai-embed-large")

with (
    PostgresStore.from_conn_string(
        DB_URI,
        index={
            "dims": 1024,
            "embed": embeddings,
            "fields": ["text"]
        }
    ) as store,
    PostgresSaver.from_conn_string(DB_URI) as checkpointer,
):
    # store.setup()
    # checkpointer.setup()
    
    workflow = StateGraph(state_schema=MessagesState)
    
    def call_model(state: MessagesState, config: RunnableConfig, *, store: BaseStore):
        """Ollama 임베딩과 PostgreSQL을 사용한 장기 메모리"""
        user_id = config["configurable"].get("user_id")
        namespace = ("memories", user_id)
        
        last_message = state["messages"][-1].content
        
        # 벡터 검색
        relevant_memories = store.search(
            namespace,
            query=last_message,
            limit=3
        )
        
        # 메모리 컨텍스트
        if relevant_memories:
            memory_context = "\n".join([
                f"- {mem.value.get('text', '')}"
                for mem in relevant_memories
            ])
            system_msg = {
                "role": "system",
                "content": f"User information:\n{memory_context}\n\nUse this to provide personalized responses. Answer in Korean."
            }
            messages = [system_msg] + state["messages"]
        else:
            messages = state["messages"]
        
        response = model.invoke(messages)
        
        # 정보 저장
        if "기억" in last_message:
            store.put(
                namespace,
                str(uuid.uuid4()),
                {"text": last_message},
                index=["text"]
            )
        
        return {"messages": [response]}
    
    workflow.add_edge(START, "model")
    workflow.add_node("model", call_model)
    
    app = workflow.compile(checkpointer=checkpointer, store=store)
    
    # 실행
    user_id = "user-ollama-postgres-001"
    
    config = {
        "configurable": {
            "thread_id": "thread-1",
            "user_id": user_id
        }
    }
    
    # 정보 저장
    memories = [
        "기억해줘: 저는 백엔드 개발자입니다.",
        "기억해줘: Node.js와 Go를 사용합니다.",
        "기억해줘: 마이크로서비스 아키텍처를 선호합니다."
    ]
    
    for memory in memories:
        response = app.invoke(
            {"messages": [{"role": "user", "content": memory}]},
            config
        )
        print(f"Stored: {memory}")
    
    # 검색
    config2 = {
        "configurable": {
            "thread_id": "thread-2",
            "user_id": user_id
        }
    }
    
    response = app.invoke(
        {"messages": [{"role": "user", "content": "제 기술 스택을 알려주세요."}]},
        config2
    )
    print(f"\nA: {response['messages'][-1].content}")
```

\<a name="all-patterns">\</a>

## 4. 모든 메모리 패턴 구현 <a href="#id-4" id="id-4"></a>

### 4.1 ConversationBufferMemory 대체 (OpenAI + Ollama)

레거시 `ConversationBufferMemory`를 LangGraph로 완전 대체.​

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import MemorySaver
from langchain.chat_models import init_chat_model

def create_conversation_app(model_name: str, model_provider: str):
    """ConversationBufferMemory 대체 구현"""
    model = init_chat_model(model_name, model_provider=model_provider)
    
    workflow = StateGraph(state_schema=MessagesState)
    
    def call_model(state: MessagesState):
        response = model.invoke(state["messages"])
        return {"messages": [response]}
    
    workflow.add_edge(START, "model")
    workflow.add_node("model", call_model)
    
    memory = MemorySaver()
    return workflow.compile(checkpointer=memory)

# OpenAI 버전
openai_app = create_conversation_app("gpt-4o-mini", "openai")

# Ollama 버전
ollama_app = create_conversation_app("llama3.1", "ollama")

# 동일한 인터페이스로 사용
config = {"configurable": {"thread_id": "test-001"}}

# OpenAI로 대화
response1 = openai_app.invoke(
    {"messages": [{"role": "user", "content": "안녕하세요"}]},
    config
)
print(f"OpenAI: {response1['messages'][-1].content}")

# Ollama로 대화
response2 = ollama_app.invoke(
    {"messages": [{"role": "user", "content": "안녕하세요"}]},
    config
)
print(f"Ollama: {response2['messages'][-1].content}")
```

### 4.2 ConversationBufferWindowMemory 대체 (최근 K개 메시지)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import MemorySaver
from langchain.chat_models import init_chat_model

def create_window_memory_app(model_name: str, model_provider: str, k: int = 5):
    """ConversationBufferWindowMemory 대체: 최근 K개 메시지만 유지"""
    model = init_chat_model(model_name, model_provider=model_provider)
    
    workflow = StateGraph(state_schema=MessagesState)
    
    def call_model_with_window(state: MessagesState):
        # 최근 K개의 메시지만 사용 (시스템 메시지 제외)
        messages = state["messages"]
        if len(messages) > k:
            # 시스템 메시지가 있다면 유지
            system_messages = [m for m in messages if m.type == "system"]
            other_messages = [m for m in messages if m.type != "system"]
            # 최근 K개만 선택
            recent_messages = other_messages[-k:]
            messages = system_messages + recent_messages
        
        response = model.invoke(messages)
        return {"messages": [response]}
    
    workflow.add_edge(START, "model")
    workflow.add_node("model", call_model_with_window)
    
    memory = MemorySaver()
    return workflow.compile(checkpointer=memory)

# OpenAI 버전 (최근 3개 턴만 유지)
openai_window_app = create_window_memory_app("gpt-4o-mini", "openai", k=6)

# Ollama 버전 (최근 3개 턴만 유지)
ollama_window_app = create_window_memory_app("llama3.1", "ollama", k=6)

# 테스트
config = {"configurable": {"thread_id": "window-test-001"}}

# 여러 턴의 대화
for i in range(10):
    response = openai_window_app.invoke(
        {"messages": [{"role": "user", "content": f"메시지 {i+1}"}]},
        config
    )
    print(f"Turn {i+1}: {response['messages'][-1].content}")
```

### 4.3 ConversationSummaryMemory 대체 (요약 기능)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.memory import MemorySaver
from langchain.chat_models import init_chat_model
from langchain_core.messages import SystemMessage, HumanMessage, AIMessage
from typing import Sequence
from typing_extensions import TypedDict

class SummaryState(TypedDict):
    messages: Sequence
    summary: str

def create_summary_memory_app(model_name: str, model_provider: str, max_messages: int = 10):
    """ConversationSummaryMemory 대체: 오래된 메시지를 요약"""
    model = init_chat_model(model_name, model_provider=model_provider)
    
    workflow = StateGraph(state_schema=SummaryState)
    
    def summarize_if_needed(state: SummaryState):
        """메시지가 많으면 요약 생성"""
        messages = state.get("messages", [])
        current_summary = state.get("summary", "")
        
        if len(messages) > max_messages:
            # 오래된 메시지들을 요약
            old_messages = messages[:-max_messages]
            
            # 요약 생성
            summary_prompt = f"""이전 요약: {current_summary}

새로운 대화:
{chr(10).join([f"{m.type}: {m.content}" for m in old_messages])}

위 대화를 간결하게 요약해주세요."""
            
            summary_response = model.invoke([
                {"role": "user", "content": summary_prompt}
            ])
            
            new_summary = summary_response.content
            
            # 최근 메시지만 유지
            return {
                "summary": new_summary,
                "messages": messages[-max_messages:]
            }
        
        return state
    
    def call_model(state: SummaryState):
        """요약을 포함하여 모델 호출"""
        messages = state.get("messages", [])
        summary = state.get("summary", "")
        
        # 요약이 있으면 시스템 메시지에 포함
        if summary:
            system_msg = {
                "role": "system",
                "content": f"이전 대화 요약: {summary}"
            }
            full_messages = [system_msg] + messages
        else:
            full_messages = messages
        
        response = model.invoke(full_messages)
        return {"messages": [response]}
    
    # 그래프 구성
    workflow.add_edge(START, "summarize")
    workflow.add_node("summarize", summarize_if_needed)
    workflow.add_edge("summarize", "model")
    workflow.add_node("model", call_model)
    
    memory = MemorySaver()
    return workflow.compile(checkpointer=memory)

# OpenAI 버전
openai_summary_app = create_summary_memory_app("gpt-4o-mini", "openai", max_messages=6)

# Ollama 버전
ollama_summary_app = create_summary_memory_app("llama3.1", "ollama", max_messages=6)

# 테스트
config = {"configurable": {"thread_id": "summary-test-001"}}

# 긴 대화 시뮬레이션
topics = [
    "제 이름은 김민지입니다.",
    "저는 마케팅 매니저로 일합니다.",
    "SNS 마케팅을 담당하고 있어요.",
    "Instagram과 Facebook을 주로 사용합니다.",
    "데이터 분석도 함께 합니다.",
    "Python으로 리포트를 자동화했어요.",
    "팀은 5명으로 구성되어 있습니다.",
    "월간 캠페인 기획이 주 업무입니다.",
    "최근 신제품 런칭을 준비 중입니다.",
    "다음 달에 대규모 이벤트가 있어요.",
    "제 직업과 사용 도구를 알려주세요."  # 요약된 정보를 활용해야 함
]

for i, topic in enumerate(topics):
    response = openai_summary_app.invoke(
        {
            "messages": [{"role": "user", "content": topic}],
            "summary": ""
        },
        config
    )
    print(f"Turn {i+1}: {topic}")
    print(f"AI: {response['messages'][-1].content}\n")
```

### 4.4 Agent with Memory (OpenAI + Ollama)

```python
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.memory import MemorySaver
from langchain.chat_models import init_chat_model
from langchain_core.tools import tool

@tool
def get_weather(location: str) -> str:
    """Get current weather for a location"""
    # 더미 구현
    return f"{location}의 현재 날씨는 맑음, 기온 22도입니다."

@tool
def search_web(query: str) -> str:
    """Search the web for information"""
    # 더미 구현
    return f"{query}에 대한 검색 결과입니다."

def create_agent_with_memory(model_name: str, model_provider: str):
    """메모리가 있는 에이전트 생성"""
    model = init_chat_model(model_name, model_provider=model_provider)
    memory = MemorySaver()
    
    return create_react_agent(
        model,
        tools=[get_weather, search_web],
        checkpointer=memory
    )

# OpenAI 에이전트
openai_agent = create_agent_with_memory("gpt-4o-mini", "openai")

# Ollama 에이전트
ollama_agent = create_agent_with_memory("llama3.1", "ollama")

# OpenAI 에이전트 테스트
config_openai = {"configurable": {"thread_id": "agent-openai-001"}}

print("=== OpenAI Agent ===")
response1 = openai_agent.invoke(
    {"messages": [{"role": "user", "content": "서울 날씨 알려줘"}]},
    config_openai
)
print(response1["messages"][-1].content)

response2 = openai_agent.invoke(
    {"messages": [{"role": "user", "content": "방금 어느 도시 날씨를 물어봤었지?"}]},
    config_openai
)
print(response2["messages"][-1].content)

# Ollama 에이전트 테스트
config_ollama = {"configurable": {"thread_id": "agent-ollama-001"}}

print("\n=== Ollama Agent ===")
response3 = ollama_agent.invoke(
    {"messages": [{"role": "user", "content": "부산 날씨 알려줘"}]},
    config_ollama
)
print(response3["messages"][-1].content)

response4 = ollama_agent.invoke(
    {"messages": [{"role": "user", "content": "방금 어느 도시 날씨를 물어봤었지?"}]},
    config_ollama
)
print(response4["messages"][-1].content)
```

\<a name="production">\</a>

## 5. 프로덕션 통합 예제 <a href="#id-5" id="id-5"></a>

### 5.1 Complete Production System (OpenAI)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.postgres import PostgresSaver
from langgraph.store.postgres import PostgresStore
from langchain.chat_models import init_chat_model
from langchain.embeddings import init_embeddings
from langchain_core.messages import trim_messages
from langchain_core.runnables import RunnableConfig
from langgraph.store.base import BaseStore
from typing_extensions import TypedDict, Annotated
from langchain_core.messages import BaseMessage
from langgraph.graph.message import add_messages
from typing import Sequence
import uuid
import json
from datetime import datetime

# PostgreSQL 연결
DB_URI = "postgresql://username:password@localhost:5432/production_db?sslmode=disable"

# OpenAI 설정
model = init_chat_model("gpt-4o-mini", model_provider="openai", temperature=0)
embeddings = init_embeddings("openai:text-embedding-3-small")

# 확장된 State
class ProductionState(TypedDict):
    messages: Annotated[Sequence[BaseMessage], add_messages]
    user_id: str
    session_metadata: dict

with (
    PostgresStore.from_conn_string(
        DB_URI,
        index={
            "dims": 1536,
            "embed": embeddings,
            "fields": ["content", "summary", "category"]
        }
    ) as store,
    PostgresSaver.from_conn_string(DB_URI) as checkpointer,
):
    # store.setup()
    # checkpointer.setup()
    
    workflow = StateGraph(state_schema=ProductionState)
    
    def extract_and_store_facts(
        state: ProductionState,
        config: RunnableConfig,
        *,
        store: BaseStore
    ):
        """정보 추출 및 저장"""
        user_id = state["user_id"]
        namespace = ("user_knowledge", user_id)
        
        last_message = state["messages"][-1].content
        
        # LLM을 사용한 정보 추출
        extraction_prompt = f"""다음 메시지에서 장기적으로 기억할 정보를 추출하세요:
"{last_message}"

추출할 정보:
- 개인 정보 (이름, 직업, 위치)
- 선호도 (좋아하는/싫어하는 것)
- 중요 사실

JSON 형식:
{{"facts": ["사실1", "사실2"], "category": "personal_info|preference|skill|other"}}

정보가 없으면 {{"facts": [], "category": "other"}}
"""
        
        extraction_response = model.invoke([
            {"role": "user", "content": extraction_prompt}
        ])
        
        try:
            extracted = json.loads(extraction_response.content)
            facts = extracted.get("facts", [])
            category = extracted.get("category", "other")
            
            for fact in facts:
                memory_id = str(uuid.uuid4())
                store.put(
                    namespace,
                    memory_id,
                    {
                        "content": fact,
                        "summary": last_message[:100],
                        "category": category,
                        "timestamp": datetime.now().isoformat(),
                        "session_id": config["configurable"]["thread_id"]
                    },
                    index=["content", "summary", "category"]
                )
        except Exception as e:
            print(f"Extraction error: {e}")
        
        return state
    
    def retrieve_and_respond(
        state: ProductionState,
        config: RunnableConfig,
        *,
        store: BaseStore
    ):
        """메모리 검색 및 응답 생성"""
        user_id = state["user_id"]
        namespace = ("user_knowledge", user_id)
        
        # 메시지 트리밍
        trimmed_messages = trim_messages(
            state["messages"],
            max_tokens=2000,
            token_counter=model,
            strategy="last",
            include_system=True,
            start_on="human"
        )
        
        last_message = trimmed_messages[-1].content
        
        # 의미 기반 검색
        relevant_memories = store.search(
            namespace,
            query=last_message,
            limit=5
        )
        
        # 카테고리별 정리
        memory_by_category = {}
        for mem in relevant_memories:
            category = mem.value.get("category", "other")
            if category not in memory_by_category:
                memory_by_category[category] = []
            memory_by_category[category].append(
                f"{mem.value['content']} (유사도: {mem.score:.2f})"
            )
        
        # 시스템 프롬프트
        memory_context = ""
        for category, facts in memory_by_category.items():
            memory_context += f"\n[{category}]\n"
            memory_context += "\n".join([f"- {fact}" for fact in facts])
        
        system_prompt = f"""당신은 사용자를 잘 이해하는 AI 어시스턴트입니다.

사용자 정보:
{memory_context if memory_context else "저장된 정보 없음"}

위 정보를 활용하여 개인화된 응답을 제공하세요.
"""
        
        messages = [
            {"role": "system", "content": system_prompt}
        ] + trimmed_messages
        
        response = model.invoke(messages)
        
        return {"messages": [response]}
    
    # 그래프 구성
    workflow.add_edge(START, "extract")
    workflow.add_node("extract", extract_and_store_facts)
    workflow.add_edge("extract", "respond")
    workflow.add_node("respond", retrieve_and_respond)
    
    app = workflow.compile(checkpointer=checkpointer, store=store)
    
    # 프로덕션 인터페이스
    def chat(user_id: str, thread_id: str, message: str, metadata: dict = None):
        """프로덕션용 채팅"""
        config = {
            "configurable": {
                "thread_id": thread_id,
                "user_id": user_id
            }
        }
        
        state = {
            "messages": [{"role": "user", "content": message}],
            "user_id": user_id,
            "session_metadata": metadata or {}
        }
        
        result = app.invoke(state, config)
        return result["messages"][-1].content
    
    # 실행 시나리오
    user_id = "prod-user-openai-001"
    
    print("=== Day 1 ===")
    print(chat(user_id, "day1", "안녕하세요! 저는 이수진이고 UX 디자이너입니다."))
    print(chat(user_id, "day1", "Figma와 Sketch를 주로 사용합니다."))
    print(chat(user_id, "day1", "사용자 리서치에 관심이 많아요."))
    
    print("\n=== Day 2 - New Session ===")
    print(chat(user_id, "day2", "제 직업이 뭐였죠?"))
    print(chat(user_id, "day2", "제가 사용하는 도구를 추천해주세요."))
```

### 5.2 Complete Production System (Ollama)

```````python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.postgres import PostgresSaver
from langgraph.store.postgres import PostgresStore
from langchain.chat_models import init_chat_model
from langchain_ollama import OllamaEmbeddings
from langchain_core.messages import trim_messages, count_tokens_approximately
from langchain_core.runnables import RunnableConfig
from langgraph.store.base import BaseStore
from typing_extensions import TypedDict, Annotated
from langchain_core.messages import BaseMessage
from langgraph.graph.message import add_messages
from typing import Sequence
import uuid
import json
from datetime import datetime

# PostgreSQL 연결
DB_URI = "postgresql://username:password@localhost:5432/production_db?sslmode=disable"

# Ollama 설정
model = init_chat_model("llama3.1", model_provider="ollama", temperature=0)
embeddings = OllamaEmbeddings(model="mxbai-embed-large")

class ProductionState(TypedDict):
    messages: Annotated[Sequence[BaseMessage], add_messages]
    user_id: str
    session_metadata: dict

with (
    PostgresStore.from_conn_string(
        DB_URI,
        index={
            "dims": 1024,
            "embed": embeddings,
            "fields": ["content"]
        }
    ) as store,
    PostgresSaver.from_conn_string(DB_URI) as checkpointer,
):
    # store.setup()
    # checkpointer.setup()
    
    workflow = StateGraph(state_schema=ProductionState)
    
    def extract_and_store_facts(
        state: ProductionState,
        config: RunnableConfig,
        *,
        store: BaseStore
    ):
        """Ollama를 사용한 정보 추출"""
        user_id = state["user_id"]
        namespace = ("user_knowledge", user_id)
        
        last_message = state["messages"][-1].content
        
        extraction_prompt = f"""Extract important information from this message:
"{last_message}"

Extract:
- Personal info (name, job, location)
- Preferences
- Skills

Return JSON:
{{"facts": ["fact1", "fact2"], "category": "personal|preference|skill|other"}}

If nothing to extract: {{"facts": [], "category": "other"}}

Answer in Korean.
"""
        
        extraction_response = model.invoke([
            {"role": "user", "content": extraction_prompt}
        ])
        
        try:
            # JSON 추출 시도
            content = extraction_response.content
            # JSON 부분만 추출
            if "```
                content = content.split("```json").split("```
            elif "```" in content:
                content = content.split("``````")[0]
            
            extracted = json.loads(content.strip())
            facts = extracted.get("facts", [])
            category = extracted.get("category", "other")
            
            for fact in facts:
                store.put(
                    namespace,
                    str(uuid.uuid4()),
                    {
                        "content": fact,
                        "category": category,
                        "timestamp": datetime.now().isoformat()
                    },
                    index=["content"]
                )
        except Exception as e:
            print(f"Extraction error: {e}")
        
        return state
    
    def retrieve_and_respond(
        state: ProductionState,
        config: RunnableConfig,
        *,
        store: BaseStore
    ):
        """메모리 검색 및 응답"""
        user_id = state["user_id"]
        namespace = ("user_knowledge", user_id)
        
        # 메시지 트리밍 (근사 카운팅)
        trimmed_messages = trim_messages(
            state["messages"],
            max_tokens=2000,
            token_counter=count_tokens_approximately,
            strategy="last",
            include_system=True,
            start_on="human"
        )
        
        last_message = trimmed_messages[-1].content
        
        # 벡터 검색
        relevant_memories = store.search(
            namespace,
            query=last_message,
            limit=5
        )
        
        # 메모리 컨텍스트
        memory_context = "\n".join([
            f"- {mem.value['content']}"
            for mem in relevant_memories
        ])
        
        system_prompt = f"""You are a helpful AI assistant.

User information:
{memory_context if memory_context else "No information stored"}

Use this information to provide personalized responses. Answer in Korean.
"""
        
        messages = [
            {"role": "system", "content": system_prompt}
        ] + trimmed_messages
        
        response = model.invoke(messages)
        
        return {"messages": [response]}
    
    # 그래프 구성
    workflow.add_edge(START, "extract")
    workflow.add_node("extract", extract_and_store_facts)
    workflow.add_edge("extract", "respond")
    workflow.add_node("respond", retrieve_and_respond)
    
    app = workflow.compile(checkpointer=checkpointer, store=store)
    
    # 프로덕션 인터페이스
    def chat(user_id: str, thread_id: str, message: str, metadata: dict = None):
        config = {
            "configurable": {
                "thread_id": thread_id,
                "user_id": user_id
            }
        }
        
        state = {
            "messages": [{"role": "user", "content": message}],
            "user_id": user_id,
            "session_metadata": metadata or {}
        }
        
        result = app.invoke(state, config)
        return result["messages"][-1].content
    
    # 실행
    user_id = "prod-user-ollama-001"
    
    print("=== Day 1 ===")
    print(chat(user_id, "day1", "안녕하세요! 저는 강민호이고 데이터 과학자입니다."))
    print(chat(user_id, "day1", "Python과 R을 사용합니다."))
    print(chat(user_id, "day1", "머신러닝 모델 개발이 주 업무입니다."))
    
    print("\n=== Day 2 ===")
    print(chat(user_id, "day2", "제 직업과 사용 기술을 알려주세요."))
```````

### 5.3 Multi-Model Production System (OpenAI + Ollama 하이브리드)

```python
from langgraph.graph import StateGraph, MessagesState, START
from langgraph.checkpoint.postgres import PostgresSaver
from langgraph.store.postgres import PostgresStore
from langchain.chat_models import init_chat_model
from langchain.embeddings import init_embeddings
from langchain_core.runnables import RunnableConfig
from langgraph.store.base import BaseStore
from typing import Literal
from typing_extensions import TypedDict
import uuid

DB_URI = "postgresql://username:password@localhost:5432/hybrid_db?sslmode=disable"

# 두 모델 초기화
openai_model = init_chat_model("gpt-4o-mini", model_provider="openai")
ollama_model = init_chat_model("llama3.1", model_provider="ollama")
embeddings = init_embeddings("openai:text-embedding-3-small")

class HybridState(MessagesState):
    model_provider: Literal["openai", "ollama"]
    user_id: str

with (
    PostgresStore.from_conn_string(
        DB_URI,
        index={
            "dims": 1536,
            "embed": embeddings,
            "fields": ["text"]
        }
    ) as store,
    PostgresSaver.from_conn_string(DB_URI) as checkpointer,
):
    # store.setup()
    # checkpointer.setup()
    
    workflow = StateGraph(state_schema=HybridState)
    
    def call_model(state: HybridState, config: RunnableConfig, *, store: BaseStore):
        """모델 선택 및 장기 메모리 활용"""
        user_id = state.get("user_id")
        namespace = ("hybrid_memories", user_id)
        model_provider = state.get("model_provider", "openai")
        
        # 모델 선택
        model = openai_model if model_provider == "openai" else ollama_model
        
        # 장기 메모리 검색
        last_message = state["messages"][-1].content
        relevant_memories = store.search(
            namespace,
            query=last_message,
            limit=5
        )
        
        # 메모리 컨텍스트
        if relevant_memories:
            memory_text = "\n".join([
                f"- {mem.value.get('text', '')}" for mem in relevant_memories
            ])
            system_msg = {
                "role": "system",
                "content": f"User information:\n{memory_text}\n\nModel: {model_provider.upper()}\n\nProvide personalized responses."
            }
            messages = [system_msg] + state["messages"]
        else:
            messages = state["messages"]
        
        response = model.invoke(messages)
        
        # 중요 정보 저장
        keywords = ["좋아", "싫어", "선호", "기억", "이름", "직업"]
        if any(keyword in last_message for keyword in keywords):
            store.put(
                namespace,
                str(uuid.uuid4()),
                {
                    "text": last_message,
                    "model_used": model_provider
                },
                index=["text"]
            )
        
        return {"messages": [response]}
    
    workflow.add_edge(START, "model")
    workflow.add_node("model", call_model)
    
    app = workflow.compile(checkpointer=checkpointer, store=store)
    
    # 하이브리드 채팅 인터페이스
    def chat(user_id: str, thread_id: str, message: str, model_provider: str = "openai"):
        config = {
            "configurable": {
                "thread_id": thread_id,
                "user_id": user_id
            }
        }
        
        result = app.invoke(
            {
                "messages": [{"role": "user", "content": message}],
                "model_provider": model_provider,
                "user_id": user_id
            },
            config
        )
        return result["messages"][-1].content
    
    # 하이브리드 사용 예제
    user_id = "hybrid-user-001"
    
    print("=== OpenAI Model ===")
    print(chat(user_id, "thread-1", "저는 영화를 좋아합니다.", "openai"))
    print(chat(user_id, "thread-1", "특히 SF 장르를 선호해요.", "openai"))
    
    print("\n=== Ollama Model (same user) ===")
    print(chat(user_id, "thread-2", "제가 좋아하는 것이 뭐였죠?", "ollama"))
    # Ollama 모델도 OpenAI 대화에서 저장된 정보 활용
    
    print("\n=== Back to OpenAI ===")
    print(chat(user_id, "thread-3", "영화 추천해주세요.", "openai"))
```

## 6. 메모리 패턴 비교표 <a href="#id-6" id="id-6"></a>

| 패턴         | v0.x (Deprecated)               | v1.0 (현재)                    | OpenAI 지원 | Ollama 지원 |
| ---------- | ------------------------------- | ---------------------------- | --------- | --------- |
| **기본 버퍼**  | ConversationBufferMemory​       | LangGraph + MemorySaver​     | ✅         | ✅         |
| **윈도우 버퍼** | ConversationBufferWindowMemory​ | Custom reducer​              | ✅         | ✅         |
| **요약**     | ConversationSummaryMemory​      | LLM 기반 요약 노드​                | ✅         | ✅         |
| **토큰 버퍼**  | ConversationTokenBufferMemory​  | trim\_messages​              | ✅         | ✅         |
| **영구 저장**  | N/A                             | PostgresSaver/RedisSaver​    | ✅         | ✅         |
| **장기 메모리** | VectorStoreRetrieverMemory​     | Store + Embeddings​          | ✅         | ✅         |
| **벡터 검색**  | 외부 벡터 DB​                       | InMemoryStore/PostgresStore​ | ✅         | ✅         |
| **에이전트**   | AgentExecutor​                  | create\_react\_agent​        | ✅         | ✅         |

## 7. 설치 및 설정 <a href="#id-7" id="id-7"></a>

### 7-1. 필수 패키지

```bash
# 기본 패키지
pip install -U langgraph langchain-core langchain

# OpenAI 지원
pip install -U langchain-openai

# Ollama 지원
pip install -U langchain-ollama

# PostgreSQL 지원
pip install -U langgraph-checkpoint-postgres langgraph-store-postgres psycopg

# Redis 지원
pip install -U langgraph-checkpoint-redis langgraph-store-redis redis
```

### 7-2. 환경 변수

```bash
bash# OpenAI
export OPENAI_API_KEY="your-openai-api-key"

# Ollama (로컬 실행)
# Ollama 서버가 http://localhost:11434에서 실행 중이어야 함

# PostgreSQL
export DATABASE_URL="postgresql://user:password@localhost:5432/dbname"

# Redis
export REDIS_URL="redis://localhost:6379"
```
