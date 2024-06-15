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

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Topic To Topic</p></figcaption></figure>

#### 3-1-1. Topic 생성: source-topic, target-topic

```sh
$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1  --partitions 1 --topic source-topic
$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1  --partitions 1 --topic target-topic
```

\
\- 토픽 생성&#x20;

<figure><img src="../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### 3-2. Stateful Operations (상태 저장)

간단한 단어 수 계산을 하는 프로그램을 통해서 Kafka Stream의 상태저장(Stateful Operations)에 KTable을 사용합니다.

#### 3-2-1. Topic 생성: line-topic, wordcounts-topic

```sh
$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1  --partitions 1 --topic line-topic
$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1  --partitions 1 --topic wordcounts-topic
```

#### 3-2-2. 프로그램 코드

```java
import kr.co.abacus.jmsbroker.kafka.KafkaProperties;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KTable;
import org.apache.kafka.streams.kstream.Produced;

import java.util.Arrays;
import java.util.Properties;

public class StatefulStreamsApp {

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "stateful-streams-app");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, KafkaProperties.KARFA_SERVER_IP);
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

    
        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, String> textLines = builder.stream("line-topic");

        KTable<String, Long> wordCounts = textLines
                .flatMapValues(textLine -> Arrays.asList(textLine.toLowerCase().split("\\s+")))
                .groupBy((key, word) -> word)
                .count();

        wordCounts.toStream().foreach((word, count) -> System.out.println("word: " + word + " -> " + count));


        wordCounts.toStream().to("wordcounts-topic", Produced.with(Serdes.String(), Serdes.Long()));
        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();

        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```

### 3-2. Join Operations

5분 동안 두 스트림 간의 간단한 내부 조인

```java
KStream<String, String> leftSource = builder.stream("left-topic");
KStream<String, String> rightSource = builder.stream("right-topic");

KStream<String, String> joined = leftSource.join(rightSource,
    (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue,
    JoinWindows.of(Duration.ofMinutes(5)),
    StreamJoined.with(
        Serdes.String(),
        Serdes.String(),
        Serdes.String())
);
joined.to("joined-topic");
```

### 3-3. Advanced Processing

스트림 처리를 완전히 제어할 수 있는 사용자 지정 프로세서의 사용 예제

```java
builder.stream("input-topic").process(() -> new Processor<String, String>() {
    private ProcessorContext context;

    @Override
    public void init(ProcessorContext context) {
        this.context = context;
    }

    @Override
    public void process(String key, String value) {
        // Implement custom processing logic here
        context.forward(key, new CustomValue(value));
        context.commit();
    }

    @Override
    public void close() {
        // Implement any cleanup code here
    }
}, "processor-node");
```



## 4. 오류 처리

Kafka Streams는 처리 중에 예외를 처리하는 방법으로 다음과 같은 구성을 사용하면 deserialization 및 프로덕션 예외를 로깅 및 계속 관리하거나 응용 프로그램을 종료할 수도 있는 기본 처리를 통해 관리할 수 있습니다.

```java
props.put(StreamsConfig.DEFAULT_DESERIALIZATION_EXCEPTION_HANDLER_CLASS_CONFIG, LogAndContinueExceptionHandler.class);
props.put(StreamsConfig.DEFAULT_PRODUCTION_EXCEPTION_HANDLER_CLASS_CONFIG, DefaultProductionExceptionHandler.class);
```
