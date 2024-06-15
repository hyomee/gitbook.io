# Basic Kafka Stram

## 1. 의존성

```xml
<properties>
    <java.version>21</java.version>
    <kafka.version>3.7.0</kafka.version>
</properties>

<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>${kafka.version}</version>
</dependency>
```

## 2. 카프카 구성

```
// 주키퍼 시작
$ bin/zookeeper-server-start.sh config/zookeeper.properties

// 카프카 시작 
$ bin/kafka-server-start.sh config/server.properties

$ bin/kafka-topics.sh --describe --topic iabacus-test --bootstrap-server localhost:9092

$ bin/kafka-topics.sh --list --bootstrap-server localhost:9092 
```



## 3. 예제

### 3-1. Topic To Topic&#x20;

프로듀서에서 발행된 메시지를 Topic(source-topic)에서 받아서 Topic(target-topic)으로 전달하는 기본적인 응용프로그램 입니다.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Topic To Topic</p></figcaption></figure>

#### 3-1-1. Topic 생성: source-topic, target-topic 

```
$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1  --partitions 1 --topic source-topic
$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1
```

\
\- 토픽 생성&#x20;

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

```
@/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
source-topic
target-topic
```

#### 3-1-2. 자바 코드

```java
import kr.co.abacus.jmsbroker.kafka.KafkaProperties;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.Topology;
import org.apache.kafka.streams.kstream.KStream;

import java.util.Properties;

public class BasicStreamsApp {

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "simple-streams-app");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, KafkaProperties.KARFA_SERVER_IP);
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());


        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, String> source = builder.stream("source-topic");
        source.to("target-topic");

        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();

        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```

#### 3-1-3. 결과

**Kafka Console Producer 에서 메시지 전송:**

```sh
$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic source-topic
>
```

\
**Kafka Console Consumer 에서 메시지 받음(**source-topic)

```sh
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092  --topic source-topic
```

**Kafka Console Consumer 에서 메시지 받음(**target-topic)

```sh
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092  --topic target-topic
```

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>
