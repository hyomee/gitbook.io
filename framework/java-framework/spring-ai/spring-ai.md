# Spring AI 기본 구조

Spring AI는 “Spring Boot 스타일로 AI 기능(LLM, Embedding, VectorStore, RAG 등)을 쉽게 쓰도록 만든 프레임워크” 입니다.

Spring AI는 크게 **4개의 핵심 레이어**로 구성됩니다:

1. **Model Layer** (LLM/Embedding 모델)
2. **Client Layer** (ChatClient / EmbeddingClient)
3. **Document & VectorStore Layer** (RAG)
4. **I/O abstraction Layer** (Prompt, Output, Structured Output)

## 1. Model Layer&#x20;

다양한 LLM 및 Embedding 모델 추상화로 Spring AI는 여러 AI 제공자(Provider)를 공통 인터페이스로 감싸서 사용합니다.

#### 지원 모델 예시

* **OpenAI (GPT 계열)**
* **Ollama (로컬 모델)**
* **Azure OpenAI**
* **Anthropic Claude**
* **Google Gemini**
* **Amazon Bedrock**
* **Local Embedding Models**

각 Provider는 내부적으로 같은 인터페이스를 구현합니다.

#### 대표 인터페이스

* `ChatModel`
* `EmbeddingModel`
* `ImageModel` (이미지 생성)

즉, 어떤 모델이든 **같은 코드 패턴**으로 사용 가능합니다.

```java
ChatModel model = ...;
model.call("hello");
```

***

## 2 Client Layer&#x20;

Spring AI에서 애플리케이션이 LLM과 대화할 때 가장 많이 사용하는 것은 **ChatClient** 입니다.

#### ChatClient 특징

* 빌더 패턴 기반
* 시스템 프롬프트, 유저 메시지, 히스토리 등 조합 가능
* 동기(`call()`) / 리액티브 스트리밍(`stream()`) 제공
* Structured Output(JSON → DTO 매핑) 자동 처리

#### 예시

```java
ChatClient chatClient = ...;

String answer = chatClient.prompt()
    .user("Hello?")
    .call()
    .content();
```

Spring AI의 진짜 핵심은 **ChatClient DSL**이라고 보면 됩니다.

***

## 3. Document & VectorStore Layer&#x20;

RAG(Retrieval Augmented Generation)

LLM이 검색 기반 답변을 하려면(사내 문서 Q\&A, PDF 기반 검색 등) 벡터 스토어가 필요합니다.

Spring AI는 이를 통합적으로 제공합니다.

#### 주요 구성 요소

* **Document** (문서 + metadata)
* **EmbeddingModel** (문서 → 벡터 생성)
* **VectorStore** (벡터 인덱싱 및 검색)

#### 지원 VectorStore

* PGVector (Postgres + pgvector)
* Redis Vector Store
* Milvus
* Pinecone
* ElasticSearch
* Chroma 등

#### RAG 동작 순서

1. 문서 → 청크(Chunking)
2. Chunk → EmbeddingModel로 벡터 생성
3. VectorStore에 저장
4. 사용자 질문 → 벡터 검색(similaritySearch)
5. 검색된 문서를 ChatClient 프롬프트에 넣어 응답 생성

```
Document → EmbeddingModel → VectorStore → ChatClient
```

***

## 4. Prompt Layer – PromptTemplate & Output Parser

#### Prompt 구성 요소

* System Prompt (역할 정의)
* User Prompt (사용자 질문)
* Template (파라미터 바인딩 가능)
* ChatRequest / ChatResponse

#### Output 처리 기능 (Structured Output)

LLM이 JSON 형태로 응답하도록 유도하고, 바로 DTO로 매핑할 수 있습니다.

```java
var result = chatClient.prompt()
    .user("...json 형태로 답해줘")
    .call()
    .entity(MyDto.class);
```

LLM→DTO 매핑을 공식적으로 지원하는 것이 Spring AI의 차별점입니다.

***

## 5. Spring AI 아키텍처를 도식화하면

```
┌─────────────────────────┐
│     Spring Application  │
└──────────────┬──────────┘
               │
      (ChatClient / EmbeddingClient)
               │
┌──────────────┴──────────────┐
│      Spring AI Core Layer    │
│  - Prompt                    │
│  - ChatRequest/Response      │
│  - Structured Output         │
└──────────────┬──────────────┘
               │
     ┌─────────┴─────────┐
     │                   │
(LLM Model)         (VectorStore)
(ChatModel)         (PGVector/Redis/…)
(EmbeddingModel)
```

***

#### 요약

| 구성 요소                                 | 설명                       |
| ------------------------------------- | ------------------------ |
| **ChatModel**                         | LLM 호출의 최상위 추상화          |
| **EmbeddingModel**                    | 벡터 임베딩 생성 추상화            |
| **ChatClient**                        | 프롬프트 작성 → 모델 호출 → 응답 구조화 |
| **Document**                          | 콘텐츠 + metadata           |
| **VectorStore**                       | RAG 기반 검색 기능             |
| **Prompt / Template / Output Parser** | 프롬프트와 구조화 응답 처리          |

***

## 6. Spring AI 전체 공통 구조

공통 아키텍처 + Ollama / OpenAI / Gemini(Vertex AI) 별 흐름도도 다음과 같습니다.

### 6.1 Layer 구조

```
[Client / Frontend]
        │ HTTP (REST, SSE, WebSocket)
        ▼
[Controller Layer]  - @RestController (Web or WebFlux)
        │
        ▼
[Service Layer]     - AiChatService, RagService ...
  - ChatClient 사용 (동기 call / 스트리밍 stream)
  - VectorStore, EmbeddingModel 사용 (RAG)
  - MCP Tool 호출 가능 (1.1)
        │
        ▼
[Spring AI Core]
  - ChatClient / ChatModel / EmbeddingModel
  - Prompt / ChatRequest / ChatResponse
  - Document / VectorStore (PGVector 등)
  - MCP Client/Server (tool calling)
        │
        ▼
[Provider Adapter]  (각 Starter)
  - OpenAIChatModel, OllamaChatModel,
    VertexAiGeminiChatModel, ...
        │
        ▼
[External / Local AI Service]
  - OpenAI HTTPS API
  - 로컬 Ollama HTTP 서버
  - Google Vertex AI Gemini API
  - 기타 LLM, Vector DB 등

```

### 6.2 간단한 질문->RAG 답변&#x20;

RAG까지 포함한 한 번의 요청 흐름을 순서로 보면:

1. **HTTP 요청**
   * `/api/chat/rag`로 `{"question":"..."}`
2. **Controller**
   * DTO 바인딩 → `ragService.answerWithRag(question)` 호출
3. **RAG Service**
   1. `VectorStore.similaritySearch(question, k)` 호출
   2. 결과 `List<Document>`를 컨텍스트 텍스트로 병합
   3. ChatClient에 프롬프트 구성
4. **ChatClient (Spring AI Core)**
   * `.prompt().system(...).user(...).call()` 또는 `.stream()`
   * 내부적으로 **ChatModel** 구현체 호출
5. **ChatModel → Provider**
   * OpenAI라면 OpenAI Chat API 호출
   * Ollama라면 로컬 Ollama HTTP 서버 호출
   * Gemini라면 Vertex AI Gemini API 호출
6. **응답 변환**
   * Provider JSON → `ChatResponse`
   * `.entity(AnswerDto.class)`로 DTO 매핑 (Structured Output)
7. **Service → Controller → Client**
   * `AnswerDto`를 REST JSON으로 응답
   * 스트리밍이면 SSE/Flux로 chunk 단위 전송

요게 Spring AI의 \*\*기본 “flowchart”\*\*라고 보면 됩니다.

***

### 6.3. Provider-agnostic 공통 클래스/개념

버전과 상관없이 1.0 / 1.1에서 공통적인 핵심 타입들:

* **ChatModel**
  * 특정 LLM 제공자에 대한 최소 공통 분모 인터페이스.
* **ChatClient**
  * 개발자가 주로 직접 쓰는 DSL (`.prompt().system().user().call()/stream()`).
* **EmbeddingModel**
  * 문서/쿼리 → 벡터로 바꾸는 추상화.
* **Document**
  * `content + metadata(Map<String, Object>)`
* **VectorStore**
  * `add(List<Document>)`, `similaritySearch(query, k)` 같은 인터페이스.
* **MCP(Model Context Protocol) / Tool**
  * 1.1에서 강화된 툴 호출/외부 시스템 연동 레이어.

이 공통 추상화 위에 **각 Provider별 Starter + AutoConfiguration**이 올라갑니다.

***

### 6.4. OpenAI 기반 구조 (spring-ai-starter-model-openai)

#### 6.4.1. 구성 요소

* 의존성:\
  `org.springframework.ai:spring-ai-starter-model-openai`
* Auto-config 클래스:
  * `OpenAiChatModelAutoConfiguration`
  * `OpenAiEmbeddingModelAutoConfiguration` (요약)
* 주요 설정: `spring.ai.model.chat=openai`, `spring.ai.openai.*`

#### 6.4.2. 아키텍처 흐름도 (OpenAI)

```
Controller
  └─ AiChatService
       ├─ ChatClient
       │    └─ OpenAiChatModel
       │          └─ OpenAI Chat Completions HTTPS API
       └─ (옵션) VectorStore (PGVector 등)
              ├─ EmbeddingModel (OpenAI Embeddings)
              └─ Postgres+pgvector
```

* OpenAI Starter가 **OpenAiChatModel**과 **OpenAiEmbeddingModel** 빈을 자동 생성.
* ChatClient는 내부적으로 이 모델들을 사용해 OpenAI의 `/v1/chat/completions`, `/v1/embeddings` 엔드포인트로 HTTPS 요청을 날립니다.

**특징**

* 완전 SaaS, 인터넷/토큰 필수.
* Spring AI 1.1부터 MCP/Retry 설정 등이 더 표준화(예: `spring.ai.retry.*`).

***

### 6.4. Ollama 기반 구조 (spring-ai-starter-model-ollama)

#### 6.4.1. 구성 요소

* 의존성:\
  `org.springframework.ai:spring-ai-starter-model-ollama`
* 설정: `spring.ai.model.chat=ollama`, `spring.ai.ollama.base-url=http://localhost:11434` 등.
* 로컬에서 돌아가는 Ollama 서버가 **LLM + Embedding 엔진** 역할을 수행.

#### 6.4.2. 아키텍처 흐름도 (Ollama + PGVector)

```
Controller
  └─ AiChatService / RagService
       ├─ ChatClient
       │    └─ OllamaChatModel
       │          └─ 로컬 Ollama HTTP 서버
       │                └─ (예: llama3, qwen2 등 로컬 모델)
       └─ VectorStore (PGVector)
            ├─ EmbeddingModel (OllamaEmbeddingModel)
            │    └─ Ollama 서버의 embedding endpoint (예: bge-m3)
            └─ Postgres + pgvector 확장
```

**특징**

* API 키 없이 로컬에서 모델 실행 → 비용/프라이버시 장점.
* 구조는 OpenAI와 동일하지만, **네트워크 홉이 로컬(11434)** 이고 속도/리소스는 내 머신에 의존.
* Spring AI 입장에서는 ChatModel/EmbeddingModel 인터페이스만 보므로, OpenAI ↔ Ollama 교체가 설정만으로 가능.

***

### 6.5. Gemini 기반 구조 (Vertex AI Gemini Starter)

Spring AI 1.1에서는 **Vertex AI Gemini용 Spring Boot Starter**가 별도로 있습니다.

#### 6.5.1. 구성 요소

* 의존성:\
  `org.springframework.ai:spring-ai-vertex-ai-gemini-spring-boot-starter`
* 설정:
  * `spring.ai.model.chat=vertexai-gemini`
  * `spring.ai.vertex.ai.gemini.*` (프로젝트, 위치, 모델 이름 등)
* Gemini는 **멀티모달(text+image+video)** 지원에 특화.

#### 6.5.2. 아키텍처 흐름도 (Gemini)

```
Controller
  └─ GeminiChatService / RagService
       ├─ ChatClient
       │    └─ VertexAiGeminiChatModel
       │          └─ Vertex AI Gemini API
       │                ├─ 텍스트/이미지/비디오 멀티모달 입력
       │                └─ 텍스트/코드 출력
       └─ (옵션) VectorStore
            ├─ EmbeddingModel (Gemini embeddings 또는 별도)
            └─ Vector DB (PGVector, Redis, Pinecone 등)
```

**특징**

* GCP 인증/프로젝트 셋업 필요.
* 멀티모달 Prompt/Media 구조 지원 → Spring AI의 `Media` 타입과 잘 맞물림.

***

### 6.6. Spring AI 1.1에서 추가로 들어온 아키텍처 요소 (MCP 기준)

1. **MCP(Model Context Protocol) Client/Server**
   * LLM이 “툴 호출”을 통해 데이터베이스/HTTP API/파일 시스템 등에 접근할 때,
   * 툴 정의와 실행을 Spring 스타일로 구성.
2. 구조에 넣어 보면:

```
Service
  └─ ChatClient
       ├─ ChatModel (OpenAI/Ollama/Gemini ...)
       └─ MCP Tooling
            ├─ Tool definitions (schema)
            ├─ Tool callback beans (@Component)
            └─ MCP Client/Server (외부 도구/리소스 연결)
```

* 즉, 기존 “LLM + VectorStore” 구조 위에 **툴 호출 레이어**가 하나 더 올라간 형태라고 보면 됩니다.

***

### 6.7. 버전 선택 & 구조 요약

* **1.0.x vs 1.1.x**
  * 기본적인 **ChatClient / ChatModel / VectorStore / EmbeddingModel 구조는 동일**
  * 1.1은 MCP, Provider 추가, 설정 개선, 버그 픽스가 많이 들어간 “성숙한” 버전.
* **Provider에 따라 달라지는 것**
  * 사용하는 **Starter + 설정 prefix + 실제 외부 엔드포인트** 뿐
  * 애플리케이션 서비스/컨트롤러/도메인 설계는 90% 이상 재사용 가능
