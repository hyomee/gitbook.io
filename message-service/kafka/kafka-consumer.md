# Kafka-Consumer

Kafka 브로커에서 메시지를,읽어와서  소비하는 역할을 하는구것으로  지정돤 토픽의 데이터(메시지)를 레코드 단위로 가져와 소비합니다. 메시지를 읽을 때, 부하 분산과 가용성 및 처리량 증가에 도움이 되는 파티션을 사용하고, 오프셋을 통해 메시지를 추적합니다. 다음은 Kafka Consumer에 관한 주요 사항입니다.

<figure><img src="../../.gitbook/assets/image (344).png" alt=""><figcaption></figcaption></figure>

1. **데이터 가져오기(Kafka Consumer)**:
   * Consumer는 브로커에게 가져올 파티션을 지정하는 “fetch” 요청을 보냅니다.
   * 각 요청은 로그 오프셋을 지정하며 해당 오프셋 위치부터 로그의 일부를 수신합니다.
   * Consumer는 위치를 제어하고 필요한 경우 데이터를 다시 소비할 수 있습니다.
   * <mark style="color:purple;">할당된 파티션에서 데이터를 읽어오는 것으로 토픽 파티션에서 데이터를 읽어오는 등록 절차 입니다.</mark>
2. **토픽(Topic) 및 파티션(Partition):**
   * **Topic**: 메세지 범주&#x20;
   * **Partition:** Topic에서 가장 작은 저장 단위로 Topic에 대한 파티션 수를 구성할 수 있습니다.
3. **Consumer 그룹**:
   * Consumer는 **데이터를 소비하기 위해 그룹을 형성**할 수 있습니다.
   * Consumer 그룹은 소비자 구성에서 group.id 속성을 설정하여 정의됩니다.
   * &#x20;group.id가 지정되지 않으면 무작위로 생성됩니다.
   * 모든 토픽의 파티션은 그룹 내의 소비자에게 분배됩니다.
   * 새 멤버가 가입하거나 나가면 파티션은 비례적으로 재할당됩니다.
   * 한 브로커가 그룹 코디네이터 역할을 하며 멤버와 할당을 관리합니다.
   * 코디네이터는 내부 오프셋 토픽(\_\_consumer\_offsets)의 리더 중에서 선택됩니다.
   * Consumer 그룹은 동일한 토픽에서 메시지를 병렬로 처리할 수 있도록 합니다.
4. **Heartbeat:**
   * Kafka 그룹 관리 기능을 사용할 때 컨슈머 코디네이터에게 주기적으로 보내는 신호입니다.
   * 이 신호는 다음 목적으로 사용됩니다:
     1. **세션 활성 유지**:
        * Heartbeat은 컨슈머 세션이 활성 상태인지 확인합니다.
        * 컨슈머가 살아있고 세션이 유지되는지 확인하며, 세션 타임아웃 시간보다 낮은 간격으로 전송됩니다.
     2. **리밸런싱 지원**:
        * 새로운 컨슈머가 그룹에 가입하거나 나가면 리밸런싱이 발생합니다.
        * Heartbeat은 리밸런싱을 원활하게 지원합니다.
        * 일반적으로 Heartbeat 간격은 세션 타임아웃 시간의 1/3 이하로 설정됩니다.
5. **오프셋 관리**:
   * 할당을 받은 후 Consumer는 각 파티션에 대한 초기 위치를 결정합니다.
   * Consumer는 로그 내에서 위치를 조정할 수 있습니다.
   * Kafka는 동일한 토픽에 여러 Consumer 애플리케이션이 동시에 구독할 수 있도록 합니다.
6. **Rewind (되감기):**
   * 컨슈머가 특정 파티션에서 이전 오프셋으로 되돌아가는 것을 말합니다.
   * 이는 특정 메시지를 다시 처리하거나 오류를 수정하는 데 유용합니다.
   * 프로그래밍 방식으로 컨슈머 오프셋을 조작하려면 ConsumerSeekAware 인터페이스를 구현하면 됩니다. 예를 들어, Spring Kafka에서는 ConsumerSeekAware 를 사용하여 컨슈머 오프셋을 재설정할 수 있습니다.
7. **Skip (건너뛰기):**
   * 컨슈머가 특정 메시지를 건너뛰고 다음 메시지로 진행하는 것을 말합니다. 예를 들어, 오류가 있는 메시지를 건너뛰고 처리를 계속할 수 있습니다.
   * 이는 ConsumerRecordFilter를 사용하여 구현할 수 있습니다.
   * Spring Kafka에서는 SeekToCurrentErrorHandler를 사용하여 오류 메시지를 건너뛸 수 있습니다.
8. **역직렬화:**&#x20;
   * 카프카 프로듀서에서 카프카에 메시지를 보내기 전에 바이트 배열로 객체를 직렬화하여 보내면 컨슈머에서 바이트 배열을 역직렬화하여야 합니다.

Kafka Consumer는 확장 가능하고 내결함성 있는 데이터 처리 파이프라인 구축에 중요한 역할을 합니다.

## 1. 간단한 Use Case를 통해서 보는 Consumer 그룹

<figure><img src="../../.gitbook/assets/image (345).png" alt=""><figcaption></figcaption></figure>

### 1-1. 일반적인 구성&#x20;

하나의 Consumer 에서 하나의 Consumer Grop에서 받는 경우는 group\_id에는 topic 명으로 작성하여 하나의 그룹으로 메시지를 수신 받게 합니다.

```java
public static Properties getConsumerBaseProperties() {
    Properties configs = new Properties();
    // kafka server host 및 port
    configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KARFA_SERVER_IP);
    // session 설정
    configs.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "10000");

    // 그룹설정
    configs.put(ConsumerConfig.GROUP_ID_CONFIG, KARFA_TOPIC_P5);

    // key deserializer
    configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

    // value deserializer
    configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

    return configs;
}
```

* Consumer.subscribe 메서드를 사용하여 Topic을 지정 합니다.

```java
Properties configs = KafkaProperties.getConsumerBaseProperties();

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);    // consumer 생성
consumer.subscribe(Collections.singletonList(KafkaProperties.KARFA_TOPIC_P5)); // topic 설정
```

### 1-2. 파티션을 기준으로 컨슈머 그룹 지정&#x20;

* 1-1  그룹 설정에서 컨슈머 그룹을 지정 합니다. ( 2 Consumer - 2 Consumer group )

```java
public static Properties getConsumerProperties() {
    Properties configs = new Properties();
    // kafka server host 및 port
    configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KARFA_SERVER_IP);
    // session 설정
    configs.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "10000");

    // 컨슈머 그룹 설정 
    configs.put(ConsumerConfig.GROUP_ID_CONFIG, KARFA_TOPIC_GROUP_ID_00);

    // key deserializer
    configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

    // value deserializer
    configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

    return configs;
}

public static Properties getConsumerProperties01() {
    Properties configs = new Properties();
    // kafka server host 및 port
    configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KARFA_SERVER_IP);
    // session 설정
    configs.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "10000");
    
    // 컨슈머 그룹 설정 
    configs.put(ConsumerConfig.GROUP_ID_CONFIG, KARFA_TOPIC_GROUP_ID_01);

    // key deserializer
    configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

    // value deserializer
    configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

    return configs;
}
```

* Consumer 실행 할 Main Class를 각각 만듭니다.

{% tabs %}
{% tab title="ConsumerMain " %}


```java
public class ConsumerMain {
    public static void main(String[] args) {

        System.out.println("Consumer Start ....");
        Properties configs = KafkaProperties.getConsumerProperties();

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);    // consumer 생성

        TopicPartition partition0 = 
            new TopicPartition(KafkaProperties.KARFA_TOPIC_P5, 0);
            
        TopicPartition partition1 = 
            new TopicPartition(KafkaProperties.KARFA_TOPIC_P5, 1);
        
        TopicPartition partition3 = 
            new TopicPartition(KafkaProperties.KARFA_TOPIC_P5, 3);

        consumer.assign(Arrays.asList(partition0, partition1, partition3));
        
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(500);
            for (ConsumerRecord<String, String> record : records) {
                String input = record.topic();
                if (KafkaProperties.KARFA_TOPIC.equals(input)) {
                    printRecord(record);
                } if (KafkaProperties.KARFA_TOPIC_P5.equals(input)) {
                    printRecord(record);
                } else {
                    throw new IllegalStateException("get message on topic " + record.topic());
                }
            }
        }
    }

    private static void printRecord(ConsumerRecord<String, String> record) {

        System.out.println("topic = " + record.topic() +
                            ":: partition = " + record.partition()  +
                            ":: key = " + record.key() +
                            ":: value = " + record.value());



        record.headers().forEach(header -> {
            System.out.println(header.key() + ": " + new String(header.value()));
        });
    }
}
```
{% endtab %}

{% tab title="ConsumerMain01 " %}
```java
public class ConsumerMain01 {
    public static void main(String[] args) {

        System.out.println("Consumer Start ....");
        Properties configs = KafkaProperties.getConsumerProperties01();

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);    // consumer 생성

        TopicPartition partition2 = 
            new TopicPartition(KafkaProperties.KARFA_TOPIC_P5, 2);
            
        TopicPartition partition3 = 
            new TopicPartition(KafkaProperties.KARFA_TOPIC_P5, 3);

        consumer.assign(Arrays.asList(partition2, partition3));
        
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(500);
            for (ConsumerRecord<String, String> record : records) {
                String input = record.topic();
                if (KafkaProperties.KARFA_TOPIC.equals(input)) {
                    printRecord(record);
                } if (KafkaProperties.KARFA_TOPIC_P5.equals(input)) {
                    printRecord(record);
                } else {
                    throw new IllegalStateException("get message on topic " + record.topic());
                }
            }
        }
    }

    private static void printRecord(ConsumerRecord<String, String> record) {

        System.out.println("topic = " + record.topic() +
                            ":: partition = " + record.partition()  +
                            ":: key = " + record.key() +
                            ":: value = " + record.value());



        record.headers().forEach(header -> {
            System.out.println(header.key() + ": " + new String(header.value()));
        });
    }
}
```
{% endtab %}
{% endtabs %}





참고: [https://docs.confluent.io/platform/current/clients/consumer.html](https://docs.confluent.io/platform/current/clients/consumer.html)

참고: [https://www.confluent.io/blog/apache-kafka-data-access-semantics-consumers-and-membership/?session\_ref=https://harunpeksen.medium.com/how-apache-kafka-consumer-works-6cee4eb83147&\_ga=2.87875742.724744377.1718006844-1261962290.1705303831&\_gl=1\*17ejtkh\*\_gcl\_au\*MTc1ODYzODkyNi4xNzE2NTMwMjU2\*\_ga\*MTI2MTk2MjI5MC4xNzA1MzAzODMx\*\_ga\_D2D3EGKSGD\*MTcxODAwNjg0My43LjEuMTcxODAwNzEyNi40NC4wLjA.](https://www.confluent.io/blog/apache-kafka-data-access-semantics-consumers-and-membership/?session\_ref=https://harunpeksen.medium.com/how-apache-kafka-consumer-works-6cee4eb83147&\_ga=2.87875742.724744377.1718006844-1261962290.1705303831&\_gl=1\*17ejtkh\*\_gcl\_au\*MTc1ODYzODkyNi4xNzE2NTMwMjU2\*\_ga\*MTI2MTk2MjI5MC4xNzA1MzAzODMx\*\_ga\_D2D3EGKSGD\*MTcxODAwNjg0My43LjEuMTcxODAwNzEyNi40NC4wLjA.)
