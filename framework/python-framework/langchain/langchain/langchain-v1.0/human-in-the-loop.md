# Human-in-the-loop

\*\*Human-in-the-loop (HITL)\*\*은 AI 에이전트가 중요한 작업을 수행할 때 사람의 승인, 수정, 또는 입력을 받아 진행하는 패턴으로 LangChain v1.0은 두 가지 방식으로 HITL을 지원합니다: **HumanInTheLoopMiddleware**를 통한 자동화된 도구 승인과 **interrupt()** 함수를 통한 세밀한 제어.​



***

## 1. HITL 핵심 개념 <a href="#hitl" id="hitl"></a>

### Human-in-the-loop란?

HITL은 다음과 같은 시나리오에서 필수적입니다:​

* **금융 거래**: 송금, 결제 전 승인
* **데이터베이스 변경**: 삭제, 수정 작업 검토
* **외부 통신**: 이메일 발송, API 호출 승인
* **민감한 정보 처리**: 개인정보 접근 통제

### Interrupt 타입

1. **Dynamic Interrupt**: `interrupt()` 함수로 조건부 중단​
2. **Static Interrupt**: `interrupt_before`/`interrupt_after`로 고정 지점 중단 (디버깅용)​

### Decision 타입

| 결정 타입         | 설명             | 사용 예시               |
| ------------- | -------------- | ------------------- |
| ✅ **approve** | 작업을 그대로 승인     | 이메일 초안을 검토 후 그대로 발송 |
| ✏️ **edit**   | 작업 인자를 수정 후 실행 | 수신자를 변경한 후 이메일 발송   |
| ❌ **reject**  | 작업 거부 및 피드백 제공 | 이메일 초안 거부하고 재작성 요청  |

***

### 1. 설치 및 기본 설정 <a href="#id-1" id="id-1"></a>

```
python# 필수 패키지 설치
pip install --pre -U langchain
pip install -U langchain-openai langchain-ollama
pip install -U langgraph langgraph-checkpoint

# 환경 변수 설정
import os
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"
# Ollama는 로컬에서 실행 (http://localhost:11434)
```

***

## 2. 패턴 1: HumanInTheLoopMiddleware (OpenAI) <a href="#id-2--1-humanintheloopmiddleware-openai" id="id-2--1-humanintheloopmiddleware-openai"></a>

### 2.1 기본 도구 승인 워크플로우

```python
from langchain.agents import create_agent
from langchain.agents.middleware import HumanInTheLoopMiddleware
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langgraph.checkpoint.memory import MemorySaver
from langgraph.types import Command

# init_chat_model로 OpenAI 모델 초기화
model = init_chat_model("openai:gpt-4o", temperature=0.7)

# 도구 정의
@tool
def send_email(to: str, subject: str, body: str) -> str:
    """Send an email to a recipient."""
    print(f"[EMAIL SENT] To: {to}, Subject: {subject}")
    return f"Email sent to {to}"

@tool
def delete_database(table: str) -> str:
    """Delete a database table."""
    print(f"[DATABASE] Deleted table: {table}")
    return f"Table '{table}' deleted"

@tool
def read_data(query: str) -> str:
    """Read data from database."""
    return f"Data results for: {query}"

# Agent 생성 with HITL Middleware
agent = create_agent(
    model=model,
    tools=[send_email, delete_database, read_data],
    middleware=[
        HumanInTheLoopMiddleware(
            interrupt_on={
                "send_email": True,  # approve, edit, reject 모두 허용
                "delete_database": {"allowed_decisions": ["approve", "reject"]},  # edit 불가
                "read_data": False,  # 승인 불필요 (안전한 작업)
            },
            description_prefix="Tool execution pending approval",
        )
    ],
    checkpointer=MemorySaver(),  # 상태 저장 필수
    system_prompt="You are a helpful assistant with email and database tools."
)

# ===== 실행 1: Approve =====
print("\n" + "="*70)
print("Example 1: Approve Email")
print("="*70)

config = {"configurable": {"thread_id": "hitl-001"}}

# 이메일 발송 요청
result1 = agent.invoke(
    {"messages": [{"role": "user", "content": "Send an email to john@example.com with subject 'Meeting' and body 'Let's meet tomorrow'"}]},
    config=config
)

# Interrupt 확인
if "__interrupt__" in result1:
    print("\n[INTERRUPT]")
    interrupt_data = result1["__interrupt__"][0].value
    print(f"Action requests: {interrupt_data['action_requests']}")
    
    # Approve
    result1_resumed = agent.invoke(
        Command(resume={"decisions": [{"type": "approve"}]}),
        config=config
    )
    print("\n[RESULT]")
    print(result1_resumed["messages"][-1].content)

# ===== 실행 2: Edit =====
print("\n" + "="*70)
print("Example 2: Edit Email Before Sending")
print("="*70)

config2 = {"configurable": {"thread_id": "hitl-002"}}

result2 = agent.invoke(
    {"messages": [{"role": "user", "content": "Send an email to alice@example.com about the project update"}]},
    config=config2
)

if "__interrupt__" in result2:
    print("\n[INTERRUPT]")
    interrupt_data = result2["__interrupt__"][0].value
    action = interrupt_data['action_requests'][0]
    print(f"Original: to={action['arguments']['to']}, subject={action['arguments']['subject']}")
    
    # Edit - 수신자와 제목 변경
    result2_resumed = agent.invoke(
        Command(resume={
            "decisions": [{
                "type": "edit",
                "arguments": {
                    "to": "bob@example.com",  # 수신자 변경
                    "subject": "Urgent: Project Update",  # 제목 변경
                    "body": action['arguments']['body']  # 본문 유지
                }
            }]
        }),
        config=config2
    )
    print("\n[RESULT]")
    print(result2_resumed["messages"][-1].content)

# ===== 실행 3: Reject =====
print("\n" + "="*70)
print("Example 3: Reject Database Deletion")
print("="*70)

config3 = {"configurable": {"thread_id": "hitl-003"}}

result3 = agent.invoke(
    {"messages": [{"role": "user", "content": "Delete the users table from database"}]},
    config=config3
)

if "__interrupt__" in result3:
    print("\n[INTERRUPT]")
    interrupt_data = result3["__interrupt__"][0].value
    print(f"Dangerous action: {interrupt_data['action_requests'][0]['name']}")
    
    # Reject
    result3_resumed = agent.invoke(
        Command(resume={
            "decisions": [{
                "type": "reject",
                "explanation": "Deleting the users table is too risky. Please backup first."
            }]
        }),
        config=config3
    )
    print("\n[RESULT]")
    print(result3_resumed["messages"][-1].content)
```

***

## 3. 패턴 2: Direct Interrupt - Approval Workflow (Ollama) <a href="#id-3--2-direct-interrupt---approval-workflow-ollama" id="id-3--2-direct-interrupt---approval-workflow-ollama"></a>

### 3.1 Approve/Reject 패턴

```python
from typing import Literal, TypedDict
from langchain.chat_models import init_chat_model
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, END
from langgraph.types import Command, interrupt

# init_chat_model로 Ollama 모델 초기화
model = init_chat_model("ollama:llama3.1", temperature=0.7)

class TransferState(TypedDict):
    amount: float
    recipient: str
    status: str

def approval_node(state: TransferState) -> Command[Literal["proceed", "cancel"]]:
    """송금 승인 노드"""
    # Interrupt로 승인 요청
    decision = interrupt({
        "question": f"Approve transfer of ${state['amount']} to {state['recipient']}?",
        "details": {
            "amount": state["amount"],
            "recipient": state["recipient"]
        }
    })
    
    # 승인 여부에 따라 라우팅
    if decision:
        return Command(goto="proceed")
    else:
        return Command(goto="cancel")

def proceed_node(state: TransferState):
    """송금 실행"""
    print(f"[TRANSFER] ${state['amount']} sent to {state['recipient']}")
    return {"status": "completed"}

def cancel_node(state: TransferState):
    """송금 취소"""
    print(f"[CANCEL] Transfer cancelled")
    return {"status": "cancelled"}

# 그래프 구축
builder = StateGraph(TransferState)
builder.add_node("approval", approval_node)
builder.add_node("proceed", proceed_node)
builder.add_node("cancel", cancel_node)

builder.add_edge(START, "approval")
builder.add_edge("proceed", END)
builder.add_edge("cancel", END)

# Checkpointer 필수
graph = builder.compile(checkpointer=MemorySaver())

# ===== 실행 1: Approve =====
print("\n" + "="*70)
print("Example 1: Approve Transfer")
print("="*70)

config = {"configurable": {"thread_id": "transfer-001"}}

# 초기 실행 - Interrupt 발생
result = graph.invoke(
    {"amount": 500.0, "recipient": "Alice", "status": "pending"},
    config=config
)

print(f"\n[INTERRUPT] {result['__interrupt__'][0].value}")

# Approve
resumed = graph.invoke(Command(resume=True), config=config)
print(f"[STATUS] {resumed['status']}")

# ===== 실행 2: Reject =====
print("\n" + "="*70)
print("Example 2: Reject Transfer")
print("="*70)

config2 = {"configurable": {"thread_id": "transfer-002"}}

result2 = graph.invoke(
    {"amount": 10000.0, "recipient": "Unknown", "status": "pending"},
    config=config2
)

print(f"\n[INTERRUPT] {result2['__interrupt__'][0].value}")

# Reject
resumed2 = graph.invoke(Command(resume=False), config=config2)
print(f"[STATUS] {resumed2['status']}")
```

***

## 4. 패턴 3: Review and Edit State (OpenAI) <a href="#id-4--3-review-and-edit-state-openai" id="id-4--3-review-and-edit-state-openai"></a>

### 4.1 LLM 출력 검토 및 수정

```python
from typing import TypedDict
from langchain.chat_models import init_chat_model
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, END
from langgraph.types import Command, interrupt

model = init_chat_model("openai:gpt-4o-mini")

class ContentState(TypedDict):
    user_request: str
    generated_text: str
    final_text: str

def generate_node(state: ContentState):
    """LLM으로 컨텐츠 생성"""
    response = model.invoke([
        {"role": "system", "content": "You are a content writer."},
        {"role": "user", "content": state["user_request"]}
    ])
    return {"generated_text": response.content}

def review_node(state: ContentState):
    """사람이 검토 및 수정"""
    # Interrupt로 검토 요청
    edited_content = interrupt({
        "instruction": "Review and edit this generated content",
        "content": state["generated_text"]
    })
    
    return {"final_text": edited_content}

# 그래프 구축
builder = StateGraph(ContentState)
builder.add_node("generate", generate_node)
builder.add_node("review", review_node)

builder.add_edge(START, "generate")
builder.add_edge("generate", "review")
builder.add_edge("review", END)

graph = builder.compile(checkpointer=MemorySaver())

# 실행
print("\n" + "="*70)
print("Example: Review and Edit Generated Content")
print("="*70)

config = {"configurable": {"thread_id": "review-001"}}

# 컨텐츠 생성
result = graph.invoke(
    {"user_request": "Write a short introduction about AI", "generated_text": "", "final_text": ""},
    config=config
)

print(f"\n[GENERATED CONTENT]")
print(result["generated_text"])

print(f"\n[INTERRUPT] {result['__interrupt__'][0].value}")

# 수정된 컨텐츠로 Resume
edited_text = """
AI (Artificial Intelligence) is transforming the world through machine learning, 
natural language processing, and computer vision. It enables machines to learn 
from data and make intelligent decisions.
"""

resumed = graph.invoke(Command(resume=edited_text.strip()), config=config)

print(f"\n[FINAL CONTENT]")
print(resumed["final_text"])
```

***

## 5. 패턴 4: Tool Interrupt (Ollama) <a href="#id-5--4-tool-interrupt-ollama" id="id-5--4-tool-interrupt-ollama"></a>

### 5.1 도구 내부에서 승인 요청

```python
from typing import TypedDict
from langchain.tools import tool
from langchain.chat_models import init_chat_model
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, END
from langgraph.types import Command, interrupt

model = init_chat_model("ollama:llama3.1")

class AgentState(TypedDict):
    messages: list[dict]

# 도구 내부에서 interrupt 호출
@tool
def send_notification(recipient: str, message: str) -> str:
    """Send a notification to a recipient."""
    # Interrupt로 승인 요청
    response = interrupt({
        "action": "send_notification",
        "recipient": recipient,
        "message": message,
        "question": f"Approve sending notification to {recipient}?"
    })
    
    # 승인 결정에 따라 처리
    if response.get("action") == "approve":
        final_recipient = response.get("recipient", recipient)
        final_message = response.get("message", message)
        print(f"[NOTIFICATION] Sent to {final_recipient}: {final_message}")
        return f"Notification sent to {final_recipient}"
    
    return "Notification cancelled by user"

@tool
def create_user(username: str, email: str) -> str:
    """Create a new user account."""
    response = interrupt({
        "action": "create_user",
        "username": username,
        "email": email,
        "question": f"Approve creating user '{username}' with email {email}?"
    })
    
    if response.get("action") == "approve":
        final_username = response.get("username", username)
        final_email = response.get("email", email)
        print(f"[USER CREATED] Username: {final_username}, Email: {final_email}")
        return f"User '{final_username}' created successfully"
    
    return "User creation cancelled"

# 모델에 도구 바인딩
model_with_tools = model.bind_tools([send_notification, create_user])

def agent_node(state: AgentState):
    """Agent 노드"""
    messages = state["messages"]
    response = model_with_tools.invoke(messages)
    return {"messages": messages + [response]}

def should_continue(state: AgentState):
    """도구 호출 여부 확인"""
    last_message = state["messages"][-1]
    if hasattr(last_message, 'tool_calls') and last_message.tool_calls:
        return "tools"
    return END

from langgraph.prebuilt import ToolNode

tool_node = ToolNode([send_notification, create_user])

# 그래프 구축
builder = StateGraph(AgentState)
builder.add_node("agent", agent_node)
builder.add_node("tools", tool_node)

builder.add_edge(START, "agent")
builder.add_conditional_edges("agent", should_continue, {"tools": "tools", END: END})
builder.add_edge("tools", "agent")

graph = builder.compile(checkpointer=MemorySaver())

# ===== 실행 1: Approve =====
print("\n" + "="*70)
print("Example 1: Approve Notification")
print("="*70)

config = {"configurable": {"thread_id": "tool-001"}}

result = graph.invoke(
    {"messages": [{"role": "user", "content": "Send a notification to admin@example.com about system maintenance"}]},
    config=config
)

if "__interrupt__" in result:
    print(f"\n[INTERRUPT] {result['__interrupt__'][0].value}")
    
    # Approve
    resumed = graph.invoke(
        Command(resume={"action": "approve"}),
        config=config
    )
    print(f"\n[RESULT] {resumed['messages'][-1].content}")

# ===== 실행 2: Edit =====
print("\n" + "="*70)
print("Example 2: Edit User Creation")
print("="*70)

config2 = {"configurable": {"thread_id": "tool-002"}}

result2 = graph.invoke(
    {"messages": [{"role": "user", "content": "Create a user with username 'john_doe' and email 'john@example.com'"}]},
    config=config2
)

if "__interrupt__" in result2:
    print(f"\n[INTERRUPT] {result2['__interrupt__'][0].value}")
    
    # Edit - 이메일 변경
    resumed2 = graph.invoke(
        Command(resume={
            "action": "approve",
            "username": "john_doe",
            "email": "john.doe@company.com"  # 이메일 수정
        }),
        config=config2
    )
    print(f"\n[RESULT] {resumed2['messages'][-1].content}")
```

***

## 6. 패턴 5: Validate Human Input (OpenAI) <a href="#id-6--5-validate-human-input-openai" id="id-6--5-validate-human-input-openai"></a>

### 6.1 입력 검증 (여러 번 질문)

```python
from typing import TypedDict
from langchain.chat_models import init_chat_model
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, END
from langgraph.types import interrupt

model = init_chat_model("openai:gpt-4o")

class FormState(TypedDict):
    name: str
    age: int
    email: str

def collect_name_node(state: FormState):
    """이름 수집"""
    prompt = "What is your name?"
    while True:
        answer = interrupt(prompt)
        
        # 검증
        if isinstance(answer, str) and len(answer) > 0:
            return {"name": answer}
        
        prompt = f"'{answer}' is not a valid name. Please enter your name."

def collect_age_node(state: FormState):
    """나이 수집 및 검증"""
    prompt = "What is your age?"
    while True:
        answer = interrupt(prompt)
        
        # 검증
        if isinstance(answer, int) and 0 < answer < 120:
            return {"age": answer}
        
        prompt = f"'{answer}' is not a valid age. Please enter a number between 1 and 119."

def collect_email_node(state: FormState):
    """이메일 수집 및 검증"""
    prompt = "What is your email?"
    while True:
        answer = interrupt(prompt)
        
        # 간단한 이메일 검증
        if isinstance(answer, str) and "@" in answer and "." in answer:
            return {"email": answer}
        
        prompt = f"'{answer}' is not a valid email. Please enter a valid email address."

def summary_node(state: FormState):
    """입력 요약"""
    return {
        "name": state["name"],
        "age": state["age"],
        "email": state["email"]
    }

# 그래프 구축
builder = StateGraph(FormState)
builder.add_node("collect_name", collect_name_node)
builder.add_node("collect_age", collect_age_node)
builder.add_node("collect_email", collect_email_node)
builder.add_node("summary", summary_node)

builder.add_edge(START, "collect_name")
builder.add_edge("collect_name", "collect_age")
builder.add_edge("collect_age", "collect_email")
builder.add_edge("collect_email", "summary")
builder.add_edge("summary", END)

graph = builder.compile(checkpointer=MemorySaver())

# 실행
print("\n" + "="*70)
print("Example: Validate Human Input with Multiple Attempts")
print("="*70)

config = {"configurable": {"thread_id": "form-001"}}

# Step 1: 이름 입력
result1 = graph.invoke({"name": "", "age": 0, "email": ""}, config=config)
print(f"\n[PROMPT] {result1['__interrupt__'][0].value}")

# 빈 이름 입력 (잘못된 입력)
result2 = graph.invoke(Command(resume=""), config=config)
print(f"\n[RETRY] {result2['__interrupt__'][0].value}")

# 올바른 이름 입력
result3 = graph.invoke(Command(resume="Alice"), config=config)
print(f"\n[PROMPT] {result3['__interrupt__'][0].value}")

# 잘못된 나이 입력
result4 = graph.invoke(Command(resume="twenty"), config=config)
print(f"\n[RETRY] {result4['__interrupt__'][0].value}")

# 올바른 나이 입력
result5 = graph.invoke(Command(resume=28), config=config)
print(f"\n[PROMPT] {result5['__interrupt__'][0].value}")

# 잘못된 이메일 입력
result6 = graph.invoke(Command(resume="invalid-email"), config=config)
print(f"\n[RETRY] {result6['__interrupt__'][0].value}")

# 올바른 이메일 입력
final = graph.invoke(Command(resume="alice@example.com"), config=config)

print("\n" + "="*70)
print("[FINAL RESULT]")
print(f"Name: {final['name']}")
print(f"Age: {final['age']}")
print(f"Email: {final['email']}")
```

***

## 7. 고급 패턴: Multi-Agent HITL (OpenAI + Ollama) <a href="#id-7---multi-agent-hitl-openai--ollama" id="id-7---multi-agent-hitl-openai--ollama"></a>

### 7.1 여러 에이전트에서 승인 워크플로우

```python
from typing import TypedDict, Literal
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, START, END
from langgraph.types import Command, interrupt
from langgraph.prebuilt import ToolNode

# 두 모델 초기화
openai_model = init_chat_model("openai:gpt-4o-mini", temperature=0.7)
ollama_model = init_chat_model("ollama:llama3.1", temperature=0.7)

class MultiAgentState(TypedDict):
    messages: list[dict]
    current_agent: str
    approval_required: bool

# 도구 정의
@tool
def process_payment(amount: float, card_number: str) -> str:
    """Process payment with credit card."""
    response = interrupt({
        "action": "process_payment",
        "amount": amount,
        "card_last4": card_number[-4:],
        "question": f"Approve payment of ${amount}?"
    })
    
    if response.get("action") == "approve":
        return f"Payment of ${amount} processed successfully"
    return "Payment cancelled"

@tool
def send_report(recipient: str, report_type: str) -> str:
    """Send report to recipient."""
    response = interrupt({
        "action": "send_report",
        "recipient": recipient,
        "report_type": report_type,
        "question": f"Approve sending {report_type} report to {recipient}?"
    })
    
    if response.get("action") == "approve":
        return f"{report_type} report sent to {recipient}"
    return "Report sending cancelled"

# Agent 1: Payment Agent (OpenAI)
payment_model = openai_model.bind_tools([process_payment])

def payment_agent_node(state: MultiAgentState):
    """Payment processing agent"""
    messages = state["messages"]
    response = payment_model.invoke(messages)
    return {
        "messages": messages + [response],
        "current_agent": "payment"
    }

# Agent 2: Reporting Agent (Ollama)
reporting_model = ollama_model.bind_tools([send_report])

def reporting_agent_node(state: MultiAgentState):
    """Reporting agent"""
    messages = state["messages"]
    response = reporting_model.invoke(messages)
    return {
        "messages": messages + [response],
        "current_agent": "reporting"
    }

# Router
def route_agent(state: MultiAgentState):
    """사용자 요청에 따라 에이전트 선택"""
    last_message = state["messages"][-1]
    content = last_message.get("content", "").lower()
    
    if "payment" in content or "pay" in content:
        return "payment_agent"
    elif "report" in content or "send" in content:
        return "reporting_agent"
    else:
        return END

def should_continue(state: MultiAgentState):
    """도구 호출 여부 확인"""
    last_message = state["messages"][-1]
    if hasattr(last_message, 'tool_calls') and last_message.tool_calls:
        return "tools"
    return END

# Tool Node
tool_node = ToolNode([process_payment, send_report])

# 그래프 구축
builder = StateGraph(MultiAgentState)
builder.add_node("payment_agent", payment_agent_node)
builder.add_node("reporting_agent", reporting_agent_node)
builder.add_node("tools", tool_node)

builder.add_conditional_edges(START, route_agent, {
    "payment_agent": "payment_agent",
    "reporting_agent": "reporting_agent",
    END: END
})

builder.add_conditional_edges("payment_agent", should_continue, {
    "tools": "tools",
    END: END
})

builder.add_conditional_edges("reporting_agent", should_continue, {
    "tools": "tools",
    END: END
})

builder.add_edge("tools", END)

graph = builder.compile(checkpointer=MemorySaver())

# ===== 실행 1: Payment Agent =====
print("\n" + "="*70)
print("Example 1: Payment Agent with Approval")
print("="*70)

config1 = {"configurable": {"thread_id": "multi-001"}}

result1 = graph.invoke(
    {
        "messages": [{"role": "user", "content": "Process a payment of $150 with card number 1234-5678-9012-3456"}],
        "current_agent": "",
        "approval_required": False
    },
    config=config1
)

if "__interrupt__" in result1:
    print(f"\n[INTERRUPT - Payment Agent] {result1['__interrupt__'][0].value}")
    
    # Approve
    resumed1 = graph.invoke(
        Command(resume={"action": "approve"}),
        config=config1
    )
    print(f"\n[RESULT] {resumed1['messages'][-1].content}")

# ===== 실행 2: Reporting Agent =====
print("\n" + "="*70)
print("Example 2: Reporting Agent with Approval")
print("="*70)

config2 = {"configurable": {"thread_id": "multi-002"}}

result2 = graph.invoke(
    {
        "messages": [{"role": "user", "content": "Send a monthly sales report to manager@company.com"}],
        "current_agent": "",
        "approval_required": False
    },
    config=config2
)

if "__interrupt__" in result2:
    print(f"\n[INTERRUPT - Reporting Agent] {result2['__interrupt__'][0].value}")
    
    # Approve
    resumed2 = graph.invoke(
        Command(resume={"action": "approve"}),
        config=config2
    )
    print(f"\n[RESULT] {resumed2['messages'][-1].content}")
```

***

## 8. 고급 패턴: Streaming with HITL (Ollama) <a href="#id-8---streaming-with-hitl-ollama" id="id-8---streaming-with-hitl-ollama"></a>

### 8.1 스트리밍과 HITL 결합

```python
import asyncio
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langgraph.checkpoint.memory import MemorySaver
from langgraph.prebuilt import create_react_agent
from langgraph.types import Command, interrupt

model = init_chat_model("ollama:llama3.1")

@tool
def execute_code(code: str) -> str:
    """Execute Python code."""
    response = interrupt({
        "action": "execute_code",
        "code": code,
        "warning": "This will execute code on your system!",
        "question": "Approve code execution?"
    })
    
    if response.get("action") == "approve":
        # 실제로는 안전한 샌드박스에서 실행
        return f"Code executed successfully: {code[:50]}..."
    return "Code execution cancelled"

agent = create_react_agent(
    model,
    [execute_code],
    checkpointer=MemorySaver()
)

async def streaming_hitl():
    """스트리밍으로 HITL 실행"""
    print("\n" + "="*70)
    print("Example: Streaming with HITL")
    print("="*70)
    
    config = {"configurable": {"thread_id": "stream-001"}}
    
    # 초기 실행 - Interrupt 발생
    print("\n[Streaming Events]")
    async for event in agent.astream(
        {"messages": [{"role": "user", "content": "Write and execute a Python code that prints 'Hello, World!'"}]},
        config=config,
        stream_mode="values"
    ):
        if "__interrupt__" in event:
            print(f"\n[INTERRUPT] {event['__interrupt__'][0].value}")
            break
        
        if "messages" in event:
            last_msg = event["messages"][-1]
            if hasattr(last_msg, 'content') and last_msg.content:
                print(f"[MESSAGE] {last_msg.content[:100]}...")
    
    # Approve and resume
    print("\n[Resuming with approval...]")
    async for event in agent.astream(
        Command(resume={"action": "approve"}),
        config=config,
        stream_mode="values"
    ):
        if "messages" in event:
            last_msg = event["messages"][-1]
            if hasattr(last_msg, 'content'):
                print(f"[RESULT] {last_msg.content}")

# 실행
asyncio.run(streaming_hitl())
```

***

## 9. 실전 예제: Complete HITL System <a href="#id-9---complete-hitl-system" id="id-9---complete-hitl-system"></a>

### 9.1 완전한 HITL 시스템 (OpenAI + Ollama)

```python
from typing import TypedDict, Literal
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.agents.middleware import HumanInTheLoopMiddleware
from langchain.tools import tool
from langgraph.checkpoint.sqlite import SqliteSaver
from langgraph.types import Command, interrupt
import sqlite3

# 두 모델 초기화
openai_model = init_chat_model("openai:gpt-4o", temperature=0.7)
ollama_model = init_chat_model("ollama:llama3.1", temperature=0.7)

# ===== 도구 정의 =====
@tool
def transfer_funds(from_account: str, to_account: str, amount: float) -> str:
    """Transfer funds between accounts."""
    return f"Transferred ${amount} from {from_account} to {to_account}"

@tool
def update_database(table: str, record_id: int, field: str, value: str) -> str:
    """Update database record."""
    return f"Updated {table}.{field} = '{value}' for ID {record_id}"

@tool
def send_bulk_email(recipients: list, subject: str, body: str) -> str:
    """Send email to multiple recipients."""
    return f"Sent email to {len(recipients)} recipients"

@tool
def read_config(config_name: str) -> str:
    """Read configuration (safe operation)."""
    return f"Config value for {config_name}: enabled"

# ===== OpenAI Agent with HITL Middleware =====
openai_agent = create_agent(
    model=openai_model,
    tools=[transfer_funds, update_database, send_bulk_email, read_config],
    middleware=[
        HumanInTheLoopMiddleware(
            interrupt_on={
                "transfer_funds": True,  # approve, edit, reject 모두 허용
                "update_database": {"allowed_decisions": ["approve", "reject"]},
                "send_bulk_email": True,
                "read_config": False,  # 안전한 작업
            },
            description_prefix="🔐 Security approval required",
        )
    ],
    checkpointer=SqliteSaver(sqlite3.connect("hitl_openai.db", check_same_thread=False)),
    system_prompt="""You are a financial assistant with strict security controls.
Always use tools to perform operations. Be cautious with sensitive operations."""
)

# ===== Ollama Agent with Direct Interrupt =====
from langgraph.graph import StateGraph, START, END
from langgraph.prebuilt import ToolNode

class OllamaState(TypedDict):
    messages: list[dict]
    approved_operations: list[str]

@tool
def deploy_service(service_name: str, environment: str) -> str:
    """Deploy service to environment."""
    # Direct interrupt in tool
    response = interrupt({
        "action": "deploy_service",
        "service": service_name,
        "environment": environment,
        "warning": f"Deploying {service_name} to {environment.upper()}!",
        "question": "Approve deployment?"
    })
    
    if response.get("action") == "approve":
        return f"✅ {service_name} deployed to {environment}"
    return f"❌ Deployment cancelled"

ollama_model_with_tools = ollama_model.bind_tools([deploy_service])

def ollama_agent_node(state: OllamaState):
    messages = state["messages"]
    response = ollama_model_with_tools.invoke(messages)
    return {"messages": messages + [response]}

def should_continue_ollama(state: OllamaState):
    last_message = state["messages"][-1]
    if hasattr(last_message, 'tool_calls') and last_message.tool_calls:
        return "tools"
    return END

tool_node_ollama = ToolNode([deploy_service])

builder_ollama = StateGraph(OllamaState)
builder_ollama.add_node("agent", ollama_agent_node)
builder_ollama.add_node("tools", tool_node_ollama)
builder_ollama.add_edge(START, "agent")
builder_ollama.add_conditional_edges("agent", should_continue_ollama, {
    "tools": "tools",
    END: END
})
builder_ollama.add_edge("tools", "agent")

ollama_agent = builder_ollama.compile(
    checkpointer=SqliteSaver(sqlite3.connect("hitl_ollama.db", check_same_thread=False))
)

# ===== 통합 실행 =====
def run_complete_hitl_system():
    print("\n" + "="*70)
    print("COMPLETE HITL SYSTEM - OpenAI + Ollama")
    print("="*70)
    
    # ===== OpenAI Agent: Transfer Funds =====
    print("\n" + "-"*70)
    print("OpenAI Agent: Transfer Funds with HITL Middleware")
    print("-"*70)
    
    config_openai = {"configurable": {"thread_id": "complete-openai-001"}}
    
    result_openai = openai_agent.invoke(
        {"messages": [{"role": "user", "content": "Transfer $5000 from account A123 to account B456"}]},
        config=config_openai
    )
    
    if "__interrupt__" in result_openai:
        interrupt_data = result_openai["__interrupt__"][0].value
        print(f"\n[INTERRUPT]")
        print(f"  Action: {interrupt_data['action_requests'][0]['name']}")
        print(f"  Arguments: {interrupt_data['action_requests'][0]['arguments']}")
        
        # Edit - amount 변경
        resumed_openai = openai_agent.invoke(
            Command(resume={
                "decisions": [{
                    "type": "edit",
                    "arguments": {
                        "from_account": "A123",
                        "to_account": "B456",
                        "amount": 2500.0  # Amount changed
                    }
                }]
            }),
            config=config_openai
        )
        print(f"\n[RESULT] {resumed_openai['messages'][-1].content}")
    
    # ===== OpenAI Agent: Update Database (Reject) =====
    print("\n" + "-"*70)
    print("OpenAI Agent: Update Database (Reject)")
    print("-"*70)
    
    config_openai2 = {"configurable": {"thread_id": "complete-openai-002"}}
    
    result_openai2 = openai_agent.invoke(
        {"messages": [{"role": "user", "content": "Update the users table, set status to 'deleted' for user ID 42"}]},
        config=config_openai2
    )
    
    if "__interrupt__" in result_openai2:
        print(f"\n[INTERRUPT] Dangerous database operation detected")
        
        # Reject
        resumed_openai2 = openai_agent.invoke(
            Command(resume={
                "decisions": [{
                    "type": "reject",
                    "explanation": "Deleting user status is not allowed. Please use archive instead."
                }]
            }),
            config=config_openai2
        )
        print(f"\n[RESULT] {resumed_openai2['messages'][-1].content}")
    
    # ===== Ollama Agent: Deploy Service =====
    print("\n" + "-"*70)
    print("Ollama Agent: Deploy Service with Direct Interrupt")
    print("-"*70)
    
    config_ollama = {"configurable": {"thread_id": "complete-ollama-001"}}
    
    result_ollama = ollama_agent.invoke(
        {"messages": [{"role": "user", "content": "Deploy the payment-service to production"}], "approved_operations": []},
        config=config_ollama
    )
    
    if "__interrupt__" in result_ollama:
        interrupt_data = result_ollama["__interrupt__"][0].value
        print(f"\n[INTERRUPT]")
        print(f"  Service: {interrupt_data['service']}")
        print(f"  Environment: {interrupt_data['environment']}")
        print(f"  Warning: {interrupt_data['warning']}")
        
        # Approve
        resumed_ollama = ollama_agent.invoke(
            Command(resume={"action": "approve"}),
            config=config_ollama
        )
        print(f"\n[RESULT] {resumed_ollama['messages'][-1].content}")
    
    # ===== OpenAI Agent: Safe Operation (No Interrupt) =====
    print("\n" + "-"*70)
    print("OpenAI Agent: Safe Operation (No Interrupt)")
    print("-"*70)
    
    config_openai3 = {"configurable": {"thread_id": "complete-openai-003"}}
    
    result_openai3 = openai_agent.invoke(
        {"messages": [{"role": "user", "content": "Read the database_timeout configuration"}]},
        config=config_openai3
    )
    
    # No interrupt expected
    print(f"\n[RESULT] {result_openai3['messages'][-1].content}")

# 실행
run_complete_hitl_system()
```

***

## 10. Best Practices <a href="#id-10-best-practices" id="id-10-best-practices"></a>

### HITL 설계 원칙

1. **중요한 작업만 Interrupt**: 모든 작업에 승인을 요구하면 사용자 피로도 증가​
2. **명확한 컨텍스트 제공**: Interrupt 페이로드에 충분한 정보 포함​
3. **적절한 Decision 타입 선택**: 금융 거래는 edit 허용, 삭제 작업은 approve/reject만​
4. **Idempotent 작업**: `interrupt()` 전의 코드는 여러 번 실행될 수 있음​

### Interrupt 규칙​

**✅ 해야 할 것:**

* Interrupt 호출 순서를 일관되게 유지
* JSON 직렬화 가능한 값만 사용
* Idempotent 작업을 interrupt 전에 배치
* 특정 예외 타입만 catch

**❌ 하지 말아야 할 것:**

* Interrupt를 bare try/except로 감싸지 않기
* Interrupt 호출을 조건부로 건너뛰지 않기
* 함수나 클래스 인스턴스를 interrupt에 전달하지 않기
* Non-idempotent 작업을 interrupt 전에 배치하지 않기

### Production 체크리스트

1. **Persistent Checkpointer 사용**: `AsyncPostgresSaver`, `SqliteSaver` 등​
2. **Thread ID 관리**: UUID 사용, 사용자/세션별 고유 ID​
3. **타임아웃 설정**: Interrupt가 너무 오래 대기하지 않도록 모니터링
4. **에러 처리**: Resume 실패 시 fallback 로직 구현
5. **로깅**: 모든 승인/거부 결정 기록
