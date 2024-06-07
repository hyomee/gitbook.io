# Kafka-Producer

카프카 프로듀서는 데이터를 카프카 브로커로 전송하는 역할을 하며, 단순한 전송을 넘어서 파티셔너와 배치 생성 과정을 통해 데이터를 효율적으로 적재합니다. 프로듀서는 카프카 메시지를 브로커의 특정 토픽 파티션으로 전송하고, 이 과정에서 여러 단계를 거쳐 데이터를 효율적으로 처리합니다. 이제 다음과 같은 개념과 기능에 대해 알아보겠습니다.

## 1.  프로듀서(Producer)의 개념

애플리케이션이 Kafka에 메시지를 써야 하는 이유는 메트릭 기록, 로그 메시지 저장, 데이터베이스에 쓰기 전 정보 버퍼링, 센서에서 가져온 데이터 기록 등 여러 가지가 있습니다.

<figure><img src="../../.gitbook/assets/image.png" alt="" width="563"><figcaption><p>카프카:프로듀서  요소 참조:<a href="https://dzone.com/articles/take-a-deep-dive-into-kafka-producer-api">https://dzone.com/articles/take-a-deep-dive-into-kafka-producer-api</a></p></figcaption></figure>

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

프로듀서에서 생성하는 레코드로, 보낼 데이터에 대한 정의를 담고 있습니다. 주요 구성 요소는 토픽, 파티션, 타임스탬프, 메세지 키, 메세지 값입니다. 기본적으로 토픽과 파티션만 있어도 데이터를 보내는 데 지장이 없습니다. 다음은 기본적인 메시지 전달코드 입니다.

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

프로듀서가 메시지를 전송하는 방식은 다음과 같습니다.&#x20;

### 3-1.  Fire and Forget

Future\<RecordMetadata>  리턴값을 무시하여 메시지에 대한 누락 여부를 확인 할 수 없습니다.

```java
try {
    producer.send(producerRecord);
} catch (Exception e) {
    e.printStackTrace();
}
```

### 3-2. Synchronous Send

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

### **3-3. Aynchronous Send**:&#x20;

org.apache.kafka.clients.producer.Callback 인터페이스를 구현한 사용자 정의 클래스를 만들어서 결과를 받아 처리 하면 됩니다.   구현코드는 4가지 방법으로 작성이 가능 합니다.

#### 3-3-1. 클래스 생성&#x20;

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

#### 3-3-2. 익명 함수 생성

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

#### 3-3-3. 람다 생성

<pre class="language-java"><code class="lang-java">Callback onCompletion = (RecordMetadata metadata, Exception exception) -> {
    System.out.println("onCompletion :: " + metadata);
    if (exception != null) {
        exception.printStackTrace();
    }
};

// 호출 방법 
<strong>producer.send(producerRecord, onCompletion );
</strong></code></pre>

#### 3-3-4. 함수 생성

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

## **4. 파티셔너(Partitioner)**

어느 파티션으로 보낼지 지정하는 역할을 합니다. 메세지 키에 따라 지정되며, 동일한 메세지 키는 같은 파티션으로 전송됩니다.

## **5. Accumulator**

배치로 데이터를 묶어 전송할 버퍼를 모으는 역할을 합니다. TCP 통신을 최소화하기 위해 사용됩니다.

*

## 6. 에러 처리

## 7. 객체 직력화



