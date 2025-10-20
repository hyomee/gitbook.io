# LangChain 및 LangGraph 통합

**LangChain** 체인 호출과 **LangGraph** 그래프 생성 API를 **LangSmith**에 자동 기록하는 워크플로우를 보여준다. LangSmith의 콜백 핸들러를 사용해 각 단계별 입력·출력·메타데이터를 수집한다.

***

### 1. 사전 준비 <a href="#undefined" id="undefined"></a>

1.  패키지 설치

    ```bash
    pip install langsmith langchain langgraph openai requests
    ```
2.  환경 변수 설정

    ```bash
    export LANGSMITH_API_KEY="YOUR_LANGSMITH_API_KEY"
    export OPENAI_API_KEY="YOUR_OPENAI_API_KEY"
    export LANGGRAPH_API_KEY="YOUR_LANGGRAPH_API_KEY"
    ```

***

### 2. LangSmith 콜백 핸들러 초기화 <a href="#langsmith" id="langsmith"></a>

```python
from langsmith import Client, LangSmithCallbackHandler

client = Client()
project_name = "LangChainLangGraphProject"
handler = LangSmithCallbackHandler(
    client=client,
    project_name=project_name,
    tags=["langchain","langgraph"]
)
```

***

### 1. LangChain 연동 예제 <a href="#id-1-langchain" id="id-1-langchain"></a>

```python
from langchain.llms import OpenAI
from langchain import LLMChain, PromptTemplate

# OpenAI LLM에 LangSmith 콜백 추가
llm = OpenAI(model_name="gpt-4", callbacks=[handler])

template = PromptTemplate(
    input_variables=["subject"],
    template="‘{subject}’에 대해 간단히 설명해줘."
)
chain = LLMChain(llm=llm, prompt=template)

# 체인 실행 시 자동 기록
result = chain.run({"subject": "강화학습"})
print("LangChain 응답:", result)
```

* 기록 항목: 입력(subject), LLM 파라미터, 출력 텍스트, 응답 시간, 토큰 사용량 등

***

### 2. LangGraph 연동 예제 <a href="#id-2-langgraph" id="id-2-langgraph"></a>

LangGraph는 입력 텍스트를 기반으로 그래프(노드·엣지)를 생성하는 API입니다. 아래 예제는 생성 과정과 결과를 LangSmith에 기록한다.

```python
import requests
from langsmith import Run

# LangGraph API 설정
LANGGRAPH_ENDPOINT = "https://api.langgraph.ai/v1/graphs/generate"
API_KEY = os.getenv("LANGGRAPH_API_KEY")

def generate_graph_with_langgraph(prompt: str):
    # Run 생성 및 시작
    run = Run(client=client, project_name=project_name, tags=["langgraph"])
    run.start()
    run.log_input("graph_prompt", prompt)

    # API 호출
    payload = {"prompt": prompt, "max_nodes": 10}
    headers = {
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json"
    }
    resp = requests.post(LANGGRAPH_ENDPOINT, json=payload, headers=headers)
    resp.raise_for_status()
    graph_data = resp.json()  # {"nodes": [...], "edges": [...]}

    # 결과 및 메타데이터 기록
    run.log_output("nodes", graph_data["nodes"])
    run.log_output("edges", graph_data["edges"])
    run.log_metadata("max_nodes", payload["max_nodes"])
    run.finish()

    return graph_data

# 예제 실행
graph = generate_graph_with_langgraph("Machine Learning 개념 맵 생성")
print("생성된 그래프 노드:", graph["nodes"])
```

* 기록 항목: 그래프 프롬프트, 파라미터(max\_nodes), 생성된 노드·엣지 목록, 호출 응답 시간

***

### 3. 대시보드 확인 <a href="#id-3" id="id-3"></a>

LangSmith 웹 UI의 **LangChainLangGraphProject** 프로젝트에서 다음을 확인할 수 있다.

* **LangChain Runs**: 체인별 프롬프트, 응답, 토큰·시간 지표 비교
* **LangGraph Runs**: 그래프별 노드·엣지 결과, 파라미터, 실행 시간
* **태그 필터링**: `langchain`, `langgraph`로 분류된 실험을 개별 혹은 비교 조회
