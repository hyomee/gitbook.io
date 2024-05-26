# AMQP 송수신 확인 모델

<figure><img src="../../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

1. **Producer (생산자 or Publisher)**: 메시지를 발행하는 주체로. Producer는 보내고자 하는 메시지를 Exchange에 publish합니다.
2. **Exchange (교환기)**: Producer로부터 전달받은 메시지를 어떤 Queue로 보낼지 결정하는 장소로  RabbitMQ에서는 다음과 같은 Exchange Type을 지원합니다:
   * **Direct Exchange**: 메시지에 포함된 라우팅 키(routing key)를 기반으로 Queue로 메시지를 전달합니다. 각 Queue는 특정 라우팅 키와 연결됩니다.
   * **Fanout Exchange**: 라우팅 키와 관계없이 연결된 모든 Queue에 동일한 메시지를 전달합니다.
   * **Topic Exchange**: 라우팅 키가 일치하는 Queue로 메시지를 전달합니다. 라우팅 키는 점(.)으로 구분된 단어를 조합해서 정의하며, 와일드 카드(\*)와(#)을 사용하여 패턴을 지정할 수 있습니다.
   * **Headers Exchange**: 메시지 헤더를 통해 binding key를 사용하는 것보다 더 다양한 속성을 사용할 수 있습니다. 헤더 값이 바인딩 시 지정된 값과 같은 경우에만 일치하는 것으로 간주합니다.
3. **Binding (바인딩)**: Exchange와 Queue의 관계를 정의합니다. 보통 사용자가 특정 Exchange가 특정 Queue를 binding하도록 설정합니다. (단, fanout 타입은 예외입니다.)
4. **Queue (큐)**: Consumer가 메시지를 consume하기 전까지 보관하는 장소입니다. Queue는 반드시 미리 정의되어야 하며, 이름, 내구성, 자동 삭제 여부 등의 속성을 갖습니다.
5. **Consumer (소비자)**: Producer로부터 메시지를 받아 처리하는 주체입니다. Consumer는 Queue를 통해 메시지를 가져갑니다.

AMQP는 네트워크에 문제가 발생하거나 요청을 처리하지 못했을 경우를 대비해 2가지 수신 확인 모델을 가지고 있습니다:

* [**Consumer Acknowledgment Model**: Consumer가 메시지를 받으면 브로커에게 통지하고, 브로커는 통지를 받았을 때만 Queue에서 해당 메시지를 삭제합니다](https://jonnung.dev/rabbitmq/2019/02/06/about-amqp-implementtation-of-rabbitmq/)
