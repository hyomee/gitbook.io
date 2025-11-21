# Spring AI 아키텍처 1

Ollama (OpenAI) + PGVector + Spring AI 1.1 + WebFlux 구조

* Stack
  * Spring Boot **3.4.x**
  * Spring AI **1.1.0**
  * LLM Provider: **Ollama (로컬)** + **OpenAI (클라우드)**
  * Vector DB: **Postgres + PGVector**
  * Web Layer: **Spring WebFlux (Streaming / SSE)**
  * 기능: **RAG + WebFlux Streaming + 멀티 프로바이더(Ollama/OpenAI) 선택**

### 1. 전체 개요

#### 1.1 아키텍처 개념도

```
[Browser / Frontend]
        │  (HTTP, SSE)
        ▼
[WebFlux Controller]  e.g. StreamingChatController
        │
        ▼
[Service Layer]
  - AiChatService          (기본 Chat)
  - RagService             (RAG + PGVector)
  - ProviderRouter         (Ollama / OpenAI 선택)
        │
        ▼
[Spring AI Core]
  - ChatClient
  - ChatModel (OllamaChatModel / OpenAiChatModel)
  - EmbeddingModel (OllamaEmbeddingModel / OpenAiEmbeddingModel)
  - VectorStore (PgVectorStore)
        │
        ▼
[External Systems]
  - Ollama HTTP API (local 11434)
  - OpenAI HTTPS API 
  - PostgreSQL + PGVectorExtension 
```

***

### 2. 패키지 구조

#### 2.1 패키지 트리

```
com.example.ai
 ├─ config
 │   ├─ AiProviderConfig.java          # ChatClient, Provider Router 설정
 │   ├─ WebFluxConfig.java             # WebFlux/SSE 공통 설정 (필요 시)
 │   └─ VectorStoreConfig.java         # VectorStore 래핑/커스터마이징
 ├─ controller
 │   ├─ ChatController.java            # 단건 응답 (Mono)
 │   └─ StreamingChatController.java   # 스트리밍 응답 (Flux, SSE)
 ├─ service
 │   ├─ AiChatService.java             # 공통 Chat 비즈니스 로직
 │   ├─ RagService.java                # RAG (VectorStore + ChatClient)
 │   ├─ ProviderRouter.java            # Ollama / OpenAI 라우팅
 │   └─ DocumentIndexer.java           # 문서 인덱싱 ETL
 ├─ domain
 │   ├─ dto
 │   │   ├─ ChatRequestDto.java        # {"question": "...", "provider": "..."}
 │   │   └─ AnswerDto.java             # {answer, reasoning, sources[]}
 │   └─ entity
 │       └─ SourceDocument.java        # (옵션) 원본 문서 JPA Entity
 └─ AiApplication.java                 # Spring Boot main
```

***

### 3. 클래스 다이어그램

#### 3.1 주요 애플리케이션 클래스 다이어그램 (개념 수준)

```
+---------------------+
| ChatController      |
+---------------------+
| - aiChatService     |
| - ragService        |
+---------------------+
           │ uses
           ▼
+---------------------+        +---------------------+
| AiChatService       |        | RagService          |
+---------------------+        +---------------------+
| - chatClient        |        | - chatClient        |
| - providerRouter    |        | - providerRouter    |
+---------------------+        | - vectorStore       |
           │ uses              +---------------------+
           ▼
+---------------------+
| ProviderRouter      |
+---------------------+
| - ollamaChatClient  |
| - openaiChatClient  |
| - strategy(rule)    |
+---------------------+
```

```
+---------------------+            +---------------------+
| DocumentIndexer     |            | VectorStoreConfig   |
+---------------------+            +---------------------+
| - vectorStore       |<---------->| - vectorStore bean  |
+---------------------+            +---------------------+
```

```
(Spring AI Core)
+------------------------+
| ChatClient (interface) |
+------------------------+
         ▲                  ▲
         │                  │
  +--------------+   +----------------+
  | OllamaClient |   | OpenAiClient   |
  +--------------+   +----------------+
  (ChatModel:           (ChatModel:
   OllamaChatModel)      OpenAiChatModel) 
```

> 실제로는 `ChatClient.Builder` 하나에서 기본 모델을 쓰되, ProviderRouter에서 다른 `ChatClient` 인스턴스를 써도 되고,ChatModel/ChatClient 를 Provider별로 따로 주입해도 됩니다. 여기선 개념적으로 분리

#### 3.2 WebFlux Streaming 관련 클래스

```
+---------------------------+
| StreamingChatController   |
+---------------------------+
| - reactiveService         |
+---------------------------+
           │
           ▼
+---------------------------+
| AiChatReactiveService     |
+---------------------------+
| - chatClient              |
| - providerRouter          |
+---------------------------+
           │
           ▼
    Flux<String> (SSE)
```

***

### 4. Maven 설정 (pom.xml)

#### 4.1 Spring Boot + Spring AI 1.1.0 + WebFlux + PGVector + Ollama + OpenAI

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.1.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- WebFlux (Reactive) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>

    <!-- (선택) 단순 MVC 엔드포인트도 쓸 경우 -->
    <!--
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    -->

    <!-- Validation / Actuator / JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- DB: Postgres + Driver -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
    </dependency>

    <!-- Spring AI: Ollama -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-ollama</artifactId>
    </dependency>

    <!-- Spring AI: OpenAI -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-model-openai</artifactId>
    </dependency>

    <!-- Spring AI: PGVector VectorStore -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-starter-vector-store-pgvector</artifactId>
    </dependency>

    <!-- 테스트 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

* OpenAI / Ollama 스타터는 각각 OpenAiChatModel/OllamaChatModel 을 자동 구성합니다.

***

### 5. application.yml 구성 예시

#### 5.1 공통 (WebFlux + DB)

```yaml
server:
  port: 8080

spring:
  application:
    name: ai-rag-streaming-demo

  main:
    web-application-type: reactive

  datasource:
    url: jdbc:postgresql://localhost:5432/ai_db
    username: ai_user
    password: ai_password

  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        format_sql: true
    show-sql: false
```

#### 5.2 Spring AI – 멀티 프로바이더 (Ollama + OpenAI)

> Spring AI 1.1에서는 모델 provider 를 `spring.ai.model.chat` 으로 지정하고,\
> 각 provider별 설정을 `spring.ai.ollama.*`, `spring.ai.openai.*` 로 분리합니다.

```yaml
spring:
  ai:
    # 기본 Chat/Embedding provider (기본값: openai or ollama 중 선택 가능)
    model:
      chat: ollama        # 기본 LLM은 ollama
      embedding: ollama   # 기본 임베딩도 ollama

    # --- Ollama 설정 ---
    ollama:
      base-url: http://localhost:11434
      init:
        pull-model-strategy: when_missing
      chat:
        options:
          model: llama3         # ollama run llama3
          temperature: 0.2
          num-predict: 1024
      embedding:
        options:
          model: bge-m3         # 임베딩 모델
          # dim: 1024 (PGVector dimension과 일치)

    # --- OpenAI 설정 ---
    openai:
      api-key: ${OPENAI_API_KEY}
      base-url: https://api.openai.com
      chat:
        options:
          model: gpt-4o-mini    # 예시
          temperature: 0.2
          max-tokens: 1024
```

* OpenAI 관련 속성들은 `spring.ai.openai.chat.options.model`, `temperature`, `max-tokens` 등으로 구성됩니다.
* Ollama는 `spring.ai.ollama.chat.options.model` 과 같은 prefix로 세부 옵션을 제어합니다.

#### 5.3 PGVector VectorStore 설정

```yaml
spring:
  ai:
    vectorstore:
      type: pgvector
      pgvector:
        initialize-schema: true         # 1.1부터 기본값 false → 명시해줘야 자동 생성 
        dimension: 1024                 # bge-m3 차원
        distance-type: COSINE_DISTANCE
        index-type: HNSW
        # schema-name: public
        # table-name: vector_store
```

***

### 6. Config / Service / Controller 샘플 코드

#### 6.1 ProviderRouter – Ollama vs OpenAI 선택

```java
package com.example.ai.service;

import lombok.RequiredArgsConstructor;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.stereotype.Component;

@Component
@RequiredArgsConstructor
public class ProviderRouter {

    private final ChatClient ollamaChatClient;  // Builder로부터 구성
    private final ChatClient openaiChatClient;

    public ChatClient route(String provider) {
        if ("openai".equalsIgnoreCase(provider)) {
            return openaiChatClient;
        }
        // default: ollama
        return ollamaChatClient;
    }
}
```

#### 6.2 ChatClient Bean 구성 (Config)

```java
package com.example.ai.config;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.model.ChatModel;
import org.springframework.ai.ollama.OllamaChatModel;
import org.springframework.ai.openai.OpenAiChatModel;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AiProviderConfig {

    // Ollama 기본 ChatClient
    @Bean
    public ChatClient ollamaChatClient(ChatClient.Builder builder,
                                       OllamaChatModel ollamaChatModel) {
        return builder
                .clone()                       // builder 를 복제해서 다른 모델 지정
                .defaultModel(ollamaChatModel)
                .build();
    }

    // OpenAI ChatClient
    @Bean
    public ChatClient openaiChatClient(ChatClient.Builder builder,
                                       OpenAiChatModel openAiChatModel) {
        return builder
                .clone()
                .defaultModel(openAiChatModel)
                .build();
    }
}
```

* `ChatClient.Builder` 는 Spring AI에서 자동 제공되고, 기본적으로 하나의 ChatModel에 바인딩됩니다.
* 여기서는 **Provider별 ChatClient 인스턴스**를 따로 만들어서 ProviderRouter 에서 선택.

#### 6.3 VectorStoreConfig

```java
package com.example.ai.config;

import org.springframework.ai.embedding.EmbeddingModel;
import org.springframework.ai.vectorstore.VectorStore;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class VectorStoreConfig {

    @Bean
    public RagVectorStore ragVectorStore(VectorStore vectorStore,
                                         EmbeddingModel embeddingModel) {
        return new RagVectorStore(vectorStore, embeddingModel);
    }

    public record RagVectorStore(VectorStore vectorStore,
                                 EmbeddingModel embeddingModel) {}
}
```

#### 6.4 DTO

```java
package com.example.ai.domain.dto;

public record AnswerDto(
        String answer,
        String reasoning,
        String[] sources
) {}
```

```java
package com.example.ai.domain.dto;

import jakarta.validation.constraints.NotBlank;

public record ChatRequestDto(
        @NotBlank String question,
        String provider  // "ollama" / "openai" (optional)
) {}
```

#### 6.5 AiChatService (단건 응답 – Mono)

```java
package com.example.ai.service;

import com.example.ai.domain.dto.AnswerDto;
import lombok.RequiredArgsConstructor;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class AiChatService {

    private final ProviderRouter providerRouter;

    private static final String SYSTEM_PROMPT = """
            You are a helpful assistant for Korean users.
            - Answer in Korean.
            - Provide brief reasoning.
            """;

    public AnswerDto ask(String question, String provider) {
        ChatClient client = providerRouter.route(provider);

        return client
                .prompt()
                .system(SYSTEM_PROMPT)
                .user(question)
                .call()
                .entity(AnswerDto.class);
    }
}
```

#### 6.6 WebFlux Streaming 서비스

ChatClient의 스트리밍 API는 `stream()` 이후 Flux를 반환하는 `StreamResponseSpec` 형태로 사용합니다.

```java
package com.example.ai.service;

import lombok.RequiredArgsConstructor;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.chat.client.ChatClientResponse;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;

@Service
@RequiredArgsConstructor
public class AiChatReactiveService {

    private final ProviderRouter providerRouter;

    private static final String SYSTEM_PROMPT = """
            You are a helpful assistant for Korean users.
            - Answer in Korean.
            - Stream your answer gradually.
            """;

    public Flux<String> streamAnswer(String question, String provider) {
        ChatClient client = providerRouter.route(provider);

        return client
                .prompt()
                .system(SYSTEM_PROMPT)
                .user(question)
                .stream()                          // StreamResponseSpec
                .chatClientResponse()              // Flux<ChatClientResponse> 
                .map(ChatClientResponse::getResult)
                .map(result -> result.getOutput().getContent());
    }
}
```

#### 6.7 WebFlux Streaming Controller (SSE)

```java
package com.example.ai.controller;

import com.example.ai.domain.dto.ChatRequestDto;
import com.example.ai.service.AiChatReactiveService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Flux;

@RestController
@RequestMapping("/api/flux/chat")
@RequiredArgsConstructor
public class StreamingChatController {

    private final AiChatReactiveService reactiveService;

    @PostMapping(
        value = "/stream",
        produces = MediaType.TEXT_EVENT_STREAM_VALUE
    )
    public Flux<String> stream(@Valid @RequestBody ChatRequestDto request) {
        return reactiveService.streamAnswer(
                request.question(),
                request.provider()
        );
    }
}
```

***

### 7. RAG + PGVector 흐름 요약 (시퀀스)

**RAG 스트리밍 요청** 플로우(예: `/api/flux/chat/rag-stream` 같은 엔드포인트를 만든 경우):

1. 클라이언트: 질문 + provider(openai/ollama)를 POST (SSE 채널 오픈)
2. Controller: `RagService.streamWithRag(question, provider)` 호출
3. RagService:
   1. `vectorStore.similaritySearch(question, k)` 호출 → Document 리스트
   2. Document를 context 문자열로 합치고 `userMessage` 구성
   3. ProviderRouter로부터 적절한 ChatClient 선택
   4. `chatClient.prompt().system(...).user(userMessage).stream()`
4. ChatClient:
   * 선택된 ChatModel (OllamaChatModel / OpenAiChatModel)로 스트리밍 호출
5. Provider:
   * Ollama: 로컬 HTTP 스트리밍 응답
   * OpenAI: `chat/completions` 스트리밍 엔드포인트
6. Flux가 Controller까지 올라와 SSE로 브라우저에 전송
