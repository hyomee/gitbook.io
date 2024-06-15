# Kafka-Stream

## 1. Stream 이란

**Stream이란** 데이터의 추상화된 흐름을 의미로 데이터를 연속적으로 처리하기 위한 수단으로 사용됩니다. 자바 1.8에서는  스트림 API를 통해 데이터를 추상화하여 다루며, 배열이나 컬렉션과 같은 다양한 데이터 소스에서 생성할 수 있습니다.

<figure><img src="../../../.gitbook/assets/image (350).png" alt=""><figcaption><p>Java Stream</p></figcaption></figure>

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

**Kafka Streams**는 실시간 데이터 스트림 처리를 위해 설계되었으며, 입력 Kafka 토픽을 출력 토픽으로 변환하거나 외부 서비스를 호출하거나 데이터베이스를 갱신하는 등의 작업을 수행합니다.&#x20;

Kafka Streams API는 Apache Kafka의 스트림 처리를 위한 공식 프레임워크입니다. 이 Java 라이브러리를 사용하여 Kafka 클러스터에 저장된 데이터로 애플리케이션과 마이크로서비스를 구축하고, Kafka에 저장된 데이터를 처리 및 분석할 수 있습니다. 다음은 그 특징들입니다.

* **탄력적이고 확장 가능**: Kafka Streams API로 구축된 애플리케이션은 탄력적이며, 필요에 따라 확장이 가능합니다.
* **고장 허용성**: 분산 시스템으로 설계되어 있어, 서버나 네트워크 장애에도 견딜 수 있습니다.
* **실시간 처리**: 실시간으로 데이터를 처리하고, 비즈니스의 핵심을 이루는 애플리케이션을 지원합니다.
* **정확한 처리 시맨틱스**: 정확한 한 번만 처리(exactly-once processing)를 보장하여 데이터의 신뢰성을 높입니다.
* **간단한 상태 관리**: 애플리케이션 상태를 효율적으로 관리할 수 있습니다.
* **이벤트 시간과 처리 시간 구분**: 이벤트 발생 시간과 실제 처리 시간을 명확히 구분합니다.
* **윈도우 지원**: 데이터를 특정 시간 범위로 그룹화하여 처리할 수 있는 윈도우 기능을 지원합니다.
* **일반 자바 애플리케이션**: Kafka Streams API로 구축된 애플리케이션은 일반 자바 애플리케이션처럼 패키징, 배포, 모니터링이 가능합니다,
* **강력한 스트림 처리 애플리케이션 구축**: 복잡한 인프라가 없어도 강력한 스트림 처리 애플리케이션을 구축할 수 있는 유용한 도구가 됩니다.

## 3 카프카 스트림 구조

Kafka Streams는 Kafka 생산자 및 소비자 라이브러리 위에 구축되어 Kafka의 핵심 기능을 활용함으로써 데이터 병렬 처리, 분산 조정, 내결함성 및 운영의 단순화를 통해 애플리케이션 개발을 간소화합니다. 아래 그림은 Kafka Streams 라이브러리를 활용하는 애플리케이션의 구조를 나타냅니다.

<figure><img src="../../../.gitbook/assets/image (351).png" alt=""><figcaption><p>카프카 스트림 구조 (참고:<a href="https://kafka.apache.org/37/documentation/streams/architecture">https://kafka.apache.org/37/documentation/streams/architecture</a>)</p></figcaption></figure>

* **핵심 AP**I: Kafka Streams는 두 가지 주요 API를 제공합니다. KStream은 레코드 스트림을 나타내며, KTable은 변경 가능한 테이블을 나타냅니다.&#x20;
* **Topology**: 스트림 처리를 위한 연산자들이 어떻게 연결되어 데이터를 처리하는지를 나타내는 논리적인 처리 그래프입니다.&#x20;
* **State Management**: Kafka Streams는 로컬 상태 저장소를 사용하여 상태를 유지하며, 이는 고장 허용성과 복원력을 제공합니다.&#x20;
* **Time Concepts**: 이벤트 시간, 로그 시간, 처리 시간을 구분하여 정확한 시간 기반 처리를 지원합니다.
* **Scalability**: Kafka Streams 애플리케이션은 Kafka의 내구성과 확장성을 활용하여 높은 가용성과 확장성을 제공합니다.

### **3-1.** KStream

KStream은 Streams API에서 사용되는 주요 추상화 중 하나이며, 연속적인 레코드 흐름을 표현합니다. 이는 실시간 데이터 이벤트 처리에 활용되고, 각각의 레코드는 키와 값으로 이루어져 있으며, 그 주요 특성은 다음과 같습니다:

* **Stateless processing**: KStream은 각 이벤트 레코드를 독립적으로 처리할 수 있으며, 이전의 상태를 기억하지 않는 Stateless 처리가 가능합니다.&#x20;
* **Stateful processing**: 필요에 따라 이전 이벤트의 상태를 저장하고 이를 활용하여 복잡한 처리를 수행하는 Stateful 처리도 지원합니다.&#x20;
* **Transformation operations**: KStream은 데이터를 필터링, 매핑, 집계 등 다양한 방식으로 변환할 수 있는 연산자를 제공합니다.&#x20;
* **Windowing**: 특정 시간 범위에 대한 데이터를 그룹화하여 처리할 수 있는 윈도잉 기능을 지원합니다.&#x20;
* **Join operation**s: 다른 KStream이나 KTable, GlobalKTable과 조인하여 더 풍부한 데이터 처리를 할 수 있습니다.

### **3-2.** KTable

KTable은 Apache Kafka의 Streams API에서 사용되는 핵심 추상화 중 하나이며, 변화하는 데이터 테이블을 나타냅니다. KTable은 키와 값의 쌍으로 이루어진 업데이트 스트림을 나타내며, 각 키에 대해 최신 값만을 유지합니다. 즉, KTable은 각 키의 최신 상태를 유지하며, 이벤트 스트림의 변화를 반영합니다. 그 주요 특성은 다음과 같습니다:

* **Stateful processing**: KTable은 Stateful 처리를 지원하여, 키의 최신 상태를 유지하고 이를 기반으로 집계나 조인 등의 연산을 수행할 수 있습니다.
* **Change log stream**: Kafka Streams는 KTable의 상태 변경을 내부적으로 변경 로그 토픽에 기록하여, 시스템 장애 발생 시 상태를 복구할 수 있게 합니다.
* **Windowing**: KTable도 윈도잉 연산을 지원하여, 특정 시간 범위에 대한 데이터를 그룹화하여 처리할 수 있습니다
* **Join operations**: KTable은 다른 KTable, KStream 또는 GlobalKTable과 조인을 수행할 수 있어, 더 복잡한 데이터 처리 작업을 가능하게 합니다.

예를 들어, 사용자의 구매 이력을 나타내는 `KTable`을 사용하여, 실시간으로 사용자의 최신 구매 상태를 유지하고, 이를 다른 스트림과 조인하여 사용자에게 맞춤형 광고를 제공하는 애플리케이션을 구축할 수 있습니다

참고: [https://www.slingacademy.com/article/understanding-stateful-and-stateless-processing-in-kafka-streams/](https://www.slingacademy.com/article/understanding-stateful-and-stateless-processing-in-kafka-streams/)

### **3-3. State Management**

Kafka Streams에서 상태 관리는 스트림 처리 애플리케이션에 있어 필수적인 요소입니다. 상태 관리 기능을 통해 애플리케이션은 이전에 계산된 상태를 활용하여 레코드를 처리하고, 집계, 조인, 윈도잉과 같은 상태 의존적 연산을 수행할 수 있습니다.

* **로컬 상태 저장소(Local State Stores)**: Kafka Streams는 각 인스턴스에 로컬 상태 저장소를 유지하여 빠른 상태 접근을 가능하게 합니다. 이 저장소는 키-값 스토어로서 작동하며, 스트림 처리 중 필요한 상태 정보를 저장합니다
* **변경 로그 토픽(Changelog Topics)**: 상태 저장소의 내용은 변경 로그 토픽에도 기록됩니다. 이는 시스템 장애 발생 시 상태를 복구하는 데 사용되며, 고장 허용성(fault-tolerance)을 제공합니다
* **트랜잭션 처리(Transaction Processing)**: Kafka Streams는 트랜잭션 API를 사용하여 메시지를 소스 토픽에서 읽고, 처리한 후 목적지 토픽으로 원자적으로(atomic) 메시지를 전송합니다. 이 과정에서 상태 저장소는 트랜잭션과 일관성을 유지하며 관리됩니다
* **복구 및 재처리(Recovery and Re-processing):** Kafka Streams는 상태 저장소의 데이터가 변경 로그 토픽에 기록되어 있기 때문에, 장애 발생 후에도 상태를 복구하고 필요한 경우 데이터를 재처리할 수 있습니다

예를 들어, 은행 거래를 추적하는 Kafka Streams 애플리케이션은 각 고객별 인출 금액을 상태 저장소에 기록하며, 24시간 동안 특정 금액 이상을 인출할 수 없도록 제한하는 규칙을 적용합니다. 고객이 인출을 시도할 때마다, 애플리케이션은 상태 저장소를 확인하여 규칙을 적용하고, 허용되면 '허용(ALLOW)' 이벤트를, 허용되지 않으면 '거부(DENY)' 이벤트를 생성합니다.

상태 관리 기능은 Kafka Streams를 활용하여 복잡한 스트림 처리 애플리케이션을 구축하는 데 필수적입니다. 개발자들은 이 기능들을 통해 실시간 데이터 처리, 복잡한 이벤트 처리, 그리고 정확한 데이터 분석 작업을 수행할 수 있습니다.

참고: [https://www.slingacademy.com/article/understanding-stateful-and-stateless-processing-in-kafka-streams/](https://www.slingacademy.com/article/understanding-stateful-and-stateless-processing-in-kafka-streams/)

### **3.3. Time Concepts**

Kafka Streams에서 'Time Concepts'은 스트림 처리에서 중요한 역할을 담당합니다. Kafka Streams는 주로 세 가지 시간 개념이 있습니다.

* **이벤트 시간(Event Time)**: 이벤트 타임스탬프는 이벤트가 실제로 발생한 시간을 나타냅니다. 프로듀서는 이벤트를 생성할 때 타임스탬프를 할당하며, 이는 이벤트의 실제 시간을 반영합니다. 예를 들어 센서 데이터의 경우, 센서가 측정을 수행한 정확한 시간이 이벤트의 시간으로 기록됩니다.
* **로그 추가 시간(Log-Append Time)**: 레코드가 브로커에 도착했을 때 브로커가 레코드를 로그에 추가하면서 부여하는 타임스탬프입니다. 이는 브로커 환경의 현재 시간을 기준으로 설정됩니다
* **스트림 시간(Stream Time)**: Kafka Streams에서는 스트림 시간이라는 개념을 사용합니다. 스트림 시간은 처리 중인 레코드들 중 가장 큰 타임스탬프를 기준으로 하며, 이 시간은 앞으로만 진행되고 뒤로 가지 않습니다. 만약 순서가 뒤바뀐 레코드가 도착하더라도 (즉, 현재 스트림 시간보다 이전이지만 여전히 윈도우와 유예 기간 내에 있는 경우), 스트림 시간은 그대로 유지됩니다\
  \
  이러한 시간 개념들은 Kafka Streams에서 윈도잉 연산(Windowing Operations)과 같은 시간에 민감한 처리를 수행할 때 중요합니다. 예를 들어, 특정 시간 범위 내의 데이터를 집계하거나, 이벤트가 발생한 순서대로 처리를 해야 할 때 이러한 시간 개념들이 사용됩니다. Kafka Streams는 TimeStampExtractor 인터페이스를 사용하여 현재 레코드에서 타임스탬프를 추출하며, 이는 윈도잉 연산이나 지연된 레코드 처리 등에 영향을 미칩니다

이 시간 개념들은 Kafka Streams가 실시간 데이터 처리를 정확하고 효율적으로 수행할 수 있도록 돕습니다. 개발자는 이러한 개념들을 이해하고 적절히 활용함으로써, 복잡한 스트림 처리 애플리케이션을 구축할 수 있습니다.

참고: [https://developer.confluent.io/courses/kafka-streams/time-concepts/](https://developer.confluent.io/courses/kafka-streams/time-concepts/)

### 3-4. **Scalability**

Kafka Streams에서의 확장성(Scalability)은 다음과 같은 특징과 원리에 기반합니다:

* 분산 처리: Kafka Streams는 Kafka의 분산 특성을 활용하여, 여러 노드에 걸쳐 수평적으로 확장할 수 있습니다. 이를 통해 대량의 데이터와 높은 처리량을 다룰 수 있으며, 데이터 파티션을 여러 브로커(서버)에 걸쳐 복제함으로써 고장 허용성을 보장합니다
* 엘라스틱 스케일링: Kafka Streams 애플리케이션은 실행 중에도 처리 용량을 동적으로 추가하거나 제거할 수 있습니다. 이는 다운타임이나 데이터 손실 없이 애플리케이션의 탄력성을 높이고, 필요에 따라 유지보수를 수행할 수 있게 합니다.
* 태스크 기반 병렬성: Kafka Streams의 병렬 처리 단위는 스트림 태스크입니다. 태스크는 단일 Kafka 파티션에서 데이터를 소비하고, 처리 그래프를 통해 레코드를 처리합니다. 상태 저장이 필요한 처리의 경우, 태스크는 상태 저장소에 쓰고 하나 이상의 Kafka 파티션으로 다시 데이터를 전송합니다. 확장성을 높이기 위해 주로 사용되는 조정 방법은 토픽의 파티션 수를 늘리는 것입니다
* 자동 로드 밸런싱: Kafka Streams 인스턴스를 시작할 때 필요한 파티션 수를 결정하면, 나머지는 자동으로 이루어집니다. 태스크는 사용자의 개입 없이 소비자 그룹 관리 기능 덕분에 로드가 균형 있게 분배됩니다.

확장성을 높이기 위해 추가적인 Kafka Streams 인스턴스를 시작하거나, 필요가 줄어들면 인스턴스를 중지할 수 있습니다. 인스턴스를 추가하면, 해당 애플리케이션의 인스턴스들은 서로 인식하고 자동으로 처리 작업을 공유하기 시작합니다. 반대로 인스턴스를 중지하면, 남아 있는 인스턴스들은 다른 인스턴스가 중지되었음을 인식하고 중지된 인스턴스의 처리 작업을 자동으로 인계받습니다

이러한 확장성은 Kafka Streams를 사용하여 대규모 스트림 처리 애플리케이션을 구축할 때 매우 중요한 요소입니다. 개발자는 이러한 특징을 활용하여 애플리케이션의 처리 용량을 필요에 따라 유연하게 조정할 수 있습니다

***

참고:  [**AFKA STREAMS**](https://kafka.apache.org/documentation/streams/)

**참고:**  [**An Introduction to Apache Kafka & Event Streamin**](https://www.slingacademy.com/article/an-introduction-to-apache-kafka-event-streaming/)[**g**  ](https://www.slingacademy.com/article/an-introduction-to-apache-kafka-event-streaming/)

참고: [**Building a Microservices Ecosystem with Kafka Streams and KSQL**](https://www.confluent.io/blog/building-a-microservices-ecosystem-with-kafka-streams-and-ksql/)

**참고:** [**What Is Apache Kafka: Everything You Need To Know About**](https://www.appventurez.com/blog/everything-you-need-to-know-about-apache-kafka)

**참고:** [**무료도서 Concepts and Patterns for Streaming Services with Apache Kafka**](https://www.dbooks.org/designing-event-driven-systems-1492038253/#google\_vignette)
