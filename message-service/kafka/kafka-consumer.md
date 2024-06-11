# Kafka-Consumer

Kafka 브로커에서 메시지를,읽어와서  소비하는 역할을 하는구것으로  지정돤 토픽의 데이터(메시지)를 레코드 단위로 가져와 소비합니다. 메시지를 읽을 때, 부하 분산과 가용성 및 처리량 증가에 도움이 되는 파티션을 사용하고, 오프셋을 통해 메시지를 추적합니다. 다음은 Kafka Consumer에 관한 주요 사항입니다.

* 토픽 구독 (Topic Subscription)
* 컨슈머 그룹 (Consumer Group)
* 파티션 할당 (Partition Assignment)
* 오프셋 관리 (Offset Management)
* 커밋 (Commit)

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
   * 컨슈머 그룹은 데이터 소비를 위해 형성될 수 있으며, 이를 통해 토픽에서 읽는 데이터의 양을 늘릴 수 있습니다. 이 방식을 통해, 토픽의 모든 파티션은 그룹 내의 컨슈머들에게 할당됩니다.
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
5. **역직렬화:**&#x20;
   * 카프카 프로듀서에서 카프카에 메시지를 보내기 전에 바이트 배열로 객체를 직렬화하여 보내면 컨슈머에서 바이트 배열을 역직렬화하여야 합니다.

Kafka Consumer는 확장 가능하고 내결함성 있는 데이터 처리 파이프라인 구축에 중요한 역할을 합니다.

## 1. Consumer 그룹

* 동일한  컨슈머 그룹에 속한 여러 개의 컨슈머들이 동일한 토픽을 구독할 경우, 각각의 컨슈머는 해당 토픽에서 서로 다른 파티션의 메세지를 받습니다.
* Consumer 그룹은 group.id 속성으로 설정되고, group.id가 지정되지 않은 경우에는 임의로 생성됩니다.

### **1-1. 리밸런스(rebalance)**

**리밸런스(rebalance)**은 토픽에 새로운 파티션이 추가될 때, 이미 할당된 파티션을 한 컨슈머에서 다른 컨슈머로 재할당하는 과정입니다. 이는 고가용성(High availability)과 확장성(Scalability)에는 필수적인 기능이지만, 문제가 없는 경우에는 가능한 피하는 것이 바람직합니다. 리밸런싱 중에는 컨슈머 그룹의 컨슈머들이 토픽의 데이터를 읽을 수 없기 때문 입니다.

다음과 같은 유형이 있습니다.

* **Round-robin (라운드 로빈)**: 기본적인 리밸런싱 전략으로, 파티션을 순차적으로 컨슈머에 할당합니다. 각 컨슈머가 하나씩 파티션을 가져갑니다
* **Eager rebalance(조급한)**: 모든 컨슈머의 작업을 중지시키고 다시 할당받는 전략(<mark style="color:purple;">2.4 부터 기본</mark>)
* **Cooperative rebalance(협력적)**: 리밸런싱이 일어나더라도 가능한 같은 파티션을 유지하기 위해, 한 컨슈머에게 할당된 파티션을 다른 컨슈머에게 재할당하는 전략을 사용합니다. 이 전략에 따라, 재할당되지 않은 파티션에서 데이터를 처리하던 컨슈머는 계속해서 작업을 수행합니다. (<mark style="color:purple;">3.1 부터 기본</mark>).
* **Range (범위)**: 파티션의 범위를 기준으로 할당하는 전략입니다. 예를 들어, 파티션 1부터 10까지는 컨슈머 A에게, 11부터 20까지는 컨슈머 B에게 할당됩니다.
* 참고: [https://cwiki.apache.org/confluence/display/KAFKA/KIP-345%3A+Introduce+static+membership+protocol+to+reduce+consumer+rebalances](https://cwiki.apache.org/confluence/display/KAFKA/KIP-345%3A+Introduce+static+membership+protocol+to+reduce+consumer+rebalances)

### **1-2.그룹 코디네이터(Group Coordinator)**

그룹 코디네이터는 카프카 브로커(컨슈머 그룹 별 틀릴 수 있음) 중 하나가 맡는 역할로, 특정 컨슈머 그룹을 담당합니다. 컨슈머들은 하트비트 메시지를 보내어 자신이 속한 컨슈머 그룹 내에서의 멤버십을 유지하고, 할당된 파티션을 관리합니다. 멤버십과 파티션 할당을 책임지는 것입니다. 컨슈머 그룹에 가장 먼저 참여한 컨슈머가 \_\_consumer\_offsets 내부 오프셋 토픽의 리더로 선출됩니다.

### &#x20;**1-3.** 정적 그룹 멤버십(Static Group Membership)

정적 그룹 멤버십(Static Group Membership)은 '고정된 컨슈머 그룹'은 구성원이 변경되지 않고, 처음 설정된 멤버들로만 유지되는 상태를 말합니다. 이는 컨슈머 그룹에 멤버가 동적으로 추가되거나 제거되지 않음을 의미합니다.

## 2. 커밋(Commit)과 오프셋(Offset)

컨슈머는 커밋(Commit)을 통해 자신이 레코드를 어디까지 가져갔는지 카프카 브로커에 기록한다. 이를 기록하기 위해 오프셋이라는 개념이 사용된다

### **2-1. Consumer의 오프셋 관리**

* 할당을 받은 후, Consumer는 각 파티션에 대한 초기 위치를 결정하며, 오프셋은 0 이상의 숫자로 구성됩니다. 이는 직접 지정할 수 없으며, 브로커에 저장된 마지막 레코드의 오프셋에 1을 더한 값으로 설정됩니다.
* 오프셋을 사용하여 각각의 카프카 컨슈머가 파티션 데이터를 어디까지 읽었는지 결정할 수 있습니다.
* 오프셋은 컨슈머 그룹별로 관리되고, 해당 컨슈머 그룹의 오프셋은 카프카 브로커 내의 \_\_consumer\_offsets라는 특별한 토픽에 저장됩니다. 이러한 방식으로, 다양한 목적을 가진 여러 컨슈머 그룹들이 동일한 토픽의 레코드를 여러 번 읽을 수 있습니다..
* Kafka는 동일한 토픽에 여러 Consumer 애플리케이션이 동시에 구독할 수 있도록 합니다.
* Consumer는 로그에서 위치를 조절할 수 있습니다.
  * **Rewind (되감기):**
    * 컨슈머가 특정 파티션에서 이전 오프셋으로 되돌아가는 것을 말합니다.
    * 이는 특정 메시지를 다시 처리하거나 오류를 수정하는 데 유용합니다.
    * 프로그래밍 방식으로 컨슈머 오프셋을 조작하려면 ConsumerSeekAware 인터페이스를 구현하면 됩니다. 예를 들어, Spring Kafka에서는 ConsumerSeekAware 를 사용하여 컨슈머 오프셋을 재설정할 수 있습니다.
  * **Skip (건너뛰기):**
    * 컨슈머가 특정 메시지를 건너뛰고 다음 메시지로 진행하는 것을 말합니다. 예를 들어, 오류가 있는 메시지를 건너뛰고 처리를 계속할 수 있습니다.
    * 이는 ConsumerRecordFilter를 사용하여 구현할 수 있습니다.
    * Spring Kafka에서는 SeekToCurrentErrorHandler를 사용하여 오류 메시지를 건너뛸 수 있습니다.

<figure><img src="../../.gitbook/assets/image (347).png" alt=""><figcaption></figcaption></figure>

### 2-2. 초기 오프셋 전략 <a href="#undefined" id="undefined"></a>

카프카의 초기 오프셋 전략은 컨슈머 그룹이 파티션에서 레코드를 읽을 때 오프셋이 지정되지 않았을 경우의 처리 방법을 정하는 것입니다. 이 전략은 'auto.offset.reset' 옵션으로 설정할 수 있으며, 데이터의 순서를 보장하고 컨슈머가 읽은 데이터의 위치를 추적하는 데 사용됩니다. 초기 오프셋 전략에는 세 가지 유형이 있습니다.

1. **latest (가장 최근의 오프셋부터)**:
   * 기본값입니다.
   * 가장 최근의 (오프셋이 가장 높은) 레코드부터 처리합니다.
2. **earliest (가장 오래된 오프셋부터)**:
   * 가장 오래된 (오프셋이 가장 낮은) 레코드부터 처리합니다.
3. **none (커밋 기록이 없는 경우)**:
   * 커밋 기록이 없다면 오류를 반환하고, 커밋 기록이 있다면 마지막 커밋 이후 오프셋부터 읽기 시작합니다..&#x20;

## 2. Kafka Consumer 생성

**Apache Kafka**에서 데이터를 소비하는 역할을 합니다. 이 클라이언트는 Kafka 클러스터에서 레코드를 소비하며, Kafka 브로커의 장애를 투명하게 처리하고, 가져온 토픽 파티션이 클러스터 내에서 이동할 때 자동으로 적응합니다. 간단히 말해, KafkaConsumer는 데이터 스트림을 구독하고 처리하는 애플리케이션으로.KafkaConsumer 생성으로 시작합니다.



```sh
sudo ./bin/kafka-topics.sh --create 
          --bootstrap-server localhost:9092 
          --replication-factor 1  
          --partitions 4 
          --topic topic-04
          
sudo ./bin/kafka-topics.sh --create 
          --bootstrap-server localhost:9092 
          --replication-factor 1  
          --partitions 4 
          --topic topic-partition-04
```

### 2-1. Kafka 속성 정의

Consumer 속성을 설정하기 위한 코드 입니다.

* getProducerProperties: Producer 속성 정의
* setConsumerProperties: Consumer 속성 정의
* getConsumerGroupid: group\_id 설정&#x20;

{% code lineNumbers="true" %}
```java
public class KafkaProperties {
    public static String KARFA_SERVER_IP = "172.24.239.164:9092";
    public static String KARFA_TOPIC_04 = "topic-04";
    public static String KARFA_TOPIC_GROUP_ID  = "topic-04_group";

    public static String KARFA_TOPIC_PARTITION_04 = "topic-partition-04-p5";
    public static String KARFA_TOPIC_GROUP_ID_00 = "iabacus-test_00";
    public static String KARFA_TOPIC_GROUP_ID_01 = "iabacus-test_01";

    public static String CURRENT_TOPIC = KafkaProperties.KARFA_TOPIC_04; 

    public static Properties getProducerProperties() {
        Properties configs = new Properties();
        // kafka server host 및 port
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, KARFA_SERVER_IP);
        // acks 설정
        configs.put(ProducerConfig.ACKS_CONFIG, "1");

        // key Serializer
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG , "org.apache.kafka.common.serialization.StringSerializer");

        // value Serializer
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");

        return configs;
    }

    public static Properties getConsumerGroupid(String groupid) {
        Properties configs = setConsumerProperties();
        // Group Id 설정
        configs.put(ConsumerConfig.GROUP_ID_CONFIG, groupid);
        return configs;
    }


    private static Properties setConsumerProperties() {
        Properties configs = new Properties();
        // kafka server host 및 port
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KARFA_SERVER_IP);
        // session 설정
        configs.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "10000");

        // key deserializer
        configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

        // value deserializer
        configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

        return configs;
    }

    public Callback producerCallback = (RecordMetadata metadata, Exception exception) -> {
            System.out.println("onCompletion :: " + metadata);
            if (exception != null) {
                exception.printStackTrace();
            }
    };
}
```
{% endcode %}

### 2-2. KafkaConsumer 생성&#x20;

```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);
```

* &#x20;Producer 코드

{% code lineNumbers="true" %}
```java
for (int i = 0; i < 4; i++) {
        String v = "hello :: " + i;
        String k = "key:: partition :: 0" + i;
        
        CustomerRecord customerRecord = new CustomerRecord(v, i);
        Gson gson = new Gson();
        String json = gson.toJson(customerRecord);
        
        
        ProducerRecord producerRecord =
                new ProducerRecord<>(KafkaProperties.CURRENT_TOPIC, 
                                     i ,
                                     k , 
                                     json );
                                     
        producerRecord.headers().add("INPUT-CHANNEL", 
                                "IPHONE".getBytes(StandardCharsets.UTF_8));
        
        
        try {
            producer.send(producerRecord, onCompletion );
        } catch (Exception e) {
            e.printStackTrace();
        }
}        
```
{% endcode %}

* 11 Line: i는 Partition 위치 입니다.&#x20;

## 3. 구독

* Consumer 와 Consumer Group 관계

<figure><img src="../../.gitbook/assets/image (345).png" alt=""><figcaption></figcaption></figure>

### 3-1. **subscribe( 1** Consumer - 1 Consumer Group )

**subscribe 메서드**를 사용하여 원하는 토픽을 구독하면, 해당 토픽의 모든 파티션에서 메시지가 지정된 컨슈머에게 전달됩니다. 이 과정에서 컨슈머 그룹은 단 하나입니다.

```java
 consumer.subscribe(Collections.singleton(KafkaProperties.CURRENT_TOPIC)); // topic 설정
```

컨슈머 그룹은 KafkaConsumer 객체 생성시 속성에 선언합니다.

```java
properties.put(ConsumerConfig.GROUP_ID_CONFIG, groupid);
```

* Consumer 속성을 다음과 같이 설정한 컨슈머 코드는 다음과 같습니다.
  * "group.id": topic-04\_group
  * consumer.subscribe(): topic-04&#x20;

{% tabs %}
{% tab title="ConsumerMain " %}
```java
public class ConsumerMain {
    public static void main(String[] args) {

        System.out.println("Consumer Start ....");
        Properties configs = 
            KafkaProperties.getConsumerGroupid(KafkaProperties.KARFA_TOPIC_GROUP_ID);

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);    // consumer 생성
        consumer.subscribe(Collections.singleton(KafkaProperties.CURRENT_TOPIC)); // topic 설정

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(500);
            for (ConsumerRecord<String, String> record : records) {
                printRecord(record);
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

{% tab title="KafkaProperties " %}
{% code lineNumbers="true" %}
```java
public static String KARFA_TOPIC_04 = "topic-04";
public static String KARFA_TOPIC_GROUP_ID  = "topic-04_group";

public static String CURRENT_TOPIC = KafkaProperties.KARFA_TOPIC_04; 
```
{% endcode %}
{% endtab %}
{% endtabs %}

* Consumer 결과: "topic-04\_group" 에 모든 파티션의 메시지를 받을 것을 확인 할 수 있습니다.

```log
[main] INFO org.apache.kafka.clients.consumer.internals.SubscriptionState - [Consumer clientId=consumer-topic-04_group-1, groupId=topic-04_group]
topic = topic-04:: partition = 3:: key = key:: partition :: 03:: value = {"name":"hello :: 3","age":3}
INPUT-CHANNEL: IPHONE
topic = topic-04:: partition = 2:: key = key:: partition :: 02:: value = {"name":"hello :: 2","age":2}
INPUT-CHANNEL: IPHONE
topic = topic-04:: partition = 1:: key = key:: partition :: 01:: value = {"name":"hello :: 1","age":1}
INPUT-CHANNEL: IPHONE
topic = topic-04:: partition = 0:: key = key:: partition :: 00:: value = {"name":"hello :: 0","age":0}
INPUT-CHANNEL: IPHONE
```

### 3-2.   assign (2 Consumer - 2 Consumer Group)

Consumer Group를 파티션별로 받기 위해서는 TopicPartition 객체를 사용해서 파티션을 지정 하고 assign 메서드를 사용하여 Consumer에 할당해야 합니다.

```java
// 토픽에 사용할 파티션 지정 0, 1
TopicPartition partition2 = new TopicPartition(KafkaProperties.CURRENT_TOPIC, 2);
TopicPartition partition4 = new TopicPartition(KafkaProperties.CURRENT_TOPIC, 3);

// 컨슈머에 토픽 할당 
consumer.assign(Arrays.asList(partition2, partition4));
```

* Consumer 속성을 다음과 같이 설정한 컨슈머 코드는 다음과 같습니다.
  * group\_id: topic-partition\_01 설정
    * TopicPartition : partition 0, 1 로 설정
  * group\_id: topic-partition\_02 설정
    * TopicPartition : partition 2, 3 로 설정

{% tabs %}
{% tab title="ConsumerGroupId01Main " %}
```java
public class ConsumerGroupId01Main {
    public static void main(String[] args) {

        System.out.println("Consumer Start ....");
        // 그룹 아이디 설정 "topic-partition_01"
        Properties configs = KafkaProperties.getConsumerGroupid(KafkaProperties.KARFA_TOPIC_GROUP_ID_23);

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);    // consumer 생성

        // 토픽에 사용할 파티션 지정 0, 1
        TopicPartition partition2 = new TopicPartition(KafkaProperties.CURRENT_TOPIC, 2);
        TopicPartition partition4 = new TopicPartition(KafkaProperties.CURRENT_TOPIC, 3);

        // 컨슈머에 토픽 할당 
        consumer.assign(Arrays.asList(partition2, partition4));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(500);
            for (ConsumerRecord<String, String> record : records) {
                printRecord(record);
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

{% tab title="ConsumerGroupId23Main " %}
```java
public class ConsumerGroupId23Main {
    public static void main(String[] args) {

        System.out.println("Consumer Start ....");
        // 그룹 아이디 설정 "topic-partition_23"
        Properties configs = KafkaProperties.getConsumerGroupid(KafkaProperties.KARFA_TOPIC_GROUP_ID_23);

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);    // consumer 생성

        // 토픽에 사용할 파티션 지정 0, 1
        TopicPartition partition2 = new TopicPartition(KafkaProperties.CURRENT_TOPIC, 2);
        TopicPartition partition3 = new TopicPartition(KafkaProperties.CURRENT_TOPIC, 3);

        // 컨슈머에 토픽 할당 
        consumer.assign(Arrays.asList(partition2, partition3));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(500);
            for (ConsumerRecord<String, String> record : records) {
                printRecord(record);
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

{% tab title="속성" %}
```java
public static String KARFA_TOPIC_PARTITION_04 = "topic-partition-04";
public static String KARFA_TOPIC_GROUP_ID_01 = "topic-partition_01";
public static String KARFA_TOPIC_GROUP_ID_23 = "topic-partition_23";

public static String CURRENT_TOPIC = KafkaProperties.KARFA_TOPIC_PARTITION_04;
```
{% endtab %}
{% endtabs %}

* ConsumerGroupId01Main 결과

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

* ConsumerGroupId23Main 결과

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## 4. Polling Loop

이것은 구독한 메시지를 폴링하는 간단한 루프로, 여기서 구독한 메시지들을 처리합니다. 구독한 메시지들은 '레코드'로 불리며, 읽지 않은 모든 메시지들을 포함합니다. 'ConsumerRecords'는 리스트 형태이기 때문에, for 문을 사용하여 각 레코드를 차례대로 처리합니다.

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(500);
    for (ConsumerRecord<String, String> record : records) {
        printRecord(record);
    }
}
           
```

## 5. **오프셋 커밋 유형**

### 5-1. 비명시적 오프셋 커밋 <a href="#undefined" id="undefined"></a>

비명시적 오프셋 커밋은 poll() 메서드가 실행된 이후 일정 시간이 지나면 그 시점까지 읽은 오프셋을 커밋하는 방식입니다.

* `enable.auto.commit:기본값 true` (비명시적 오프셋 커밋을 수행)
* `auto.commit.interval.ms:` poll() 메서드가 실행된 이후 일정 시간 설정
* 주의사항: poll() 호출 이후에 리밸런싱이 발생하거나, 컨슈머가 비정상적으로 종료되었을 때 메시지가 중복처리되거나 유실될 가능성이 있습니다.

### 5-2. 명시적 오프셋 커밋 <a href="#undefined" id="undefined"></a>

명시적 오프셋 커밋은 poll() 메서드가 실행된 이후 commitSync() 메서드를 호출하여 가장 마지막 오프셋을 기준으로 커밋하는 방식입니다.

* `enable.auto.commit: false`&#x20;
* commitSync() 메서드를 호출하여 poll() 메서드를 통해 반환된 레코드의 가장 마지막 오프셋을 기준으로 커밋 합니다.
* 주의사항:  비명시적 방식에 비해 단위 시간당 처리량이 낮습니다.

### 5-3. **commitAsync As** commitAsync

* **commitAsync():** commitAsync() 메서드를 비동기적으로 사용하면 처리량을 증가시킬 수 있으며, 커밋 요청에 대한 응답을 기다리는 동안 데이터 처리가 가능합니다. 그러나 커밋에 실패할 경우 순서를 보장할 수 없고 데이터가 중복 처리될 위험이 있습니다.
  * 블로킹되지 않으며, 콜백을 통해 성공 또는 실패 여부를 처리합니다.
  * 호출 시 현재 스레드가 커밋 결과를 기다리지 않고 다음 작업을 진행합니다.
  *   빠른 처리를 위해 사용하며, 콜백 함수를 통해 커밋 결과를 처리합니다.

      ```java
      while (true) {
          ConsumerRecords<String, String> records = consumer.poll(500);
          for (ConsumerRecord<String, String> record : records) {
              printRecord(record);
          }
          consumer.commitAsync();
      }
      ```
* **commitSync()**: 실패시 성공하거나 재시도할 수 없는 오류가 발생할 때 까지 재시도 합니다.&#x20;
  * 커밋이 성공하거나 복구할 수 없는 오류가 발생할 때까지 블로킹됩니다.
  * 호출 시 현재 스레드가 커밋이 완료될 때까지 대기합니다.
  *   데이터 일관성을 보장하고자 할 때 사용합니다.

      <pre class="language-java"><code class="lang-java"><strong>while (true) {
      </strong>    ConsumerRecords&#x3C;String, String> records = consumer.poll(500);
          for (ConsumerRecord&#x3C;String, String> record : records) {
              printRecord(record);
          }
          consumer.commitSync();
      }

      or 

      while (true) {
          ConsumerRecords&#x3C;String, String> records = consumer.poll(500);
          for (ConsumerRecord&#x3C;String, String> record : records) {
              printRecord(record);
          }
          consumer.commitAsync(new OffsetCommitCallback() {
              @Override
              public void onComplete(Map&#x3C;TopicPartition, OffsetAndMetadata> map, Exception e) {
                  System.out.println("Consumer Committed Successfully");
              }
          });
      }
      </code></pre>

      \

  *   **commitAsync(), commitSync() 함꼐 사용**\


      ```java
      while (true) {
          ConsumerRecords<String, String> records = consumer.poll(500);
          for (ConsumerRecord<String, String> record : records) {
              printRecord(record);
          }
          consumer.commitAsync(); 
          // 커밋이 실패해도 다음 커밋이 있으므로 커밋이 성공하거나 회복 불가능 할때 까지 
          // 재시도를 합니다.
          consumer.commitSync();
      }
      - 
      ```
  *

#### &#x20;<a href="#undefined" id="undefined"></a>

참고: [https://d2.naver.com/helloworld/0974525](https://d2.naver.com/helloworld/0974525)

참고: [https://docs.confluent.io/platform/current/clients/consumer.html](https://docs.confluent.io/platform/current/clients/consumer.html)

참고: [https://www.confluent.io/blog/apache-kafka-data-access-semantics-consumers-and-membership/?session\_ref=https://harunpeksen.medium.com/how-apache-kafka-consumer-works-6cee4eb83147&\_ga=2.87875742.724744377.1718006844-1261962290.1705303831&\_gl=1\*17ejtkh\*\_gcl\_au\*MTc1ODYzODkyNi4xNzE2NTMwMjU2\*\_ga\*MTI2MTk2MjI5MC4xNzA1MzAzODMx\*\_ga\_D2D3EGKSGD\*MTcxODAwNjg0My43LjEuMTcxODAwNzEyNi40NC4wLjA.](https://www.confluent.io/blog/apache-kafka-data-access-semantics-consumers-and-membership/?session\_ref=https://harunpeksen.medium.com/how-apache-kafka-consumer-works-6cee4eb83147&\_ga=2.87875742.724744377.1718006844-1261962290.1705303831&\_gl=1\*17ejtkh\*\_gcl\_au\*MTc1ODYzODkyNi4xNzE2NTMwMjU2\*\_ga\*MTI2MTk2MjI5MC4xNzA1MzAzODMx\*\_ga\_D2D3EGKSGD\*MTcxODAwNjg0My43LjEuMTcxODAwNzEyNi40NC4wLjA.)
