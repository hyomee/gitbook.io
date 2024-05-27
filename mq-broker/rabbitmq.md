# RabbitMQ

RabbitMQ는 AMQP(Advanced Message Queuing Protocol)를 구현한 오픈소스 메시지 브로커로 생산자(Producer)가 메시지를 보내면 소비자(Consumer)에게 전달해주는 역할을 합니다.

참고: [https://www.rabbitmq.com/](https://www.rabbitmq.com/), [https://www.rabbitmq.com/tutorials](https://www.rabbitmq.com/tutorials)

## 1. RabbitMQ의 주요 개념

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

* **Producer**: 메시지를 보내는 주체로, 보내고자 하는 메시지를 Exchange에 publish합니다.
* **Consumer**: Producer로부터 메시지를 받아 처리하는 주체입니다.
* **Exchange**: Producer로부터 전달받은 메시지를 어떤 Queue로 보낼지 결정하는 장소로 다양한 Exchange 타입이 있으며, 라우팅 키와 규칙에 따라 적절한 Queue로 메시지를 전달합니다.
* **Queue**: Consumer가 메시지를 소비하기 전까지 보관하는 장소입니다.
* **Binding Key**: Exchange와 Queue의 관계를 정의하며, 특정 Exchange가 특정 Queue를 binding하도록 설정합니다.
* **Routing Key:** 게시자는 메시지를 게시할 때마다 메시지와 함께 라우팅 키도 지정합니다.

## 2. RabbitMQ 동작 원리

게시자가 메시지를 게시하면 먼저 교환에서 메시지를 받습니다. 그런 다음 교환은 교환 유형에 따라 메시지를 큐로 전달합니다.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

* **Producer (생산자 or Publisher)**: 메시지를 발행하는 주체로. Producer는 보내고자 하는 메시지를 Exchange에 publish합니다.
*   **Exchange (교환기)**: Producer로부터 전달받은 메시지를 어떤 Queue로 보낼지 결정하는 장소로  RabbitMQ에서는 다음과 같은 Exchange Type을 지원합니다.

    <figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

    * **Direct Exchange**: 메시지에 포함된 라우팅 키(routing key)를 기반으로 Queue로 메시지를 전달합니다. 각 Queue는 특정 라우팅 키와 연결됩니다.&#x20;
      * Binding Key가 게시자가 지정한 Routing Key와 정확히 동일한 Exchange에 연결된 Queue로 전달
    * **Fanout Exchange**: 라우팅 키와 관계없이 연결된 모든 Queue에 동일한 메시지를 전달합니다.
    * **Topic Exchange**: 라우팅 키가 일치하는 Queue로 메시지를 전달합니다. 라우팅 키는 점(.)으로 구분된 단어를 조합해서 정의하며, 와일드 카드(\*)와(#)을 사용하여 패턴을 지정할 수 있습니다.
      * Binding Key와 Routing Key가 부분 일치를 기반으로 메세지 전달)
    * **Headers Exchange**: 메시지 헤더를 통해 binding key를 사용하는 것보다 더 다양한 속성을 사용할 수 있습니다. 헤더 값이 바인딩 시 지정된 값과 같은 경우에만 일치하는 것으로 간주합니다.&#x20;
      * AMQP 메시지의 구조를 활용하며 AMQP 메시지의 헤더(사용자 지정 헤더 포함)를 기반으로 복잡한 라우팅을 수행할 수 있습니다. AMQP를 통해 전송된 각 메시지에는 헤더라는 메타데이터가 첨부되어 있습니다.\

* **Binding (바인딩)**: Exchange와 Queue의 관계를 정의합니다. 보통 사용자가 특정 Exchange가 특정 Queue를 binding하도록 설정합니다. (단, fanout 타입은 예외입니다.)
* **Queue (큐)**: Consumer가 메시지를 consume하기 전까지 보관하는 장소입니다. Queue는 반드시 미리 정의되어야 하며, 이름, 내구성, 자동 삭제 여부 등의 속성을 갖습니다.
* **Consumer (소비자)**: Producer로부터 메시지를 받아 처리하는 주체입니다. Consumer는 Queue를 통해 메시지를 가져갑니다.

