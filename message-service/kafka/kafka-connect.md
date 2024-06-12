# Kafka-Connect

Kafka-Connect는 Apache Kafka의 일부로 데이터 시스템과 Kafka사이에서 데이터를 스트리밍하는 통합 프레임워크로 데이터를 Kafka로부터 가져오거나 Kafka로 보내는 커넥터를 정의하는 공통 프레임워크를 제공하며, 이를 통해 데이터를 실시간으로 스트리밍하고, 전체 데이터베이스를 적재하거나, 애플리케이션 서버에서 메트릭을 수집하여 낮은 지연 시간으로 스트림 처리를 할 수 있게 합니다.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Kafka-Connect의 주요 이점은 다음과 같습니다:

* **데이터 중심 파이프라인**: Kafka를 사용하여 의미 있는 데이터 추상화를 통해 데이터를 끌어오거나 밀어넣습니다.
* **유연성 및 확장성**: 단일 노드(독립 실행형) 또는 전체 조직의 서비스(분산)로 스트리밍 및 배치 지향 시스템에서 실행됩니다.
* **재사용성 및 확장성**: 기존 커넥터를 활용하거나 필요에 맞게 확장하여 생산까지의 시간을 단축합니다.

## 1.   Kafka-Connect 사용처

* **JDBC Source and Sink**: 관계형 데이터베이스에서 데이터를 Kafka 토픽으로 가져오거나 Kafka에서 데이터베이스로 내보냅니다.
* **JMS Source**: JMS 호환 브로커에서 Kafka로 메시지를 이동합니다.
* **Elasticsearch Service Sink**: Kafka에서 Elasticsearch로 데이터를 전송합니다.
* **Amazon S3 Sink**: Kafka 토픽에서 S3 객체로 데이터를 내보냅니다.
* **HDFS 2 Sink**: Kafka 데이터를 HDFS 파일로 내보냅니다(쿼리를 위해 Hive와 통합).
* **Replicator**: 한 Kafka 클러스터에서 다른 클러스터로 토픽을 복제합니다.

## 2.  Kafka-Connect 사용 방법

사용하는 커넥트가 없으면 직접 커넥터를 개발하는 방법과 오픈 소스 및 상용으로 공개된 플러그인을 설치 하여 사용하는 방법이 있습니다.

* **직접 커넥트 개발**: [Connector Developer Guide](https://docs.confluent.io/platform/current/connect/devguide.html)를 참고하여 만들 수 있습니다.
* **플러인 사용**: 대표적인 Kafka Connect로 자세한 정보는 아래 링크를 참고하면 됩니다.
  * **Confluent Kafka Connect:** [**https://docs.confluent.io/platform/current/connect/index.html**](https://docs.confluent.io/platform/current/connect/index.html)
  * **Debezium Kafka Connect:** [**https://debezium.io/documentation/reference/2.6/tutorial.html**](https://debezium.io/documentation/reference/2.6/tutorial.html)

참고: [https://kafka.apache.org/documentation/#connectconfigs](https://kafka.apache.org/documentation/#connectconfigs)

참고: [https://debezium.io/](https://debezium.io/)

참고: [https://www.confluent.io/ko-kr/blog/kafka-connect-tutorial/](https://www.confluent.io/ko-kr/blog/kafka-connect-tutorial/)

