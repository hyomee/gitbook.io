# Message Service

메시지 서비스 (Message Service)는 애플리케이션 간 메시지를 교환하기 위한 통신 메커니즘입니다. 이는 분산 시스템에서 데이터를 비동기적으로 전달하고 처리하는 데 사용되며  메시지 서비스는 다음과 같은 몇 가지 주요 특징으로 가지고 있습니다.

<figure><img src="../.gitbook/assets/image (300).png" alt="" width="563"><figcaption><p>Message Service</p></figcaption></figure>

* **비동기 통신:** 애플리케이션 간의 느슨한 결합으로 송신자와 수신자 간의 직접적인 연결없이 메세지를 교환할 수 있습니다.
* **신뢰성:** 안정적인 데이터 통신을 구현할 수 있도록 메세지 전달을 보장하며 메세지가 손실되지 않도록 합니다.
* **메세지 큐:** 메세지의 순서 보장하거나 작업을 지연시킬 수 있도록 메세지 큐를 제공하여 메세지를 저장하고 처리 할 수 있습니다.
* **토픽 기반 메시징(토픽/구독)**: 다중 수신자에게 메세지를 전달하기 위해 토픽을 사용하여 메시지를 발행하고 구독할 수 있습니다

JMS를 사용하면 엔터프라이즈 애플리케이션에서 효율적인 메시징 시스템을 구축할 수 있습니다.

## 1. JMS(Java Message Service)

**JMS(Java Message Service) API**는 Java EE 및 Java SE 환경 모두에서 Java 프로그램에서 엔터프라이즈 메시징 시스템에 액세스하기 위한 Java API로 Java 프로그램이 엔터프라이즈 메시징 시스템의 메시지를 작성, 전송, 수신 및 읽을 수 있는 일반적인 방법을 제공하고 있으며[ JSR 914로 개발된 명세서](https://jcp.org/en/jsr/detail?id=343)에 의해 정의됩니다

* &#x20;JMS 프로바이더:  RabbitMQ, ActiveMQ 등

## 2. Spring JMS

**Spring JMS**는Java Message Service (JMS)를 사용하는 애플리케이션에서 메시징을 간편하게 구현할 수 있도록 지원하는 Spring 프레임워크의 통합 기능으로 JmsTemplate을 활용하여 메시지를 발행하고 구독하는 방법을 간단하게 구현할 수 있도록 지원합니다.&#x20;

* &#x20;동기 : JmsTemplate을 사용하여 메세지 발행을 큐와 토픽을 통해 쉽게 보내고 수신(소비)자에서 메세지를 받을 수 있습니다.
*   비동기: MessageListenerContainer를 사용하여 비동기 방식으로 큐나 토픽의 메세지에 반응하는 간단한 자바 객체인 메세지 구동(Message-Drive) POJO(MDP) 를 사용하여 메세지를 받는데 사용합니다.\


    <figure><img src="../.gitbook/assets/image (302).png" alt=""><figcaption></figcaption></figure>

<mark style="color:orange;">**JMS는 표준화된 비동기식 메시징을 위한 API이고, Spring JMS는 스프링에서 JMS를 지원하는 컴포넌트**</mark>

<figure><img src="../.gitbook/assets/image (303).png" alt="" width="563"><figcaption><p>JMS와 Spring JMS 차이점 </p></figcaption></figure>

## 3. JMS 프로바이더

### 3-1. **RabbitMQ**

RabbitMQ는 AMQP (Advanced Message Queuing Protocol)를 구현한 오픈소스 메시지 브로커로 산자(Producer)가 메시지를 보내고 소비자(Consumer)에게 전달해주는 역할을 하며 다음과 같은 상활에 사용됩니다.

* **요청을 많은 사용자에게 전달할 때**: RabbitMQ는 많은 사용자에게 동시에 메시지를 전달하는데 효과적입니다.
* **요청에 대한 처리 시간이 길 때**: 메시지 브로커를 통해 요청을 처리하는 동안 다른 작업을 수행할 수 있습니다.
* **많은 작업이 요청되어 처리를 해야할 때**: RabbitMQ는 요청을 다른 API에게 위임하고 빠른 응답을 할 때 유용합니다.
* **시스템 간 통신**: RabbitMQ는 시스템 간 메시지를 전달해주는데 사용됩니다.

### 3-2. **Kafka**

실시간으로 기록 스트림을 게시, 구독, 저장 및 처리할 수 있는 분산형 데이터 스트리밍 플랫폼으로 여러 소스에서 데이터 스트림을 처리하고 여러 사용자에게 전달하도록 설계되었으며 다음과 같은 상활에 사용됩니다.

* **실시간 스트리밍 데이터 파이프라인**: 데이터를 안정적으로 처리하고 한 시스템에서 다른 시스템으로 이동합니다.
* **실시간 스트리밍 애플리케이션**: 데이터 스트림을 소비하는 애플리케이션입니다.
* &#x20;IoT, 전자상거래, IT 운영 등 다양한 활용 분야에서 활용됩니다

## 4. **RabbitMQ와 Kafka 차이점**

두 시스템은 서로 다른 사용 사례를 위해 설계되었으며, 메시징 처리 방식도 다릅니다. Kafka는 대규모 이벤트 스트리밍에 적합하고, RabbitMQ는 빠른 메시지 발행 및 삭제를 위해 사용됩니다

**Kafka**와 **RabbitMQ**는 모두 메시지 대기열 시스템이지만, 각각 다른 강점과 약점을 가지고 있습니다.

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (2) (1).png" alt="" width="563"><figcaption><p>참고: <a href="https://www.confluent.io/learn/rabbitmq-vs-apache-kafka/">https://www.confluent.io/learn/rabbitmq-vs-apache-kafka/</a></p></figcaption></figure>

### 1. **RabbitMQ**

* **유연성**과 **사용 편의성**을 제공하며 다양한 메시징 패턴으로 작업 큐 관리, 마이크로서비스 간 통신, 워크플로우 조정과 같은 시나리오에서 잘 작동합니다
* **고급 라우팅 기능**을 지원하여 복잡한 메시지 라우팅을 처리할 수 있습니다. 즉 다양한 프로토콜 지원, 유연한 메시징 패턴, 복잡한 라우팅 로직을 지원합니다.
* 메시지를 빠르게 발행하고 삭제하는 전통적인 메시징 시스템입니다. 즉 <mark style="color:purple;">**복잡한 브로커, 간단한 컨슈머 접근 방식을 채택**</mark>합니다.
* **메시지 지속성**과 **보장된 전달**이 필요한 응용 프로그램에 적합합니다

### 2. **Kafka**

* **분산 로그**로 설계되어 있어 높은 메시지 처리량, 내결함성 및 확장성을 제공합니다.
* **이벤트 스트리밍**에 특화되어 있으며 대용량 데이터의 고속처리와실시간 처리를에 필요로 하는 시나리오에 뛰어납니다.
* **복잡성**이 있을 수 있으며, 복잡한 메시지 라우팅을 지원하지 않습니다. 즉 <mark style="color:purple;">**단순한 브로커, 복잡한 컨슈머 모델**</mark>을 따릅니다.
* 정확한 한 번만 전달(Exactly-once semantics)을 지원하여 데이터 무결성이 중요한 경우에 적합합니다

## 6. 학습 자료

* **Kafka 기본 자료**

{% file src="../.gitbook/assets/KAFKA 기본 자료_20201012_v0.1.pdf" %}

* RabbitMQ: [https://www.rabbitmq.com/](https://www.rabbitmq.com/)



*   **참고 사이트:**

    * 아파치 카프가 : [https://kafka.apache.org/quickstart](https://kafka.apache.org/quickstart)
    * 아파치 카프카 스트림 : [https://kafka.apache.org/22/documentation/streams/quickstart](https://kafka.apache.org/22/documentation/streams/quickstart)
    * 주키퍼 : [https://zookeeper.apache.org/doc/current/zookeeperOver.html](https://zookeeper.apache.org/doc/current/zookeeperOver.html)
    * 아파치 카프가 오픈 소스 Confluent: https://www.confluent.io/download
    * 스프링 카프카 Reference:  [https://docs.spring.io/spring-kafka/reference/](https://docs.spring.io/spring-kafka/reference/)
    * 스프링 카프카 API: [https://docs.spring.io/spring-kafka/api/overview-summary.html](https://docs.spring.io/spring-kafka/api/overview-summary.html)
    * Intro to Apache Kafka with Spring: [https://www.baeldung.com/spring-kafka](https://www.baeldung.com/spring-kafka)\
      [https://github.com/eugenp/tutorials/blob/master/spring-kafka/src/main/java/com/baeldung/spring/kafka/KafkaApplication.java](https://github.com/eugenp/tutorials/blob/master/spring-kafka/src/main/java/com/baeldung/spring/kafka/KafkaApplication.java)
    * Spring Boot와 함께 Kafka 사용 : \
      [https://reflectoring.io/spring-boot-kafka/\
      https://github.com/thombergs/code-examples/tree/master/spring-boot/spring-boot-kafka](https://reflectoring.io/spring-boot-kafka/)
    * Spring for Apache Kafka 심층 분석 :\
      [https://www.confluent.io/blog/spring-for-apache-kafka-deep-dive-part-1-error-handling-message-conversion-transaction-support/](https://reflectoring.io/spring-boot-kafka/)


* **참고 도서**:
  * 실천 아파치 카프카 - 한빛미디어
  * 아파치 카프카로 데이터 스트리밍 애플리케이션 제작 - 에이콘출판사
