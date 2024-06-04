# Binding

Exchange와 Queue를 연결하는 관계로 Exchange 타입과 binding 규칙에 따라 적절한 Queue로 전달됩니다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* API 참고: [https://www.rabbitmq.com/client-libraries/java-client](https://www.rabbitmq.com/client-libraries/java-client)

## 1. Exchange&#x20;

메세지를 받고 받은 매새지를 큐로 전달하는 요소로 Exchange가 어떤 Queue로 메시지를 전달하는지 결정하는 라우팅 알고리즘은 Exchange Type과 Binding 규칙에 의해 결정됩니다 즉 Exchange와 Queue를 적절하게 설정하여 메시지를 효율적으로 라우팅할 수 있습니다.

* **Name**: Exchange 이름
* **Type**: 메시지 전달 방식으로 다음과 같은 타입을 지원합니다.
  * **Direct**: 메시지에 포함된 라우팅 키(routing key)를 기반으로 Queue로 메시지를 전달합니다. 각 Queue는 특정 라우팅 키와 연결됩니다.
  * **Fanout**: 라우팅 키와 관계없이 연결된 모든 Queue에 동일한 메시지를 전달합니다.
  * **Topic**: 라우팅 키가 일치하는 Queue로 메시지를 전달합니다. 라우팅 키는 점(.)으로 구분된 단어를 조합해서 정의하며, 와일드 카드(\*)와(#)을 사용하여 패턴을 지정할 수 있습니다.
  * **Headers**: 메시지 헤더를 통해 binding key를 사용하는 것보다 더 다양한 속성을 사용할 수 있으며 헤더 값이 바인딩 시 지정된 값과 같은 경우에만 일치하는 것으로 간주합니다.
* **Durability**: 브로커가 재시작될 때 남아있는지 여부를 결정합니
  * **Durable**: 디스크에 저장되어 재시작 시에도 유지됩니다.
  * **Transient**: 브로커가 재시작되면 삭제됩니다.
* **Auto-delete**: 마지막 Queue 연결이 해제되면 삭제됩니다.

Exchange는 Producer에서 발행한 메시지를 받아서 적절한 Queue로 전달하며, Consumer는 Queue를 통해 메시지를 가져갑니다. Queue는 반드시 미리 정의되어야 하며, 이름, 내구성, 자동 삭제 여부 등의 속성을 갖습니다.

### 1-1. 생성 방법

Exchage는 관리 UI 또는 프로그램 방식을 통해서 만들 수 있습니다.&#x20;

#### 1-1-1. 프로그램 방식으로 Exchange 설정&#x20;

Exchange를 만드는 동안 Name, Durable, Auto-delete 및 Exchange 유형의 3가지 속성을 처리해야 합니다. 기본적으로 생성된 교환은 지속적이며 자동 삭제는 false로 **channel.exchangeDeclare** 메서드를 사용해서 만듭니다. 다음은 기본적인 파라메터 입니다.

* exchange (교환기 이름): 교환기의 이름을 지정합니다. 이 이름은 메시지를 어떤 큐로 라우팅할지 결정하는 데 사용됩니다.&#x20;
* type (교환기 유형): 교환기의 유형을 지정합니다. 일반적으로 다음과 같은 유형이 있습니다:&#x20;
  * fanout: 모든 바인딩된 큐에 메시지를 브로드캐스트합니다.&#x20;
  * direct: 라우팅 키와 일치하는 큐로 메시지를 라우팅합니다.&#x20;
  * topic: 라우팅 패턴을 사용하여 메시지를 큐로 라우팅합니다.&#x20;
  * headers: 헤더 속성을 기반으로 메시지를 큐로 라우팅합니다.&#x20;
* durable (지속성 설정): 교환기를 지속성 있게 만들지 여부를 결정합니다. 지속성이 설정되면 교환기가 서버 재시작 시에도 유지됩니다. (true 또는 false)

아래는RabbitMQ 관리자 UI 에서 Exchange 생성 전 화면입니다.

<figure><img src="../../.gitbook/assets/image (316).png" alt=""><figcaption><p>관리 Tool</p></figcaption></figure>

```java
public class CreateExchange {
    private final static Logger logger = LoggerFactory.getLogger(CreateExchange.class);

    public static void main(String[] args) throws IOException, TimeoutException {
        logger.info("RabbitMQ Create CreateExchange Start [" + CommonConfigs.DEFAULT_QUEUE + "]");

        // 1. Connection 생성
        Connection connection = CommonConfigs.getRabittMQConnection();

        // 2. 채널 생성
        Channel channel = connection.createChannel();

        // 3. Exchange 생성
        channel.exchangeDeclare("My-Direct-Exchange", BuiltinExchangeType.DIRECT, true);

        channel.close();
        connection.close();
    }
}
```

<figure><img src="../../.gitbook/assets/image (317).png" alt=""><figcaption></figcaption></figure>

## &#x20;2. Queue

Queue는 반드시 미리 정의되어야 하며, 이름, 내구성, 자동 삭제 여부 등의 속성을 갖습니다. RabbitMQ를 사용할 때 적절한 Queue 설정을 통해 메시지를 효율적으로 처리할 수 있습니다

* **Name**: Queue 이름으로 `amq.`은 예약어로써 사용할 수 없습니다.
* **Durability**: Queue가 브로커 재시작 시에도 남아 있는지 여부를 결정합니다
  * **Durable**: 브로커가 재시작되어도 디스크에 저장되어 남아 있습니다.
  * **Transient**: 브로커가 재시작되면 사라집니다. 단, Queue에 저장되는 메시지는 내구성을 갖지 않습니다.
* **Auto delete**: 마지막 Consumer가 consume을 끝낼 경우 자동 삭제됩니다.
* **Arguments**: 메시지 TTL, Max Length 같은 추가 기능을 명시할 수 있습니다

