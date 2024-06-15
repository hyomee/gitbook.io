# Kafka-Stream

## 1. Stream 이란

**Stream이란** 데이터의 추상화된 흐름을 의미로 데이터를 연속적으로 처리하기 위한 수단으로 사용됩니다. 자바 1.8에서는  스트림 API를 통해 데이터를 추상화하여 다루며, 배열이나 컬렉션과 같은 다양한 데이터 소스에서 생성할 수 있습니다.

<figure><img src="../../.gitbook/assets/image (350).png" alt=""><figcaption><p>Java Stream</p></figcaption></figure>

스트림을 사용하는 방법은 다음과 같은 순서로 이루어집니다:

1. 스트림 생성&#x20;
2. 중간 연산 (스트림 변환)&#x20;
3. 최종 연산 (스트림 사용)

<details>

<summary>스트림 API의 주요 특징은 다음과 같습니다</summary>

* **내부 반복**: 스트림은 내부 반복을 통해 작업을 수행하므로, 개발자는 요소당 처리해야 할 코드만 제공하면 됩니다.&#x20;
* **단 한 번만 사용 가능**: 스트림은 재사용이 불가능하며, 한 번 사용한 스트림은 다시 사용할 수 없습니다.&#x20;
* **원본 데이터 변경** 없음: 스트림은 데이터 소스를 변경하지 않고, 필터링, 매핑, 정렬 등의 중간 연산을 수행할 수 있습니다.&#x20;
* **지연 연산**: 스트림의 연산은 필터-맵 기반의 API를 사용하여 지연 연산을 통해 성능을 최적화합니다.&#x20;
* **병렬 처리 지원**: parallelStream() 메소드를 통해 손쉬운 병렬 처리를 지원합니다.

</details>

## 2.  **Kafka Streams API 란**

**Kafka Streams API**는 Apache Kafka의 공식적인 스트림 프로세싱 프레임워크입니다. 이 Java 라이브러리는 Kafka 클러스터에 저장된 입력 및 출력 데이터를 사용하여 애플리케이션과 마이크로서비스를 구축하는 데 사용되며 Kafka에 저장된 데이터를 처리하고 분석하는 데 필요한 기능을 제공합니다

Kafka Streams API는 다음과 같은 특징을 제공합니다.

* **탄력적이고 확장 가능**: Kafka Streams API로 구축된 애플리케이션은 탄력적이며, 필요에 따라 확장이 가능합니다.
* **고장 허용성**: 분산 시스템으로 설계되어 있어, 서버나 네트워크 장애에도 견딜 수 있습니다.
* **실시간 처리**: 실시간으로 데이터를 처리하고, 비즈니스의 핵심을 이루는 애플리케이션을 지원합니다.
* **정확한 처리 시맨틱스**: 정확한 한 번만 처리(exactly-once processing)를 보장하여 데이터의 신뢰성을 높입니다.
* **간단한 상태 관리**: 애플리케이션 상태를 효율적으로 관리할 수 있습니다.
* **이벤트 시간과 처리 시간 구분**: 이벤트 발생 시간과 실제 처리 시간을 명확히 구분합니다.
* **윈도우 지원**: 데이터를 특정 시간 범위로 그룹화하여 처리할 수 있는 윈도우 기능을 지원합니다.
* **일반 자바 애플리케이션**: Kafka Streams API로 구축된 애플리케이션은 일반 자바 애플리케이션처럼 패키징, 배포, 모니터링이 가능합니다,

Kafka Streams API의 특징 덕분에, 복잡한 인프라가 없어도 강력한 스트림 처리 애플리케이션을 구축할 수 있는 유용한 도구가 됩니다.



### 2-1 카프카 스트림 구

<figure><img src="../../.gitbook/assets/image (351).png" alt=""><figcaption><p>카프카 스트림 구조 (참고:<a href="https://kafka.apache.org/37/documentation/streams/architecture">https://kafka.apache.org/37/documentation/streams/architecture</a>)</p></figcaption></figure>

참고:  [**AFKA STREAMS**](https://kafka.apache.org/documentation/streams/)

참고: [**Building a Microservices Ecosystem with Kafka Streams and KSQL**](https://www.confluent.io/blog/building-a-microservices-ecosystem-with-kafka-streams-and-ksql/)

**참고:** [**What Is Apache Kafka: Everything You Need To Know About**](https://www.appventurez.com/blog/everything-you-need-to-know-about-apache-kafka)

**참고:** [**무료도서 Concepts and Patterns for Streaming Services with Apache Kafka**](https://www.dbooks.org/designing-event-driven-systems-1492038253/#google\_vignette)
