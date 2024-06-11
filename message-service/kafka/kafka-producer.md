# Kafka-Producer

카프카 프로듀서는 데이터를 카프카 브로커로 전송하는 역할을 하며, 단순한 전송을 넘어서 파티셔너와 배치 생성 과정을 통해 데이터를 효율적으로 적재합니다. 프로듀서는 카프카 메시지를 브로커의 특정 토픽 파티션으로 전송하고, 이 과정에서 여러 단계를 거쳐 데이터를 효율적으로 처리합니다. 이제 다음과 같은 개념과 기능에 대해 알아보겠습니다.

## 1.  프로듀서(Producer)의 개념

애플리케이션이 Kafka에 메시지를 써야 하는 이유는 메트릭 기록, 로그 메시지 저장, 데이터베이스에 쓰기 전 정보 버퍼링, 센서에서 가져온 데이터 기록 등 여러 가지가 있습니다.

<figure><img src="../../.gitbook/assets/image (6).png" alt="" width="563"><figcaption><p>카프카:프로듀서  요소 참조:<a href="https://dzone.com/articles/take-a-deep-dive-into-kafka-producer-api">https://dzone.com/articles/take-a-deep-dive-into-kafka-producer-api</a></p></figcaption></figure>

* 카프카 메시지를 작성하려면 ProducerRecord 객체를 생성하여 카프카 브로커에 메시지를 전송해야 합니다. 메시지가 저장될 토픽(Topic)과 값(Value)은 필수 요소이며, 파티션(Partition)과 메시지 키(Key)는 선택적으로 설정할 수 있습니다.
* 메시지는 Send 메서드를 사용하여 전송되며, 직렬화 과정을 거쳐 배열로 변환된 후 명시적으로 지정된 파티션으로 전달됩니다. 지정된 파티션이 없을 경우에는 파티셔너가 메시지를 처리합니다.
* 토픽과 파티션을 통해 전송되는 레코드들은 배치로 모아져 별도의 쓰레드를 통해 카프카 브로커로 전송됩니다.
* 카프카 브로커는 메시지 수신 시 응답을 반환하며, 실패 시 재시도 여부를 결정해 해당 횟수만큼 재시도를 진행합니다.

## 2. KafkaProducer

KafkaProducer는 클라이언트 애플리케이션으로, 이벤트를 카프카 클러스터에 발행합니다. Java에서는 KafkaProducer 클래스를 통해 클러스터와 연결하고, 이 클래스는 브로커 주소와 같은 구성 매개변수를 맵 형태로 제공합니다. 프로듀서는 스레드에 안전하여, 여러 스레드가 단일 프로듀서 인스턴스를 공유하는 것이 여러 인스턴스를 사용하는 것보다 보통 더 효율적입니다.

### 2-1. 카프카 프로듀서 생성

```java
// KafkaProducer 생성
public class ProducerMain {
    public static void main(String[] args) throws IOException {
    
        // KafkaProducer 설정 정보
        Properties configs = KafkaProperties.getProducerProperties();
        
        // KafkaProducer 생성
        KafkaProducer<String, String> producer = new KafkaProducer<>(configs);
        
    }
}
```

### 2-2. 카프카 연결 정보 설정

참고: [카프카 프로듀서 설정 3.3 Producer Configs](https://kafka.apache.org/documentation/#producerconfigs)

```java
public class KafkaProperties {
    public static String KARFA_SERVER_IP = "172.24.239.164:9092";
    public static String KARFA_TOPIC = "iabacus-test";

    public static Properties getProducerProperties() {
        Properties configs = new Properties();
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, KARFA_SERVER_IP);
        configs.put(ProducerConfig.ACKS_CONFIG, "1"); 
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG ,
             "org.apache.kafka.common.serialization.StringSerializer");
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
            "org.apache.kafka.common.serialization.StringSerializer");
        return configs;
    } 
}
```

* **BOOTSTRAP\_SERVERS\_CONFIG = "bootstrap.servers"**: 카프카 클러스터와 연결할 브커커 연결 정보
  * 카프카 브로커 중 하나 정지되어도 카프카 클러스터에 연결할 수 있도록 최소 2개 이상을 권장합니다.
  * 예:  "172.24.239.164:9092, 172.24.239.165:9092, 172.24.239.166:9092"&#x20;
* **KEY\_SERIALIZER\_CLASS\_CONFIG = "key.serializer"**: 카프키에 쓸 레코드의 키 값을 직렬화하기 위해 사용하는 시리얼라이저 클래스 이름입니다.
  * 참고: o[rg.apache.kafka.common.serialization ](https://kafka.apache.org/31/javadoc/org/apache/kafka/common/serialization/package-summary.html)
  * 기본적으로 제공되는 직렬화 클래스는 StringSerializer, ShortSerializer, IntegerSerializer, LongSerializer DoubleSerializer, BytesSerializer 입니다.
  *   사용자 정의 객체를 직렬화하려면 Serializer 인터페이스를 구현해야 합니다.

      ```java
      @Data
      @AllArgsConstructor
      @NoArgsConstructor
      @Builder
      public class MessageDto {
          private String message;
          private String version;
      }

      public class CustomSerializer implements Serializer<MessageDto> {
          // Implement the serialization logic here
          // ...
      }
      ```
* **VALUE\_SERIALIZER\_CLASS\_CONFIG = "value.serializer"**: 카프키에 쓸 레코드의 값을직렬화하기 위해 사용하는 시리얼라이저 클래스 이름입니다.
* **ACKS\_CONFIG = "acks**": 프로듀서(Producer)가 메시지를 보내고 그 메시지를 카프카가 잘 받았는지 확인할 것인지를 결정하는 옵션입니다.
  * **acks = 0**: 브로커로부터 응답을 요청하지 않습니다. 따라서 브로커가 오프라인이 되거나 예외가 발생하면 우리는 알 수 없으며 데이터를 잃게 됩니다. 메트릭이나 로그 수집과 같이 메시지 손실이 가능한 데이터에 적합합니다. 프로듀서는 어떤 확인도 기다리지 않으므로 최상의 성능을 제공합니다.&#x20;
  * **acks = 1 (기본값)**: 리더 파티션의 응답이 요청됩니다. 그러나 복제는 보장되지 않습니다. 이는 백그라운드에서 발생합니다. 응답을 받지 못하면 프로듀서는 중복 데이터 없이 재시도할 수 있습니다. 리더 브로커가 오프라인되지만 복제본이 아직 데이터를 복제하지 않은 경우 데이터가 손실됩니다.&#x20;
  * **acks = all**: 리더와 복제본 모두 응답을 요청합니다. 이는 프로듀서가 계속하기 전에 브로커의 어떤 복제본에서든 확인을 받아야 함을 의미합니다. 이는 지연을 추가하지만 데이터 손실 없이 안전성을 보장합니다.

## **3. ProducerRecord**&#x20;

ProducerRecord는 Kafka로 데이터를 전송하는 데 사용되는 클래스입니다. 이 클래스는 토픽 이름, 선택적 파티션 번호, 선택적 키 및 값으로 구성됩니다. 파티션 번호가 명시되지 않은 경우, 키를 기반으로 파티션이 선택되고, 키가 없을 때는 라운드 로빈 방식으로 파티션이 할당됩니다. 레코드에는 또한 브로커가 사용할 타임스탬프가 포함되어 있으며, 이는 토픽 설정에 따라 CreateTime 또는 LogAppendTime으로 결정됩니다.

```java
public class ProducerRecord<K, V> {
    private final String topic;
    private final Integer partition;
    private final Headers headers;
    private final K key;
    private final V value;
    private final Long timestamp;

    public ProducerRecord(String topic, 
                          Integer partition, 
                          Long timestamp, 
                          K key, 
                          V value, 
                          Iterable<Header> headers) {
        if (topic == null) {
            throw new IllegalArgumentException("Topic cannot be null.");
        } else if (timestamp != null && timestamp < 0L) {
            throw new IllegalArgumentException(String.format("Invalid timestamp: %d. Timestamp should always be non-negative or null.", timestamp));
        } else if (partition != null && partition < 0) {
            throw new IllegalArgumentException(String.format("Invalid partition: %d. Partition number should always be non-negative or null.", partition));
        } else {
            this.topic = topic;
            this.partition = partition;
            this.key = key;
            this.value = value;
            this.timestamp = timestamp;
            this.headers = new RecordHeaders(headers);
        }
    }
    ....
}
     
```

* topic: 레코드가 전송될 Kafka 토픽의 이름입니다.&#x20;
* partition: 레코드를 전송할 파티션 번호입니다. 파티션 번호를 직접 지정하면 해당 파티션으로 레코드가 전송됩니다.
* timestamp: 레코드의 타임스탬프입니다. 이 값은 레코드가 생성된 시간을 나타냅니다. 사용자가 타임스탬프를 제공하지 않으면 프로듀서는 현재 시간으로 레코드를 타임스탬프합니다.&#x20;
* key: 레코드의 키입니다. 키는 레코드를 특정 파티션에 정렬하기 위해 사용됩니다.&#x20;
* value: 레코드의 값입니다. 실제 데이터를 포함하며 Kafka로 전송됩니다.&#x20;
* headers: 레코드에 연결된 헤더 정보입니다. 헤더는 추가적인 메타데이터를 포함할 수 있습니다.&#x20;

### 3-1. Key **Partitioner**

카프카 메시지는 Key-Value 구조를 가지고 있는데 Key 없이  사용할 수 있습니다. Key의 역할은 메시지에 저장 되는 추가적인 정보이지만 하나의 토픽에 속한 여러개의 파티션 중 메시지가 저장될 파티션을 결정짓는 기준점입니다.

```java
public ProducerRecord(String topic, K key, V value) 
public ProducerRecord(String topic, V value)
```

#### **3-1-1. Key = null**:&#x20;

* 레코드는 토픽의 파티션 중 하나에 라운드 로빈(Round Robin) 알고리즘을 사용하여 각 파티션별로 저당되는 메시지의 균형을 맞춥니다.
*   아파치 카프카 2.4 부터는 접착성 처리를 하기 위해 라운드 로빈 알고리즘을 사용 합니다. 즉 프로듀서가 메시지를 배치 할 떄 이전 배치를 먼저 채우게 하여 더 적은 요청을 같은 수의 메시지를 전송하게하여 지연시간을 줄이고 브로커의 CPU 사용량을 줄입니다.\


    <figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>파티션에서의 접착성 차이</p></figcaption></figure>

    \


```java
for (int i = 0; i < 5; i++) {
    String v = "hello :: " + i; 
    ProducerRecord producerRecord = 
        new ProducerRecord<>(KafkaProperties.KARFA_TOPIC_P5, v );
    try {
        producer.send(producerRecord, onCompletion );
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

* &#x20;Consumer 결과: 발송 할 때 마다 파티션이 변경됨

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

#### **3-1-2.** Key = 있음:&#x20;

지정한 Key를 해시(Hash)한 결과를 기준으로 파티션을 결정합니다. 해시한 값은 변경되지 않으므로 동일한 파티션에 저장됩니다.

```java
for (int i = 0; i < 5; i++) {
    String v = "hello :: " + i;
    String k = "key :: " + i;
    ProducerRecord producerRecord 
        = new ProducerRecord<>(KafkaProperties.KARFA_TOPIC_P5, k , v );
    try {
        producer.send(producerRecord, onCompletion );
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

*   &#x20;Consumer 결과: key에  의해 파티션이 고정됨\


    <figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

다음은 기본적인 메시지 전달코드 입니다.

{% code lineNumbers="true" %}
```java
public class ProducerMain {
    public static void main(String[] args) throws IOException {
        // KafkaProducer 설정 정보
        Properties configs = KafkaProperties.getProducerProperties();

        // KafkaProducer 생성
        KafkaProducer<String, String> producer = new KafkaProducer<>(configs);
         
        
        for (int i = 0; i < 5; i++) {
        
            String msg = "hello"+i;
            
            // ProducerRecord 생성
            ProducerRecord producerRecord = 
                new ProducerRecord<>(KafkaProperties.KARFA_TOPIC, msg ); 
            
            // 카프카 브로커에 메시지 전달
            try {
                producer.send(producerRecord);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        // 종료
        producer.flush();
        producer.close();
    }

}
```
{% endcode %}

* 14 Line: 전송되는 레코드 생성&#x20;
* 20 Line: Fire and Forget 전송 방법으로 카프카 브로커에 메시지 전달합니다.
  * send는Future\<RecordMetadata>로  리턴을 받는데 값을 무시 하기 때문에 성공 여부를 알 수 없습니다.
* 22 Line: 메세지 전달 중 오류 발생
  * SerializationException: 메세지 직렬화 오류
  * TimeoutException: 버퍼가 가득찰 경우 오류
  * InterruptException: 전송 작업 중 인터럽트 오류
  * 재시도 오류는 재시도 횟수 지정으로 방지 할 수 있지만 초과시는 오류가 발생합니다.

### **3-2. 지정 Partitioner**

기본 파티셔너는 키 값의 존재 여부에 따라 메시지를 어떤 파티션으로 보낼지 결정합니다. 메시지 키를 기준으로 지정되며, 같은 메시지 키를 가진 메시지는 동일한 파티션으로 전송됩니다. 파티션을 지정하면 해당 파티션에 저장됩니다&#x20;

```java
public ProducerRecord(String topic, 
                     Integer partition, 
                     Long timestamp, 
                     K key, 
                     V value) 
public ProducerRecord(String topic, 
                      Integer partition, 
                      K key, 
                      V value)
```

#### 3-2-1. 파티션 지정

지정된 파티션에 저장됩니다.

```java
for (int i = 0; i < 5; i++) {
        String v = "hello :: " + i;
        String k = "key :: " + i;
       
        ProducerRecord producerRecord =
                new ProducerRecord<>(KafkaProperties.KARFA_TOPIC_P5,
                        KafkaProperties.KARFA_TOPIC_PARTITION_3,
                        k ,
                        v );
        try {
            producer.send(producerRecord, onCompletion );
        } catch (Exception e) {
            e.printStackTrace();
        }
 }
```

* Consumer 결과: 지정된 파티션에 고정

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### 3-3.  Custom **Partitioner**

key, value 를 사용 하여 특정 파티션에 메시지를 저장하는 하게 하는 기능으로 특정 메세지를 특정 파티션으로 보내어 많이 발생하는 메세지를 하나로 보내어 성능 향상을 할 수 있습니다.

*   **KafkaProducer 객체 생성 시 프로퍼티설정**:\
    \
    "partitioner.class"(ProducerConfig.PARTITIONER\_CLASS\_CONFIG) 에 작성한 클래스를 지정 합니다.\


    ```java
     public static Properties getProducerProperties() {
            Properties configs = new Properties();
            configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, KARFA_SERVER_IP);
            configs.put(ProducerConfig.ACKS_CONFIG, "1");
            configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG , "org.apache.kafka.common.serialization.StringSerializer");
            configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
            configs.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, "kr.co.abacus.jmsbroker.kafka.CustomPartition");
            return configs;
        }
    ```



*   **Partitioner 인터페이스 구현**:\


    {% code lineNumbers="true" %}
    ```java
    public class CustomPartition implements Partitioner {
        @Override
        public int partition(String topic,
                             Object key,
                             byte[] keyBytes,
                             Object value,
                             byte[] valueBytes,
                             Cluster cluster) {

            List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
            int numPartitions = partitions.size();

            if (keyBytes == null || !(key instanceof String)) {
                throw new InvalidRecordException("Invalid key for CustomPartitioner");
            }

            // 마미막 파티션으로 리턴
            if (key.equals("key :: 3")){
                return numPartitions - 1 ;
            }
            
            // 다른 파티션으로 리턴
            return Math.abs(Utils.murmur2(keyBytes)) % (numPartitions-1);
        }

        @Override
        public void close() {

        }

        @Override
        public void configure(Map<String, ?> map) {

        }
    }
    ```
    {% endcode %}

### 3-4.  Header

메시지의 Key-Value 구조를 변경하지 않고 메타데이터를 추가하는 데 사용되며, 헤더는 순서와 상관없이 Key-Value 쌍으로 작성됩니다. 직렬화와는 무관하며, 주요 목적은 다음과 같습니다.

* 메시지의 전당 내역 기록: 메시지의 생성 정보(출처, 암호화 정보 등)

```java
public ProducerRecord(String topic, 
                     Integer partition, 
                     Long timestamp, 
                     K key, 
                     V value, 
                     Iterable<Header> headers) 
public ProducerRecord(String topic, 
                      Integer partition, 
                      K key, 
                      V value, 
                      Iterable<Header> headers) 
```

* producerRecord.headers().add() 메서드를 사용하여 해더를 추가 합니다.

{% code lineNumbers="true" %}
```java
for (int i = 0; i < 5; i++) {
    String v = "hello :: " + i;
    String k = "key :: " + i;
    
    ProducerRecord producerRecord =
             new ProducerRecord<>(KafkaProperties.KARFA_TOPIC_P5, k , v );
    producerRecord.headers().add("INPUT-CHANNEL", "IPHONE".getBytes(StandardCharsets.UTF_8));

    try {
        producer.send(producerRecord, onCompletion );
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
{% endcode %}

*   Consumer 결과: record.headers()를사용하여  header 객체의  Key-Value를 꺼내 사용합니다.\


    ```java
    private static void printRecord(ConsumerRecord<String, String> record) {    
        System.out.println("topic = " + record.topic() + 
                            ":: partition = " + record.partition()  + 
                            ":: key = " + record.key() + 
                            ":: value = " + record.value());
        
        record.headers().forEach(header -> {
            System.out.println(header.key() + ": " + new String(header.value()));
        });
    }

    겱과
    topic = iabacus-p5:: partition = 2:: key = key :: 2:: value = hello :: 2
    INPUT-CHANNEL: IPHONE
    topic = iabacus-p5:: partition = 4:: key = key :: 3:: value = hello :: 3
    INPUT-CHANNEL: IPHONE
    topic = iabacus-p5:: partition = 1:: key = key :: 0:: value = hello :: 0
    INPUT-CHANNEL: IPHONE
    topic = iabacus-p5:: partition = 0:: key = key :: 1:: value = hello :: 1
    INPUT-CHANNEL: IPHONE
    topic = iabacus-p5:: partition = 0:: key = key :: 4:: value = hello :: 4
    INPUT-CHANNEL: IPHONE
    ```

## 4. Sender

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

프로듀서가 메시지를 전송하는 방식은 다음과 같습니다.&#x20;

### 4-1.  Fire and Forget

Future\<RecordMetadata>  리턴값을 무시하여 메시지에 대한 누락 여부를 확인 할 수 없습니다.

```java
try {
    producer.send(producerRecord);
} catch (Exception e) {
    e.printStackTrace();
}
```

### 4-2. Synchronous Send

&#x20;Future\<RecordMetadata>  리턴값을 받을 때 까지 기다리는 것으로 대량 처리인 경우 주의가 필요합니다. 리턴값을 받으므로 RecordMetadata 정보를 활용 할 수 있습니다. ( 오프셋 등 )

```java
try {
    Future<RecordMetadata> recoedMeta = producer.send(producerRecord);
    // 리턴 값이 올 때 까지 대기 
    RecordMetadata metadata = recoedMeta.get();
    System.out.println(metadata.topic());
} catch (Exception e) {
    e.printStackTrace();
}
```

### **4-3. Aynchronous Send**:&#x20;

org.apache.kafka.clients.producer.Callback 인터페이스를 구현한 사용자 정의 클래스를 만들어서 결과를 받아 처리 하면 됩니다.   구현코드는 4가지 방법으로 작성이 가능 합니다.

#### 4-3-1. 클래스 생성&#x20;

```java
public class CallBackProducer implements Callback {
    @Override
    public void onCompletion(RecordMetadata metadata, Exception exception) {
        System.out.println("onCompletion :: " + metadata);
        if (exception != null) {
            exception.printStackTrace();
        }
    }
}

// 호출 방법 
producer.send(producerRecord, new CallBackProducer());
```

#### 4-3-2. 익명 함수 생성

```java
Callback onCompletion = new Callback() {
    public void onCompletion(RecordMetadata metadata, Exception exception) {
        System.out.println("onCompletion :: " + metadata);
        if (exception != null) {
            exception.printStackTrace();
        }
    }
};

// 호출 방법 
producer.send(producerRecord, onCompletion );
```

#### 4-3-3. 람다 생성

<pre class="language-java"><code class="lang-java">Callback onCompletion = (RecordMetadata metadata, Exception exception) -> {
    System.out.println("onCompletion :: " + metadata);
    if (exception != null) {
        exception.printStackTrace();
    }
};

// 호출 방법 
<strong>producer.send(producerRecord, onCompletion );
</strong></code></pre>

#### 4-3-4. 함수 생성

```java
public Callback onCompletion () {
    return (RecordMetadata metadata, Exception exception) -> {
        System.out.println("onCompletion :: " + metadata);
        if (exception != null) {
            exception.printStackTrace();
        }
    };
}

or 

public Callback onCompletion = (RecordMetadata metadata, Exception exception) -> {
        System.out.println("onCompletion :: " + metadata);
        if (exception != null) {
            exception.printStackTrace();
        }
};

// 호출 방법 
producer.send(producerRecord, onCompletion );
    
```

## **6. 직렬화**

카프카 프로듀서를 설정할 때, 키와 값에 대한 시리얼라이저(serializer)를 지정해야 합니다. 기본적으로 제공되는 직렬화 클래스에는 StringSerializer, ShortSerializer, IntegerSerializer, LongSerializer, DoubleSerializer, BytesSerializer가 있으며, 자바 객체를 직렬화하기 위해서는 추가적인 방법이 두 가지 있습니다.

* **Serializer를 사용한 직렬화 구현**: Serializer 인터페이스를 상속 받아 구현체를 작성합니다.
  * &#x20;Serializer를 사용한 직렬화 구현은 객체의 속성에 대한 맴버 변수에 대해서 각각 정의해야 하고 타입이 변경 되거나 맴버가 추가 되면 수정을 해야 하는 등 많은 작업을 해야 하고 개발 해야하고 개발시 코드 취약점등 고려 사항이 많습니다. 다음 코드는 직렬화 예제 코드 입니다.

<details>

<summary>자바 직렬화 란</summary>

자바 직렬화는 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트 (byte) 형태로 데이터 변환하는 기술입니다. 이렇게 바이트로 변환된 데이터를 다시 객체로 변환하는 기술을 역직렬화라고 합니다

</details>

<details>

<summary>직렬화 예제</summary>

```java
public class CustomerRecordSerializer 
    implements Serializer<Customer> {

    @Override
    public void configure(Map<String, ?> configs, boolean isKey) {
        // Configuration (if needed)
    }

    @Override
    public byte[] serialize(String topic, Customer data) {

        byte[] serializedData = null;
        int stringSize;

        if (data == null) {
            return null;
        }

        if (data.getName() != null) {
            serializedData = data.name().getBytes(StandardCharsets.UTF_8);
            stringSize = serializedData.length;
        } else {
            serializedData = new byte[0];
            stringSize = 0;
        }

        ByteBuffer buffer = ByteBuffer.allocate(8 + stringSize);
        buffer.putInt(data.age());
        buffer.putInt(stringSize);
        buffer.put(serializedData);

        return buffer.array();
    }

    @Override
    public void close() {
        // Cleanup (if needed)
    }
}
```

</details>

* **벙용 직력화 오픈 소스 사용**:  에이브로(Avro) 와 같은 라이브러리 사용합니다.
  * 참고: [https://github.com/davamigo/kafka-examples-avro](https://github.com/davamigo/kafka-examples-avro)
  * 참고: [https://howtodoinjava.com/kafka/kafka-with-avro-and-schema-registry/](https://howtodoinjava.com/kafka/kafka-with-avro-and-schema-registry/)
*   **Josn/xml 관련  오픈 소스 사용**: Jackson 라이브러리, Gson 라이브러리, XStream 라이브러리\


    ```java
    for (int i = 0; i < 5; i++) {
        String v = "hello :: " + i;
        String k = "key :: " + i;
        
        CustomerRecord customerRecord = new CustomerRecord(v, i);
        Gson gson = new Gson();
        String json = gson.toJson(customerRecord);
        
        ProducerRecord producerRecord =
                 new ProducerRecord<>(KafkaProperties.KARFA_TOPIC_P5, k , json );
        producerRecord.headers().add("INPUT-CHANNEL", "IPHONE".getBytes(StandardCharsets.UTF_8));
        
        try {
            producer.send(producerRecord, onCompletion );
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    ```

## **7. Interceptor**

Kafka 인터셉터는 클라이언트 모니터링, 엔드 투 엔드 시스템 성능 감지, 메시지 감사 등의 시나리오에 활용될 수 있습니다. Kafka 인터셉터는 버전 0.10 도입 이후로 모니터링을 위해 실제로 유사한 기능을 구현하는 데에는 널리 사용되지 않았습니다.

참고 : [Add Producer and Consumer Interceptors](https://cwiki.apache.org/confluence/display/KAFKA/KIP-42%3A+Add+Producer+and+Consumer+Interceptors) , [kafka series Producer interceptor](https://programmer.ink/think/kafka-series-producer-interceptor.html)

## **8. Quota/T**hrottling

프로듀서 쿼터와 쓰로틀링은 카프카 클러스터의 안정적 운영과 성능 최적화에 필수적인 요소입니다. 이들은 카프카 클러스터의 안정성과 성능을 관리하는 데 중요한 역할을 하며, 프로듀서와 컨슈머의 리소스 사용을 제한하는 쿼터 설정을 통해 쓰로틀링을 구현할 수 있습니다.

* 프로듀서 쿼터는 카프카 클러스터 내에서 프로듀서가 생성하는 메시지의 양을 제한하는 설정입니다. 이는 클라이언트의 데이터 전송에 영향을 미치는 쓰기 쿼터, 읽기 쿼터와 브로커가 요청을 처리하는 데 영향을 미치는 요청 쿼터를 포함합니다.
  * 쓰기 쿼터, 읽기 쿼터: 전송하거나 받는 속도를 초당 바이트 수 단위로 제한 합니다.
  * 요청 쿼터: 요청간처리에 대한 시간 비율로 제한 합니다.
* 쿼터 설정에는 기본값 설정, 특정 Client.id 설정, 그리고 특정 사용자 설정이 포함됩니다. 특정 사용자 설정은 보안 기능과 클라이언트 인증 기능이 활성화된 클라이언트에 적용됩니다.
  * quote.producer.default=1M: 각각의로듀서가 초당 평균적으로 쓸 수 있는 데이터 1M 제한 설정
  * quote.producer.override="clientA:1M, clientB:2M": 클라이언트별 제한 설정(권장사항이 아님)
* 이를 통해 클러스터 리소스를 효율적으로 관리하고 과도한 메시지 생성으로 인한 장애를 방지할 수 있습니다.
* 예를 들어, 초당 메시지 생성량이 너무 많으면 프로듀서 쿼터를 설정하여 제한할 수 있습니다.
*   프로듀서 쿼터를 설정하려면  server.properties 파일에서 다음과 같이 설정합니다

    ```properties
    # 프로듀서 쿼터 설정
    producer.quota.bytes.per.second=1048576  # 초당 바이트 수 제한
    producer.quota.messages.per.second=1000  # 초당 메시지 수 제한
    ```
*   컨슈머 쿼터를 설정하려면 server.properties 파일에서 다음과 같이 설정합니다.

    ```properties
    # 컨슈머 쿼터 설정
    consumer.quota.messages.per.second=500  # 초당 메시지 수 제한
    ```
*   특정 토픽에 대한 쿼터를 설정하려면 해당 토픽의 server.properties 파일에서 다음과 같이 설정합니다.

    ```properties
    # 토픽별 쿼터 설정
    topic.<topic_name>.quota.bytes.per.second=524288  # 초당 바이트 수 제한
    topic.<topic_name>.quota.messages.per.second=200  # 초당 메시지 수 제한
    ```

## **9. 소스**

{% tabs %}
{% tab title="KafkaProperties " %}
```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Properties;

public class KafkaProperties {
    public static String KARFA_SERVER_IP = "172.24.239.164:9092";
    public static String KARFA_TOPIC = "iabacus-test";
    public static String KARFA_TOPIC_P5 = "iabacus-p5";
    public static int KARFA_TOPIC_PARTITION_3 = 3;

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
        
        // CustomPartition  
        configs.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, "kr.co.abacus.jmsbroker.kafka.CustomPartition");
        return configs;
    }


    public static Properties getConsumerProperties() {
        Properties configs = new Properties();
        // kafka server host 및 port
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KARFA_SERVER_IP);
        // session 설정
        configs.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "10000");

        // topic 설정
        configs.put(ConsumerConfig.GROUP_ID_CONFIG, KARFA_TOPIC);

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
{% endtab %}

{% tab title="ProducerMain " %}
```java
import com.google.gson.Gson;
import kr.co.abacus.jmsbroker.kafka.dto.CustomerRecord;
import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Properties;
import java.util.concurrent.Future;


public class ProducerMain {
    public static void main(String[] args) throws IOException {

        Properties configs = KafkaProperties.getProducerProperties();

        // producer 생성
        KafkaProducer<String, String> producer = new KafkaProducer<>(configs);

        // producer.send(new ProducerRecord<>(KafkaProperties.KARFA_TOPIC, "ff", "vv"));
        // message 전달
        // 비동기 전송을 위한 Call 함수
        Callback onCompletion = new Callback() {
            public void onCompletion(RecordMetadata metadata, Exception exception) {
                System.out.println("onCompletion :: " + metadata.partition() + "::"+ metadata.toString());
                if (exception != null) {
                    exception.printStackTrace();
                }
            }
        };

//        Callback onCompletion = (RecordMetadata metadata, Exception exception) -> {
//            System.out.println("onCompletion :: " + metadata);
//            if (exception != null) {
//                exception.printStackTrace();
//            }
//        };

        for (int i = 0; i < 5; i++) {
            String v = "hello :: " + i;
            String k = "key :: " + i;

            // ProducerRecord producerRecord = new ProducerRecord<>(KafkaProperties.KARFA_TOPIC_P5,  v );

            CustomerRecord customerRecord = new CustomerRecord(v, i);
            Gson gson = new Gson();
            String json = gson.toJson(customerRecord);

            ProducerRecord producerRecord =
                     new ProducerRecord<>(KafkaProperties.KARFA_TOPIC_P5, k , json );
            producerRecord.headers().add("INPUT-CHANNEL", "IPHONE".getBytes(StandardCharsets.UTF_8));

//            ProducerRecord producerRecord =
//                    new ProducerRecord<>(KafkaProperties.KARFA_TOPIC_P5,
//                            KafkaProperties.KARFA_TOPIC_PARTITION_3,
//                            k ,
//                            v );


            try {
                producer.send(producerRecord, onCompletion );
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        // 종료
        producer.flush();
        producer.close();
    }

    private static Callback producerCallback = (RecordMetadata metadata, Exception exception) -> {
        System.out.println("onCompletion :: " + metadata);
        if (exception != null) {
            exception.printStackTrace();
        }
    };

}

```
{% endtab %}

{% tab title="ConsumerMain " %}
```java
import kr.co.abacus.jmsbroker.kafka.dto.CustomerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.util.Arrays;
import java.util.Properties;

public class ConsumerMain {
    public static void main(String[] args) {

        System.out.println("Consumer Start ....");
        Properties configs = KafkaProperties.getConsumerProperties();

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(configs);    // consumer 생성
        consumer.subscribe(Arrays.asList(KafkaProperties.KARFA_TOPIC_P5)); // topic 설정

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

{% tab title="CustomerRecord" %}
```java
public record CustomerRecord(String name,
                             Integer age) {

    public CustomerRecord {
        if (name == null || age == null  ) {
            throw new IllegalArgumentException();
        }
    }


    public String getInfo() {
        return this.name + " " + this.age;
    }
}

```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="CallBackProducer " %}
```java
import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.RecordMetadata;

public class CallBackProducer implements Callback {
    @Override
    public void onCompletion(RecordMetadata metadata, Exception exception) {
        System.out.println("onCompletion :: " + metadata.partition());
        if (exception != null) {
            exception.printStackTrace();
        }
    }
}

```
{% endtab %}

{% tab title="CustomPartition " %}
```java
import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;
import org.apache.kafka.common.InvalidRecordException;
import org.apache.kafka.common.PartitionInfo;
import org.apache.kafka.common.utils.Utils;

import java.util.List;
import java.util.Map;

public class CustomPartition implements Partitioner {
    @Override
    public int partition(String topic,
                         Object key,
                         byte[] keyBytes,
                         Object value,
                         byte[] valueBytes,
                         Cluster cluster) {

        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();

        if (keyBytes == null || !(key instanceof String)) {
            throw new InvalidRecordException("Invalid key for CustomPartitioner");
        }

        if (key.equals("key :: 3")){
            return numPartitions - 1 ;
        }

         return Math.abs(Utils.murmur2(keyBytes)) % (numPartitions-1);
    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> map) {

    }
}

```
{% endtab %}
{% endtabs %}
