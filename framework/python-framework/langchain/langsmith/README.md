# LangSmith

LangSmith는 언어 모델 실험과 모니터링을 위한 플랫폼으로, LangChain 생태계와 연동되어 모델 호출, 대화 히스토리, 성능 지표 등을 시각화하고 관리할 수 있다. 아래에서는 LangSmith의 주요 개념과 설치·설정 방법을 설명한 뒤, Python SDK를 활용한 예제를 단계별로 제공한다.

***

### 1. LangSmith 주요 개념 <a href="#id-1-langsmith" id="id-1-langsmith"></a>

**1.1 프로젝트(Project)**\
– 모델 실험을 그룹화하는 단위\
– 각 프로젝트에 여러 실험(Experiment)을 생성하고 결과를 관리

**1.2 실험(Experiment)**\
– 모델 호출 시도 하나하나를 기록\
– 입력(prompt), 응답(response), 메타데이터, 성능 지표 등을 포함

**1.3 태그(Tag)**\
– 실험에 라벨을 붙여 필터링 및 비교 가능

**1.4 대시보드(Dashboard)**\
– 웹 UI에서 프로젝트/실험을 시각화\
– 토큰 사용량, 응답 시간, 성공률 등 지표 확인

***

### 2. 설치 및 초기 설정 <a href="#id-2" id="id-2"></a>

```bash
# LangSmith Python SDK 설치
pip install langsmith
```

1. LangSmith 계정 생성 후 API 키 발급
2. 환경 변수에 API 키 설정
   *   Linux/macOS:

       ```bash
       export LANGSMITH_API_KEY="your_api_key_here"
       ```
   *   Windows PowerShell:

       ```powershell
       setx LANGSMITH_API_KEY "your_api_key_here"
       ```
3.  SDK 초기화

    ```python
    from langsmith import Client

    client = Client()  # 환경 변수에서 자동으로 API 키 로드
    ```

***

### 3. 기본 사용 흐름 <a href="#id-3" id="id-3"></a>

1. **프로젝트 생성/선택**
2. **실험 실행 시 로깅 활성화**
3. **입력·출력 기록**
4. **실험 조회 및 분석**

***

### 4. Python SDK 예제 <a href="#id-4-python-sdk" id="id-4-python-sdk"></a>

다음 예제는 OpenAI GPT-4 모델에 프롬프트를 보내고, LangSmith에 호출 로그를 남기는 간단한 워크플로우

```python
from langsmith import Client, Run
import openai
import os

# 1) 환경 변수 설정 (이미 설정된 경우 불필요)
# os.environ["LANGSMITH_API_KEY"] = "your_api_key_here"
# os.environ["OPENAI_API_KEY"] = "your_openai_key_here"

# 2) 클라이언트 초기화
client = Client()

# 3) 프로젝트 선택 또는 생성
project_name = "MyLangSmithProject"
project = client.create_or_get_project(project_name)

# 4) Run 객체 생성 (실험 기록 시작)
run = Run(client=client, project_name=project.name, tags=["test", "gpt4_demo"])
run.start()

# 5) 모델 호출 및 응답 기록
prompt = "안녕하세요, 오늘 날씨 어때?"
run.log_input("prompt", prompt)

response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[{"role": "user", "content": prompt}]
)
reply = response.choices[0].message.content

run.log_output("response", reply)

# 6) 추가 메타데이터 및 지표 기록
run.log_metric("token_usage", response.usage.total_tokens)
run.log_metadata("model", "gpt-4")

# 7) Run 종료 (실험 기록 완료)
run.finish()

print("응답:", reply)
```

### 코드 설명

1. `Client()`를 통해 LangSmith API와 통신 준비
2. `create_or_get_project()`로 프로젝트 생성 또는 기존 프로젝트 로드
3. `Run` 객체로 실험 단위를 시작(`start()`)하고 종료(`finish()`)
4. `log_input()`, `log_output()`, `log_metric()`, `log_metadata()` 메서드로 다양한 정보를 기록
5. LangSmith 웹 대시보드에서 실험 결과 확인

***

### 5. 고급 기능 <a href="#id-5" id="id-5"></a>

* **자동 로깅**: LangChain `CallbackHandler`를 활용하여 체인(chain) 호출 전체를 자동 기록
* **비교 대시보드**: 여러 Run을 태그별로 비교 분석
* **경고 설정**: 특정 지표(예: 응답 시간 초과) 시 알림

### 6. LangChain 연동 예제

```python
from langchain.llms import OpenAI
from langsmith import LangSmithCallbackHandler
from langchain import LLMChain, PromptTemplate

# LangSmith Callback 핸들러 준비
handler = LangSmithCallbackHandler(project_name="MyLangSmithProject", tags=["langchain_demo"])

# LLM 객체에 핸들러 연결
llm = OpenAI(model_name="gpt-4", callbacks=[handler])

# 체인 구성
template = PromptTemplate(input_variables=["topic"], template="'{topic}'에 대해 설명해줘.")
chain = LLMChain(llm=llm, prompt=template)

# 체인 실행 (자동으로 LangSmith에 기록)
result = chain.run({"topic": "강화학습"})
print(result)
```

***

### 7. Ollama API 호출을 LangSmith에 기록하기

#### Python SDK 기본 사용 흐름

1. LangSmith 클라이언트 초기화
2. 프로젝트 생성 또는 선택
3. Run(실험) 시작 및 정보 로깅
4. 모델 호출
5. 입력·출력·메트릭·메타데이터 로깅
6. Run 종료

Ollama 텍스트 완성 API를 호출하고, LangSmith에 입력·출력·토큰 사용량을 기록하는 전체 워크플로우를 보여준다

```python
import os
import json
import requests
from langsmith import Client, Run

# 1) 환경 변수에서 API 키 로드
# os.environ["LANGSMITH_API_KEY"] = "YOUR_LANGSMITH_API_KEY"

# 2) LangSmith 클라이언트 초기화
client = Client()

# 3) 프로젝트 생성 또는 가져오기
project = client.create_or_get_project("OllamaLangSmithProject")

# 4) Run 객체 생성 및 시작
run = Run(client=client, project_name=project.name, tags=["ollama", "text_completion"])
run.start()

# 5) 입력 프롬프트 로깅
prompt = "인공지능이란 무엇인가?"
run.log_input("prompt", prompt)

# 6) Ollama API 호출
OLLAMA_HOST = "http://localhost"
OLLAMA_PORT = 11434
MODEL_NAME = "llama2"

url = f"{OLLAMA_HOST}:{OLLAMA_PORT}/v1/models/{MODEL_NAME}/completions"
payload = {
    "prompt": prompt,
    "max_tokens": 100,
    "temperature": 0.7
}
headers = {"Content-Type": "application/json"}

response = requests.post(url, json=payload, headers=headers)
response.raise_for_status()
data = response.json()
completion = data.get("completion", "")

# 7) 출력 결과 및 메타데이터 로깅
run.log_output("response", completion)
# Ollama가 반환하는 토큰 사용량 필드가 있다면 다음처럼 기록:
# run.log_metric("token_usage", data["usage"]["total_tokens"])

run.log_metadata("model_name", MODEL_NAME)
run.log_metadata("temperature", payload["temperature"])

# 8) Run 종료
run.finish()

print("Ollama 응답:", completion)
```

#### 예제 설명

1. **Client & Run**\
   LangSmith `Client`를 초기화하고, `create_or_get_project()`로 프로젝트를 준비한다.
2. **Run 시작/종료**\
   `run.start()`로 실험 기록을 시작하고, `run.finish()`로 마무리한다.
3. **로그 기록**
   * `log_input("prompt", ...)`으로 입력 프롬프트 저장
   * `log_output("response", ...)`으로 모델 응답 저장
   * `log_metric()`으로 토큰 사용량 등 성능 지표 기록
   * `log_metadata()`으로 모델명, 파라미터 등 부가 정보 기록
4. **Ollama 호출**\
   Python `requests`를 이용해 Ollama `/completions` 엔드포인트에 요청을 보낸다.

### 8. 고급 팁 <a href="#id-5" id="id-5"></a>

* **스트리밍 응답 기록**: Ollama의 스트리밍 API를 사용해 부분별 응답을 받으며, 각 청크를 `log_output_chunk()`로 기록할 수 있다.
* **LangChain 연동**: LangChain 체인에 LangSmith 콜백 핸들러를 추가하면 자동으로 모든 체인 호출을 기록할 수 있다.
* **대시보드 비교**: 태그별로 여러 Run을 필터링해 성능(응답 시간, 토큰 사용량 등)을 시각적으로 비교할 수 있다.

\
<br>
