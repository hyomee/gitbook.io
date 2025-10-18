# LangGraph

LangGraph v1.0은 LangChain 생태계의 핵심 프레임워크로, LLM 기반의 **장기 실행 상태 유지(stateful) 에이전트**를 구축하고 배포하기 위한 저수준 오케스트레이션 프레임워크이다

## 1. 핵심 설계 철학

LangGraph v1.0은 **안정성(stability)에 중점을 둔 릴리즈**로, 기존 코드의 호환성을 유지하면서도 타입 안정성, 개발자 경험, API 일관성을 개선했다. 단순한 체인 구조(chain)로는 구현이 어려웠던 사용자 피드백 루프, 조건부 반복, 복잡한 분기 로직을 **순환(cycle)을 포함한 그래프 구조**로 자연스럽게 표현할 수 있다.​

### 1-1.가지 핵심 구성 요소

#### 1-1-1. State (상태)

State는 그래프의 **메모리이자 공유 데이터 구조**입니다. 각 노드가 실행되는 동안 필요한 모든 정보(대화 기록, 중간 결과, 사용자 정보 등)를 담고 있으며, 그래프의 각 단계를 거치면서 업데이트된다.​

**State 정의 방법**:​

* `TypedDict`를 사용하여 명확한 구조로 정의
* Pydantic 모델이나 dataclass도 지원
* `Annotated` 타입과 함께 **리듀서(Reducer)** 함수를 지정하여 상태 업데이트 방식 제어

```python
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph.message import add_messages
import operator

class MessagesState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]  # 메시지 추가
    llm_calls: int
```

**리듀서(Reducer)**:​\
병렬로 실행되는 여러 노드가 동일한 상태 키에 업데이트를 시도할 때, LangGraph는 리듀서 함수를 사용하여 값을 병합한다. `operator.add`는 리스트나 숫자를 누적하고, 커스텀 리듀서로 복잡한 병합 로직을 구현할 수 있다.​

#### 1-1-2. Nodes (노드)

노드는 **그래프의 작업 단위**. 각 노드는 현재 State를 입력받아 특정 작업(LLM 호출, 도구 사용, 데이터 변환 등)을 수행하고, 업데이트된 State를 반환하는 Python 함수이다.​

```python
from langchain.messages import SystemMessage

def llm_call(state: dict):
    """LLM이 도구 호출 여부를 결정"""
    return {
        "messages": [
            llm_with_tools.invoke(
                [SystemMessage(content="You are a helpful assistant.")] 
                + state["messages"]
            )
        ],
        "llm_calls": state.get('llm_calls', 0) + 1
    }
```

**Prebuilt Components**:​\
LangGraph는 재사용 가능한 사전 구축 컴포넌트를 제공:

* **ToolNode**: 마지막 AI 메시지에서 요청된 도구들을 병렬로 실행
* **tools\_condition**: 도구 호출 여부에 따라 자동으로 라우팅하는 조건부 엣지 함수

#### 1-1-3. Edges (엣지)

엣지는 **노드 간 연결 경로**로, 실행 흐름을 결정합니다. 두 가지 유형이 있다:​

**일반 엣지(Normal Edges)**:​

```python
builder.add_edge(START, "step_1")
builder.add_edge("step_1", "step_2")
```

**조건부 엣지(Conditional Edges)**:​\
현재 상태를 기반으로 다음 노드를 동적으로 결정한다.

```python
def should_continue(state: MessagesState) -> Literal["tool_node", END]:
    last_message = state["messages"][-1]
    if last_message.tool_calls:
        return "tool_node"
    return END

builder.add_conditional_edges("llm_call", should_continue, ["tool_node", END])
```

## 2. 고급 실행 제어

### 2-1. Command 객체

`Command` 객체는 **상태 업데이트와 제어 흐름을 동시에 수행**할 수 있는 강력한 기능으로 노드 내에서 상태를 업데이트하면서 다음 노드를 지정할 수 있다:.

```python
from langgraph.types import Command
from typing import Literal

def my_node(state: State) -> Command[Literal["my_other_node"]]:
    return Command(
        update={"foo": "bar"},        # 상태 업데이트
        goto="my_other_node"          # 다음 노드 지정
    )
```

### 2-2. Send API (동적 병렬 실행)

`Send` API는 **동적으로 워커 노드를 생성하여 병렬 실행**할 때 사용한다. 각 워커는 독립적인 상태를 가지며, 모든 워커의 결과는 공유 상태 키에 병합된다.

```python
from langgraph.types import Send

def continue_to_jokes(state: OverallState):
    return [Send("generate_joke", {"subject": s}) for s in state['subjects']]

graph.add_conditional_edges("node_a", continue_to_jokes)
```

## 3. 실행 알고리즘: Pregel/BSP

LangGraph는 \*\*Pregel 알고리즘(Bulk Synchronous Parallel 모델)\*\*을 기반으로 한다. 실행은 여러 **super-step**으로 구성되며, 각 super-step은 세 단계로 진행된다.

1. **Plan**: 실행할 노드(actors) 선택
2. **Execution**: 선택된 모든 노드를 병렬로 실행
3. **Update**: 채널(channels)에 노드의 결과를 반영

이 구조는 \*\*결정론적 동시성(deterministic concurrency)\*\*을 보장하며, 순환 구조를 완벽히 지원한다.​

## 4.핵심 기능

### 4-1. Persistence (영속성)

**Checkpointer**를 통해 그래프의 상태를 각 super-step마다 자동 저장한다. 이는 대화 히스토리 유지, 오류 복구, 장기 실행 워크플로우를 가능하게 한다.​

**Checkpointer 종류**:​

* **MemorySaver**: 메모리 기반, 실험용
* **SqliteSaver**: SQLite 기반, 로컬 개발용
* **PostgresSaver**: PostgreSQL 기반, 프로덕션용 (파이프라인 모드, 채널별 버저닝 최적화)

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.checkpoint.postgres import PostgresSaver

# 개발 환경
checkpointer = MemorySaver()

# 프로덕션 환경
DB_URI = "postgresql://user:pass@localhost:5432/db"
with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    checkpointer.setup()
    graph = builder.compile(checkpointer=checkpointer)
```

**Thread**: 각 체크포인트를 식별하는 고유 ID입니다. 동일한 `thread_id`로 여러 번 실행하면 대화 히스토리가 누적된다:​

```
pythonconfig = {"configurable": {"thread_id": "conversation_1"}}
graph.invoke({"messages": [{"role": "user", "content": "Hi"}]}, config)
```

**Store (크로스 스레드 메모리)**:​\
`Store` 인터페이스는 **여러 thread 간 정보 공유**를 지원합니다. 사용자별 설정이나 장기 메모리를 저장할 때 유용합니다.​

### 4-2. Human-in-the-Loop

**interrupt** 함수를 사용하여 **그래프 실행을 일시 중지하고 사람의 입력을 받을 수 있다**. 체크포인트가 있으면 언제든 중단 후 재개가 가능하다.​

```python
from langgraph.types import interrupt, Command

@tool
def human_assistance(query: str) -> str:
    """사람의 도움 요청"""
    human_response = interrupt({"query": query})
    return human_response["data"]

# 실행 재개
human_command = Command(resume={"data": "Here's my input"})
graph.stream(human_command, config, stream_mode="values")
```

**interrupt 패턴**:​

* **승인/거부**: 중요한 작업 전 검토
* **상태 수정**: 그래프 상태를 직접 편집
* **도구 호출 검토**: LLM이 요청한 도구 실행 전 검증
* **입력 검증**: 사람의 입력 유효성 확인

### 4-3. Time Travel (시간 여행)

**체크포인트 기록을 탐색하여 과거 상태에서 실행을 재개**할 수 있다. 디버깅, 실험, "만약" 시나리오 탐색에 유용하다.​

```python
# 실행 히스토리 조회
history = graph.get_state_history(config)
for state in history:
    print(f"Checkpoint: {state.config['configurable']['checkpoint_id']}")

# 특정 체크포인트에서 재개
past_config = {"configurable": {
    "thread_id": "1", 
    "checkpoint_id": "specific_checkpoint_id"
}}
graph.invoke(None, past_config)
```

### 4-4. Streaming (스트리밍)

LangGraph는 **5가지 스트리밍 모드**를 지원하며, 동시에 여러 모드를 사용할 수 있다.​

* **values**: 각 단계 후 전체 상태
* **updates**: 각 단계의 상태 변경 사항(델타)
* **messages**: LLM 토큰 + 메타데이터 실시간 스트림
* **custom**: 사용자 정의 데이터 (진행 상황 등)
* **debug**: 상세한 실행 추적

```python
# 여러 모드 동시 사용
for mode, chunk in graph.stream(inputs, stream_mode=["updates", "custom"]):
    print(f"{mode}: {chunk}")
```

## 5. 두 가지 API 스타일

### 5-1. Graph API

**선언적 방식**으로 그래프를 명시적으로 정의한다. 복잡한 워크플로우를 시각화하고 관리하기에 적합하다:​

```python
from langgraph.graph import StateGraph, START, END

builder = StateGraph(MessagesState)
builder.add_node("chatbot", chatbot)
builder.add_node("tools", tool_node)
builder.add_edge(START, "chatbot")
builder.add_conditional_edges("chatbot", tools_condition)
builder.add_edge("tools", "chatbot")
graph = builder.compile(checkpointer=checkpointer)
```

### 5-2. Functional API

**함수형 방식**으로 기존 코드에 LangGraph 기능을 최소한의 변경으로 통합한다. 표준 Python 제어 흐름(`if`, `for`, 함수 호출)을 사용할 수 있다.​

```python
from langgraph.func import entrypoint, task

@entrypoint
def my_workflow(input_data):
    result1 = task(process_step1)(input_data)
    result2 = task(process_step2)(result1)
    return result2
```

**Graph API vs Functional API**:​

* **제어 흐름**: Graph API는 명시적 그래프 구조, Functional API는 Python 기본 구문
* **상태 관리**: Graph API는 명시적 State 선언, Functional API는 함수 스코프로 관리
* **시각화**: Graph API는 시각화 지원, Functional API는 런타임에 동적 생성되어 시각화 불가
* **체크포인트**: 둘 다 지원하지만 생성 방식이 다름

## 6. Subgraphs와 Multi-Agent Systems

**Subgraph**는 그래프를 작은 재사용 가능한 단위로 분리한다. Multi-agent 시스템 구축에 자주 사용된다.​

**공유 상태 스키마**:​

```python
# Subgraph
subgraph_builder = StateGraph(State)
subgraph_builder.add_node(subgraph_node_1)
subgraph = subgraph_builder.compile()

# Parent graph
builder = StateGraph(State)
builder.add_node("node_1", subgraph)  # subgraph를 노드로 추가
graph = builder.compile(checkpointer=checkpointer)
```

**Handoffs (에이전트 전환)**:​\
Multi-agent 아키텍처에서 한 에이전트가 다른 에이전트로 제어를 넘길 때 `Command` 객체를 사용한다:​

```python
def agent(state) -> Command[Literal["agent", "another_agent"]]:
    goto = get_next_agent(...)
    return Command(
        goto=goto,
        update={"my_state_key": "value"}
    )
```

서브그래프에서 부모 그래프로 이동할 때는 `graph=Command.PARENT`를 지정합니다.​

## 7. 프로덕션 배포

### LangGraph Platform (현재 LangSmith Deployment)

프로덕션 환경 배포를 위한 **상용 솔루션**입니다. 주요 구성 요소:​

* **LangGraph Server**: API와 아키텍처 제공
* **LangGraph Studio**: 로컬 개발용 IDE
* **LangGraph CLI**: 명령줄 인터페이스
* **SDK (Python/JS)**: 프로그래밍 방식 상호작용
* **Remote Graph**: 배포된 애플리케이션을 로컬처럼 사용

**배포 옵션**:​

* **Cloud**: 완전 관리형 호스팅 (LangSmith의 일부)
* **Hybrid**: SaaS 제어 플레인, 자체 호스팅 데이터 플레인
* **Self-Hosted**: 완전 자체 인프라 배포

### 관찰성 (Observability)

**LangSmith**와의 네이티브 통합으로 트레이싱, 디버깅, 모니터링을 지원합니다:​

```python
# LangSmith 트레이싱 활성화
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY=<your-api-key>

# 실행 시 메타데이터 추가
config = {
    "run_name": "agent_007",
    "tags": ["production", "critical"],
    "metadata": {"user_id": "12345", "version": "1.0"}
}
graph.invoke(inputs, config)
```

LangSmith는 모든 LLM 호출, 도구 사용, 데이터 변환을 자동으로 추적하며, 실행 단계별 입출력, 소요 시간, 오류를 시각화합니다.​

### 베스트 프랙티스

**상태 설계**:​

* 최소한의 타입 명시된 상태 유지
* 일시적인 값은 함수 스코프에서 처리
* 리듀서는 필요한 경우에만 사용

**노드 함수**:​

* 불변성(immutability) 유지: 입력 변경 대신 부분 업데이트 반환
* 경계에서 검증: 각 노드의 입출력 검증

**순환 제어**:​

* `max_steps` 카운터로 무한 루프 방지
* 지수 백오프로 반복 실패 처리
* 명확한 종료 조건 설정

**스트리밍 최적화**:​

* UI에 맞는 스트리밍 모드 선택
* 독립적인 작업은 Send API로 병렬 실행
* 프롬프트와 컨텍스트 크기 모니터링

**체크포인터 선택**:​

* 개발: MemorySaver 또는 SqliteSaver
* 프로덕션: PostgresSaver (최적화된 읽기/쓰기)
