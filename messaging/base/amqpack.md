# 송수신 확인

AMQP(Advanced Message Queuing Protocol)는 메시지 브로커 시스템에서 메시지 송수신을 관리하는 프로토콜입니다. AMQP는 네트워크 문제나 요청 처리 실패 시를 대비하여 두 가지 수신 확인 모델을 제공합니다:

## 1. Kafka

카프카의 메세지 승인은 Kafka 브로커(서버)와 클라이언트(생산자 및 소비자) 간의 메시지 전달의 안정성을 보장하는 데 매우 중요합니다. Kafka에서 메시지 승인이 작동하는 방식은 다음과 같습니다.

<figure><img src="../../.gitbook/assets/image (297).png" alt=""><figcaption></figcaption></figure>

### 1-1. **Producer 측에서의 Acknowledgment**

Producer는 메시지가 토픽에 성공적으로 저장되었음을 브로커로부터 확인하기 위해 Producer  **acks**설정을 사용하여 메시지가 토픽에 성공적으로 저장되었음을 브로커로부터 확인할 수 있습니다.

* 리더 파티션을 담당하는 카프카가 메세지를 수신 받으면&#x20;
  * acks가 0또는 1로 설정이 되어 있으면 브로커는 즉시 Producer(생산자)에게 수신을 합니다.
  * acks=all"로 설정된 경우 리더 브로커는 파티션의 ISR(동기화된 복제본)에 메시지를 전달합니다.

<table><thead><tr><th width="152">acks</th><th>설명</th></tr></thead><tbody><tr><td>acks=0 <br>(승인 없음)</td><td>Producer(생산자)가 승인을 기다리지 않습니다. 메시지를 보내고 수신 여부는 신경 쓰지 않습니다.</td></tr><tr><td>acks=1<br>(리더확인)</td><td>Producer(생산자)는 메시지가 전송되는 주제 파티션의 리더 브로커로부터 승인을 기다립니다. 즉, 리더 브로커가 수신을 확인하면 메시지가 전송된 것으로 간주되므로  리더가 레코드를 확인한 후 팔로워가 복제하기 전에 리더가 즉시 실패하면 레코드가 손실될 수 있습니다. (리더는 레코드를 로컬 로그에 기록하지만 모든 팔로워로부터 완전한 확인을 기다리지 않고 응답)</td></tr><tr><td>acks=all<br>(복제본 승인)<br>or<br>acks=-1  </td><td>Producer(생산자)는 파티션의 동기화된 모든 복제본에서 승인을 기다립니다. 이렇게 하면 최고 수준의 내구성이 제공되지만 대기 시간이 더 길어질 수 있습니다. 즉 메시지가 동기화된 모든 복제본에 성공적으로 복제되면 리더 브로커에 대한 수신을 다시 승인합니다. 그런 다음 리더 브로커는 생산자에게 이를 인정합니다 (인-싱크 레플리카가 살아있는 한 레코드가 손실되지 않음을 보장합니다.)</td></tr></tbody></table>

### 1-2. **Consumer 측에서의 Acknowledgment**

Consumer 측에서는 **Consumer Group** 개념을 사용하여 각 Consumer Group은 처리한 메시지를 확인하기 위해 브로커로부터 오프셋을 커밋합니다.

* 기본적으로 Consumers는 Consumer 설정인 enable.auto.commit 및 auto.commit.interval.ms를 통해 5초마다 자동으로 Kafka에 오프셋을 커밋합니다. 또는 컨슈머 내에서 처리 로직에 따라 수동으로 오프셋을 커밋할 수 있습니다
* **Auto Commit (자동 커밋)**:
  * 기본적으로 Kafka Consumer는 자동 커밋 모드로 동작합니다.
  * Consumer 설정:  enable.auto.commit 및 auto.commit.interval.ms
  * enable.auto.commit = true: 컨슈머가 메시지를 처리한 후 일정 시간속성기본값은 5초)마다 자동으로 오프셋을 커밋합니다. (auto.commit.interval.ms)
  * 이 방식은 간편하지만, 메시지 처리 중에 장애가 발생하면 메시지가 중복으로 처리될 수 있습니다[1](https://stackoverflow.com/questions/56208505/how-to-acknowledge-kafka-message-read-by-the-consumer-using-spring-integration-k).
* **Manual Commit (수동 커밋)**:
  * 수동 커밋 모드에서는 컨슈머가 메시지 처리 후 명시적으로 오프셋을 커밋해야 합니다.
  * 컨슈머는 메시지 처리가 완료된 후 Acknowledgment객체를 사용하여 오프셋을 수동으로 커밋합니다.
  * Spring Integration을 사용하는 경우, Acknowledgment객체는 KafkaHeaders.ACKNOWLEDGMENT 헤더를 통해 사용할 수 있습니다
  * 참고 : [https://stackoverflow.com/questions/56208505/how-to-acknowledge-kafka-message-read-by-the-consumer-using-spring-integration-k](https://stackoverflow.com/questions/56208505/how-to-acknowledge-kafka-message-read-by-the-consumer-using-spring-integration-k)

## 2. RabbitMQ

RabbitMQ의  메시지를 성공적으로 받았음을 확인하는 메커니즘으로 데이터 안전성과 신뢰성을 보장하기 위해 중요한 역할을 합니다.

참고: [Consumer Acknowledgements and Publisher Confirms ](https://www.rabbitmq.com/docs/confirms)

### **2-1. Publisher Acknowledgment Model (발행자 확인 모델)**:

**Publisher**가 메시지를 브로커로 성공적으로 전송했음을 확인하기 위해 사용되는.것으로 **Publisher**가 메시지를 브로커로 전송한 후 브로커로부터 확인을 받으면 메시지가 안전하게 전달되었다고 판단합니다.

#### **2-1-1. Publisher Confirms:**

&#x20;RabbitMQ에서 신뢰성 있는 메시지 발행을 구현하기 위한 확장 기능으로 퍼블리셔가 메시지를 발행한 후 브로커에서 비동기적으로 확인을 받습니다. 이는 서버 측에서 메시지 처리가 완료되었음을 의미합니다

* Publisher가 메시지를 발행한 후 브로커로부터 확인 응답을 받아야 합니다.
* 브로커는 메시지를 받았을 때 발행자에게 확인 응답을 보냅니다.
* 발행자는 브로커로부터 확인 응답을 받으면 해당 메시지를 안전하게 처리했다고 판단하고 다음 메시지를 발행합니다.
* 이 모델은 메시지 발행 시 손실을 최소화하고, 발행자가 메시지를 안전하게 전송할 수 있도록 합니다.
* 활성화 방법:
  * Publisher confirms를 사용하려면 채널에서 confirm.select 메서드를 호출해야 합니다.
  * 브로커는 confirm.select-ok 응답을 반환하며, 이후 해당 채널은 confirm 모드로 설정됩니다 ([https://stackoverflow.com/questions/58902366/how-does-rabbitmq-publisher-confirms-work](https://stackoverflow.com/questions/58902366/how-does-rabbitmq-publisher-confirms-work))
* 참고: [https://www.rabbitmq.com/tutorials/tutorial-seven-php](https://www.rabbitmq.com/tutorials/tutorial-seven-php),&#x20;

**2-1-2. Publisher Returns:**

* 브로커가 메시지를 라우팅할 수 없을 때 발생합니다.
* **Publisher** 가 메시지를 브로커로 전송한 후 브로커가 메시지를 라우팅할 수 없으면 **Publisher** 는 해당 메시지를 처리하고 실패한 메시지를 다른 곳으로 보낼 수 있습니다

### **2-2. Consumer Acknowledgment Model (소비자 확인 모델)**

메시지를 처리한 컨슈머가 해당 메시지를 성공적으로 받았음을 확인하는 메커니즘으로 데이터 안전성과 신뢰성을 보장하기 위해 중요한 역할로Consumer가 메시지를 처리했음을 명시적으로 확인하기 위해 사용되며 Consumer가 처리한 메시지를 표시하고, Consumer가 처리 중에 실패하더라도 메시지가 손실되지 않도록 보장합니다. (확인인 동일한 채널에서 수행)

* 이 모델에서는 Consumer가 메시지를 받으면 브로커에게 통지합니다.
* 브로커는 해당 메시지를 Queue에서 삭제하기 전에 Consumer로부터 통지를 받아야 합니다.
* Consumer가 메시지를 처리했을 때만 브로커는 해당 메시지를 삭제하며, 처리하지 못한 경우 메시지는 다시 Queue에 남아 있습니다.
* 이 모델은 메시지 손실을 최소화하고, Consumer가 메시지를 안전하게 처리할 수 있도록 합니다.
* 참고: [https://www.rabbitmq.com/docs/confirms](https://www.rabbitmq.com/docs/confirms)

<figure><img src="../../.gitbook/assets/image (298).png" alt=""><figcaption></figcaption></figure>

**2-2-1. Consumer Acknowledgements (소비자 확인)**:

* 컨슈머가 메시지를 처리했음을 명시적으로 확인하기 위해 사용됩니다.
* RabbitMQ에서는 다음과 같은 세 가지 유형의 Consumer Acknowledgements을 지원합니다.
  * basic.ack: 메시지를 성공적으로 처리한 경우에 사용.
  * basic.nack: Consumer가 메시지 처리를 실패했음을 RabbitMQ에 알리는 방법
    * 메시지 처리를 실패했거나 다시 처리해야 하는 경우에 사용
    * Consumer가 메시지를 처리하는 도중 예외가 발생했을 때 사용
  * basic.reject:  basic.nack:와 유사하지만, 단일 메시지에 대해서만 적용됩니다.&#x20;
    * 즉, 하나의 메시지를 Nack하고 다시 Queue로 되돌리거나 삭제할 수 있습니다
    * 특정 메시지를 처리하지 않고 다시 큐로 되돌리는 경우에 사용
* 참고:  [https://stackoverflow.com/questions/44417191/spring-cloud-stream-rabbit-delivery-acknowledgment](https://stackoverflow.com/questions/44417191/spring-cloud-stream-rabbit-delivery-acknowledgment)

**2-2-2. 자동 커밋 (Automatic Acknowledgement: autoAck=true)**:

* 기본적으로 RabbitMQ Consumer는 자동 커밋 모드로 동작합니다.
* RabbitMQ가 메시지를 한 번 읽은 후 해당 메시지를 삭제합니다
* 즉, Consumer가 메시지를 처리한 후에 RabbitMQ가 자동으로 Acknowledgement를 수행 합니다.
* 메시지가 자동으로 확인되고 삭제되며, 예외 발생 시 메시지가 완전히 손실될 수 있습니다. &#x20;
* 메시지를 한 번 읽고 나서는 다시 읽을 수 없게 됩니다. 즉, 메시지를 처리한 후에 RabbitMQ가 해당 메시지를 삭제합니다.&#x20;

**2-2-3. 수동 커밋 (Manual Acknowledgement: autoAck=false)**

* Consumer가 메시지를 처리한 후에 명시적으로 Acknowledgement를 보내야 합니다.
* 메시지를 처리한 후에도 RabbitMQ에서 해당 메시지를 유지하게 됩니다.
* 이는 메시지를 처리한 후에도 다른 Consumer가 해당 메시지를 읽을 수 있도록 합니다.
* 메시지는 수신되지만 수동으로 확인되어야 하며, 메시지 손실을 방지할 수 있습니다. &#x20;
* 메시지를 읽되 남겨 놓게 됩니다. 이 경우 메시지를 처리한 후에도 RabbitMQ에서 해당 메시지를 삭제하지 않습니다

#### 2-2-4. autoAck 설정 예시&#x20;

* **예시:** 여러 개의 Consumer가 하나의 Queue를 바라보고 있고, 메시지를 Round-Robin 방식으로 분배받는 상황이면
  * **autoAck: true로 설정**: . 메시지를 처리한 후에 RabbitMQ가 해당 메시지를 삭제하므로 다른 Consumer가 해당 메시지를 다시 읽을 수 없습니다.
  * **autoAck: false로 설정:** 메시지를 처리한 후에도 RabbitMQ에서 해당 메시지를 유지하게 되므로, 예기치 않은 종료나 예외 상황에서 메시지가 손실되지 않도록 할 수 있습니다.

