# Spring AI MVC WebFlex

Spring AI 입장에서 Web/WebFlex 둘 다 사용할 수 있습니다. 즉 Spring AI 때문에 WebFlux를 꼭 써야 하는 건 아니다 입니다. 이것은 시스템 구조에 따른 선택 사항 입니다.

* Spring AI의 핵심은 **`ChatClient` / `ChatModel` / `VectorStore`** 같은 추상화로.
* 내부적으로는 대부분 **`WebClient`(리액티브 HTTP 클라이언트)** 를 써서 LLM을 호출하지만,
  * **동기 스타일**: `.call()` → 그냥 일반 Service 메서드에서 동기 호출처럼 사용
  * **리액티브 스타일**: `.stream()` → `Flux`로 토큰 스트리밍
* 이 두 가지는 **Spring MVC에서도, WebFlux에서도 모두 사용 가능**합니다

## 1. 언제 Spring MVC

다음에 해당하면 **그냥 `spring-boot-starter-web`(MVC)** 로 가는 게 무난합니다..

1. **현재 서비스가 전통적인 MVC + JPA + JDBC 위주**
   * 대부분의 I/O가 블로킹(JDBC, 외부 REST)이고,
   * 리액티브 생태계를 새로 도입할 여력/필요가 크지 않음.
2. **개발팀이 WebFlux 경험이 거의 없음**
   * `Mono`, `Flux`, backpressure, scheduler 같은 개념을 새로 배우는 비용이 큼.
   * 디버깅/모니터링도 MVC 쪽이 더 익숙함.
3. **AI 응답을 통째로 받아서 한 번에 리턴하는 패턴이 주류**
   * 롱 폴링/스트리밍 없이,
   * “질문 → 몇 초 후 답 한 번에” 구조면 동기 MVC도 충분해요.

```java
AnswerDto answer = chatClient
    .prompt()
    .system("...")
    .user(question)
    .call()
    .entity(AnswerDto.class);
```

## 2. 언제 Spring WebFlex

1.  **LLM 토큰을 바로바로 스트리밍하고 싶다 (SSE/WebSocket)**

    * ChatGPT처럼 글자가 “주르륵” 나오는 UX를 만들고 싶다면,
    * Spring WebFlux + `TEXT_EVENT_STREAM` 가 구현이 깔끔합니다.

    ```java
    @PostMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> stream(@RequestBody ChatRequestDto req) {
        return chatClient
            .prompt()
            .system("...")
            .user(req.question())
            .stream()
            .map(r -> r.getResult().getOutput().getContent());
    }
    ```
2. **전체 서비스가 이미 리액티브 스택으로 설계됨**
   * R2DBC, 리액티브 Redis, 리액티브 Kafka 등과 함께
   * end-to-end non-blocking을 추구하는 아키텍처.
3. **고동시, 많은 클라이언트 연결 + 스트리밍**
   * LLM 스트리밍은 연결 시간이 길어지기 쉬워서,
   * WebFlux + netty 기반으로 적은 쓰레드로 많은 연결 유지가 유리할 수 있음.

## 3. 언제 Spring MVC + WebFlex

* `spring-boot-starter-web` + `spring-boot-starter-webflux` 둘 다 넣으면,
  * 기본적으로 Boot가 **WebFlux(reactive)** 를 우선합니다.
* 한 프로젝트에서 **MVC + WebFlux를 섞어 쓰는 건 가능**하지만,
  * 운영/디버깅/구성 복잡도가 올라가고,
  * 초반에는 오히려 헷갈릴 수 있어요.
* 보통은:
  * POC 단계: 둘 다 써보되,
  * 프로덕션에서는 하나를 메인으로 정하고 `spring.main.web-application-type`도 명시하는 걸 추천합니다.
