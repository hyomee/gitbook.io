# Interrupts

LangGraph v1.0의 **Interrupts** 기능은 에이전트 워크플로우 실행 중 특정 지점에서 **중단**하고 외부 입력(사용자, 시스템)을 받아 **재개**할 수 있게 해 준다. Graph API와 Functional API 양쪽에서 다양한 패턴을 적용할 수 있다.

* **interrupt()**: 노드 실행 중 사람의 개입이 필요한 지점에서 호출
* **Command.resume**: 외부에서 제공된 입력으로 워크플로우를 재개
* **패턴**
  * 검토/승인(interactive approval)
  * 사용자 피드백 삽입
  * 도구 호출 전 검증

***

## 1. `init_chat_model` 초기화 <a href="#undefined" id="undefined"></a>

```python
from langgraph.graph import StateGraph
builder = StateGraph(MyState)

# OpenAI GPT-4
builder.init_chat_model(
    provider="openai",
    model="gpt-4",
    api_key="OPENAI_API_KEY",
    stream=False
)

# Ollama llama2
builder.init_chat_model(
    provider="ollama",
    model="llama2",
    host="http://localhost:11434",
    stream=False
)
```

***

## 2. Graph API 패턴 <a href="#id-2-graph-api" id="id-2-graph-api"></a>

### 2.1. 기본 Interrupt & Resume

```python
from langgraph.graph import StateGraph, START, END
from langgraph.types import interrupt, Command
from langchain.messages import SystemMessage
from typing import TypedDict

class IState(TypedDict):
    prompt: str
    draft: str
    feedback: str

builder = StateGraph(IState)
# init_chat_model 설정 (위 코드 참조)

def draft_node(state: IState) -> dict:
    msg = builder.chat_model.invoke([
        SystemMessage(content=state["prompt"])
    ])
    return {"draft": msg.content}

def feedback_node(state: IState) -> Command:
    # 중단 지점, 'draft'를 클라이언트에 전달 후 재개 대기
    intr = interrupt({"draft": state["draft"]})
    return Command(
        resume=intr["data"],     # 외부 입력 데이터
        update={"feedback": intr["data"]}
    )

def finalize_node(state: IState) -> dict:
    msg = builder.chat_model.invoke([
        SystemMessage(content=state["feedback"])
    ])
    return {"final": msg.content}

builder.add_node("draft", draft_node)
builder.add_node("feedback", feedback_node)
builder.add_node("finalize", finalize_node)
builder.add_edge(START, "draft")
builder.add_edge("draft", "feedback")
builder.add_edge("feedback", "finalize")
builder.add_edge("finalize", END)
graph = builder.compile()

# 워크플로우 시작
result = graph.invoke({"prompt":"블로그 초안 요청"}, {"configurable":{"thread_id":"t1"}})
# 클라이언트에 result["draft"] 전달, 받은 피드백으로 재개
resume_cmd = Command(resume={"data":"피드백 내용"})
final = graph.invoke(resume_cmd, {"configurable":{"thread_id":"t1"}})
```

***

### 2.2. 승인/거부 패턴

```python
def approval_node(state: IState) -> Command:
    choice = interrupt({
        "draft": state["draft"], 
        "options": ["approve","reject"]
    })
    if choice["data"] == "approve":
        return Command(goto="finalize")
    return Command(goto=END)

builder.add_node("approval", approval_node)
builder.add_edge("draft", "approval")
# 이후 엣지: "approval"→"finalize" 및 "approval"→END
```

***

### 2.3. 도구 호출 전 검증

```python
def check_tool_node(state: IState) -> Command:
    # LLM이 요청한 도구 실행 전 사용자 확인
    tool_req = state.get("next_tool")
    intr = interrupt({"tool_request":tool_req})
    if intr["data"] == "yes":
        return Command(goto="tool_execute")
    return Command(goto=END)

# "check_tool" 노드 추가 및 엣지 구성
```

***

### 2.4. 배치 사용자 입력

```python
class BatchState(TypedDict):
    items: list[str]
    feedbacks: list[str]

def batch_feedback(state: BatchState) -> list[Command]:
    cmds = []
    for item in state["items"]:
        intr = interrupt({"item":item})
        cmds.append(Command(
            resume=intr["data"], 
            update={"feedbacks":[intr["data"]]}
        ))
    return cmds

builder.add_node("batch_fb", batch_feedback)
# START→"batch_fb"→END
```

***

## 3. Functional API 패턴 <a href="#id-3-functional-api" id="id-3-functional-api"></a>

```python
from langgraph.func import entrypoint, task, init_chat_model
from langgraph.types import interrupt, Command

# init_chat_model 설정

@task
def draft(query: str) -> str:
    return builder.chat_model.invoke([{"role":"user","content":query}]).content

@task
def finalize(feedback: str) -> str:
    return builder.chat_model.invoke([{"role":"user","content":feedback}]).content

@entrypoint(thread_id="u1")
def workflow(query: str) -> str:
    d = draft(query)
    fb = interrupt({"draft":d})["data"]
    return finalize(fb)

# 실행
response = workflow.invoke("초안 요청")
```

## 4. 통합 예제

LangGraph v1.0의 Interrupts 기능을 **OpenAI**와 **Ollama** API에 적용하고, **FastAPI**를 통해 사용자 피드백을 받아 워크플로우를 중단·재개하는 전 과정을 **클래스 기반**으로 구현한 완성형 예제

### 4.1. 프로젝트 구조 <a href="#id-2" id="id-2"></a>

```textile
.
├── app.py
└── requirements.txt
```

***

### 4.2. requirements.txt <a href="#id-3-requirementstxt" id="id-3-requirementstxt"></a>

```textile
fastapi
uvicorn[standard]
langchain
langgraph
openai
requests
```

***

### 4.3. FastAPI 애플리케이션 (app.py) <a href="#id-4-fastapi--apppy" id="id-4-fastapi--apppy"></a>

```python
import os
import uuid
from typing import TypedDict, Any, Dict
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from fastapi.responses import JSONResponse
from langgraph.graph import StateGraph, START, END
from langgraph.types import Command, interrupt
from langchain.messages import SystemMessage

# 1) 상태 스키마
class InterruptState(TypedDict):
    prompt: str
    draft: str
    feedback: str
    final: str

# 2) FastAPI 요청/응답 모델
class PromptRequest(BaseModel):
    prompt: str

class FeedbackRequest(BaseModel):
    thread_id: str
    checkpoint_id: str
    feedback: str

# 3) 클래스 기반 에이전트
class InterruptAgent:
    def __init__(self):
        # 환경변수 로드
        openai_key = os.getenv("OPENAI_API_KEY", "")
        ollama_host = os.getenv("OLLAMA_HOST", "http://localhost:11434")

        # 그래프 빌더 및 상태 스키마
        self.builder = StateGraph(InterruptState)

        # 챗 모델 초기화 (stream=False)
        self.builder.init_chat_model(
            provider="openai", model="gpt-4",
            api_key=openai_key, stream=False
        )
        self.builder.init_chat_model(
            provider="ollama", model="llama2",
            host=ollama_host, stream=False
        )

        # 3.1) 초안 생성 노드
        def draft_node(state: InterruptState) -> dict:
            msg = self.builder.chat_model.invoke([
                SystemMessage(content=state["prompt"])
            ])
            return {"draft": msg.content}

        # 3.2) 피드백 인터럽트 노드
        def feedback_node(state: InterruptState) -> Command:
            # 워크플로우를 중단하고 피드백 대기
            intr = interrupt({"draft": state["draft"]})
            return Command(resume=intr["data"], update={"feedback": intr["data"]})

        # 3.3) 최종화 노드
        def finalize_node(state: InterruptState) -> dict:
            # 피드백 반영하여 최종 응답 생성
            msg = self.builder.chat_model.invoke([
                SystemMessage(content=state["feedback"])
            ])
            return {"final": msg.content}

        # 3.4) 그래프 구성
        self.builder.add_node("draft", draft_node)
        self.builder.add_node("feedback", feedback_node)
        self.builder.add_node("finalize", finalize_node)
        self.builder.add_edge(START, "draft")
        self.builder.add_edge("draft", "feedback")
        self.builder.add_edge("feedback", "finalize")
        self.builder.add_edge("finalize", END)

        # 3.5) 그래프 컴파일
        self.graph = self.builder.compile()

    def start_workflow(self, prompt: str) -> Dict[str, Any]:
        thread_id = str(uuid.uuid4())
        config = {"configurable": {"thread_id": thread_id}}
        # 초안 생성 후 interrupt 발생 위치까지 실행
        result = self.graph.invoke({"prompt": prompt}, config)
        # checkpoint_id 반환 (interrupt 호출 시 자동 생성)
        checkpoint_id = result["configurable"]["checkpoint_id"]
        return {"thread_id": thread_id, "checkpoint_id": checkpoint_id, "draft": result["draft"]}

    def resume_workflow(self, thread_id: str, checkpoint_id: str, feedback: str) -> str:
        config = {"configurable": {"thread_id": thread_id, "checkpoint_id": checkpoint_id}}
        # resume 피드백 전달
        resume_cmd = Command(resume={"data": feedback})
        result = self.graph.stream(resume_cmd, config, stream_mode=["values"])
        # 마지막 상태 스냅샷에서 'final' 추출
        final_state = None
        for mode, chunk in result:
            if mode == "values":
                final_state = chunk
        if final_state and "final" in final_state:
            return final_state["final"]
        raise HTTPException(status_code=500, detail="Final response not generated")

# 4) FastAPI 앱 및 엔드포인트 정의
app = FastAPI()
agent = InterruptAgent()

@app.post("/chat/draft")
def create_draft(req: PromptRequest):
    data = agent.start_workflow(req.prompt)
    return JSONResponse(content=data)

@app.post("/chat/feedback")
def submit_feedback(req: FeedbackRequest):
    final = agent.resume_workflow(req.thread_id, req.checkpoint_id, req.feedback)
    return JSONResponse(content={"final": final})
```

***

### 4.5. 실행 및 사용법 <a href="#id-5" id="id-5"></a>

1.  의존성 설치

    ```bash
    pip install -r requirements.txt
    ```
2.  환경 변수 설정

    ```bash
    export OPENAI_API_KEY="YOUR_OPENAI_KEY"
    export OLLAMA_HOST="http://localhost:11434"
    ```
3.  FastAPI 서버 실행

    ```bash
    uvicorn app:app --reload --port 8000
    ```
4.  **워크플로우 시작 (초안 생성 및 인터럽트)**

    ```bash
    curl -X POST http://localhost:8000/chat/draft \
      -H "Content-Type: application/json" \
      -d '{"prompt":"블로그 게시글 초안 작성"}'
    ```

    응답 예시:

    ```
    json{
      "thread_id": "uuid-1234",
      "checkpoint_id": "cp-5678",
      "draft": "초안 내용..."
    }
    ```
5.  **피드백 제출 및 최종화**

    ```bash
    curl -X POST http://localhost:8000/chat/feedback \
      -H "Content-Type: application/json" \
      -d '{
        "thread_id":"uuid-1234",
        "checkpoint_id":"cp-5678",
        "feedback":"초안을 좀 더 전문가 톤으로 다듬어주세요."
      }'
    ```

    응답 예시:

    ```
    json{
      "final": "다듬어진 최종 게시글 내용..."
    }
    ```
