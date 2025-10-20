# Agent

LangChain v1.0에서는 \*\*`create_agent`\*\*가 Agent 생성의 표준이 되었으며, \*\*`init_chat_model`\*\*을 통해 OpenAI와 Ollama를 포함한 모든 모델을 통합된 방식으로 초기화할 수 있다. 이 가이드에서는 8가지 핵심 Agent 패턴과 실행 가능한 예제 코드를 제공한다.​

## 1. Agent 핵심 개념 <a href="#agent" id="agent"></a>

LangChain v1.0 Agent는 **ReAct (Reasoning + Acting)** 패턴을 따르며, 다음과 같은 루프로 동작한다.​

1. **Model 호출**: LLM이 상황을 분석하고 다음 행동 결정
2. **Tool 실행**: 필요한 도구를 호출하여 정보 수집
3. **반복**: 최종 답변을 생성할 때까지 1-2 반복
4. **종료**: 더 이상 도구가 필요 없으면 최종 응답 반환

## 2. init\_chat\_model 주요 파라미터 정리 <a href="#initchatmodel" id="initchatmodel"></a>

### 2-1. 기본 사용법

```python
from langchain.chat_models import init_chat_model

# 방법 1: model_provider 명시
llm = init_chat_model(
    model="llama3.1:8b",
    model_provider="ollama",
    temperature=0.7,
    base_url="http://localhost:11434"
)

# 방법 2: 콜론(:) 표기법 (권장)
llm = init_chat_model(
    "ollama:llama3.1:8b",
    temperature=0.7,
    base_url="http://localhost:11434"
)

# 방법 3: 자동 추론 (gpt-, claude- 등)
llm = init_chat_model("gpt-4o", temperature=0)  # openai 자동 인식
llm = init_chat_model("claude-3-5-sonnet-latest")  # anthropic 자동 인식
```

### 2-2. 주요 파라미터

| 파라미터                  | 설명                    | 예시                                    |
| --------------------- | --------------------- | ------------------------------------- |
| `model`               | 모델 이름                 | `"gpt-4o"`, `"llama3.1:8b"`           |
| `model_provider`      | 모델 제공자                | `"openai"`, `"ollama"`, `"anthropic"` |
| `temperature`         | 창의성 조절 (0\~1)         | `0.7`                                 |
| `max_tokens`          | 최대 토큰 수               | `1000`                                |
| `base_url`            | API 엔드포인트 (Ollama 전용) | `"http://localhost:11434"`            |
| `api_key`             | API 키 (OpenAI 등)      | `os.getenv("OPENAI_API_KEY")`         |
| `configurable_fields` | 런타임 설정 가능 필드          | `("model", "temperature")`            |
| `config_prefix`       | 설정 키 접두사              | `"llm"`                               |

### 2-3. 지원 모델 제공자​

* `openai` → `langchain-openai`
* `anthropic` → `langchain-anthropic`
* `ollama` → `langchain-ollama`
* `google_genai` → `langchain-google-genai`
* `bedrock` → `langchain-aws`
* `cohere` → `langchain-cohere`
* 기타: `mistralai`, `groq`, `fireworks`, `together`, `deepseek`, `xai` 등

***

### 2-4. 핵심 장점 <a href="#undefined" id="undefined"></a>

**`init_chat_model`의 장점**:​

1. **통일된 인터페이스**: 모든 모델을 동일한 방식으로 초기화
2. **런타임 모델 전환**: 실행 중 모델 변경 가능
3. **자동 추론**: 모델 이름으로 provider 자동 인식
4. **간편한 설정**: import 없이 한 줄로 초기화
5. **LangChain v1.0 표준**: 공식 권장 방법

***

## 3. 환경 설정 <a href="#undefined" id="undefined"></a>

### 3-1. 필수 패키지 설치

```bash
# 기본 LangChain 패키지
pip install -U langchain langchain-core

# Ollama 사용 시
pip install -U langchain-ollama ollama

# OpenAI 사용 시
pip install -U langchain-openai

# 환경 변수 관리용
pip install python-dotenv
```

### 3-2. Ollama 설정

```bash
# Ollama 설치
# macOS: brew install ollama
# Linux: curl -fsSL https://ollama.com/install.sh | sh
# Windows: https://ollama.com 에서 다운로드

# 모델 다운로드
ollama pull llama3.1:8b
ollama pull qwen2:7b-instruct

# Ollama 서버 실행 (백그라운드에서 자동 실행됨)
ollama serve
```

**원격 Ollama 서버 외부 접속 허용**​

```bash
# Ubuntu에서 외부 접속 허용
sudo mkdir -p /etc/systemd/system/ollama.service.d

echo '[Service]' | sudo tee /etc/systemd/system/ollama.service.d/environment.conf
echo 'Environment="OLLAMA_HOST=0.0.0.0:11434"' | sudo tee -a /etc/systemd/system/ollama.service.d/environment.conf

# 방화벽 포트 열기
sudo ufw allow 11434

# 서비스 재시작
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

### 3-3. OpenAI API Key 설정

프로젝트 루트에 `.env` 파일 생성:​

```bash
# .env 파일 내용
OPENAI_API_KEY=sk-your-actual-api-key-here
```

***
