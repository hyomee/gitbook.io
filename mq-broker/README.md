# MQ/Broker

**Kafka**와 **RabbitMQ**는 모두 메시지 대기열 시스템이지만, 각각 다른 강점과 약점을 가지고 있습니다.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>참고: <a href="https://www.confluent.io/learn/rabbitmq-vs-apache-kafka/">https://www.confluent.io/learn/rabbitmq-vs-apache-kafka/</a></p></figcaption></figure>

1. **Kafka**:
   * **분산 로그**로 설계되어 있어 높은 메시지 처리량, 내결함성 및 확장성을 제공합니다.
   * **이벤트 스트리밍**에 특화되어 있으며 대용량 데이터의 실시간 처리를 지원합니다.
   * **복잡성**이 있을 수 있으며, 복잡한 메시지 라우팅을 지원하지 않습니다.
2. **RabbitMQ**:
   * **유연성**과 **사용 편의성**을 제공하며 다양한 메시징 패턴에 적합합니다.
   * **고급 라우팅 기능**을 지원하여 복잡한 메시지 라우팅을 처리할 수 있습니다.
   * 메시지를 빠르게 발행하고 삭제하는 전통적인 메시징 시스템입니다

두 시스템은 서로 다른 사용 사례를 위해 설계되었으며, 메시징 처리 방식도 다릅니다. Kafka는 대규모 이벤트 스트리밍에 적합하고, RabbitMQ는 빠른 메시지 발행 및 삭제를 위해 사용됩니다



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
