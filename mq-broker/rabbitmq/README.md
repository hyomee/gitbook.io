# RabbitMQ

RabbitMQ는 AMQP(Advanced Message Queuing Protocol)를 구현한 오픈소스 메시지 브로커로 생산자(Producer)가 메시지를 보내면 소비자(Consumer)에게 전달해주는 역할을 합니다.

참고: [https://www.rabbitmq.com/](https://www.rabbitmq.com/), [https://www.rabbitmq.com/tutorials](https://www.rabbitmq.com/tutorials)

## 1. RabbitMQ의 주요 개념

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

* **Producer**: 메시지를 보내는 주체로, 보내고자 하는 메시지를 Exchange에 publish합니다.
* **Consumer**: Producer로부터 메시지를 받아 처리하는 주체입니다.
* **Exchange**: Producer로부터 전달받은 메시지를 어떤 Queue로 보낼지 결정하는 장소로 다양한 Exchange 타입이 있으며, 라우팅 키와 규칙에 따라 적절한 Queue로 메시지를 전달합니다.
* **Queue**: Consumer가 메시지를 소비하기 전까지 보관하는 장소입니다.
* **Binding Key**: Exchange와 Queue의 관계를 정의하며, 특정 Exchange가 특정 Queue를 binding하도록 설정합니다.
* **Routing Key:** 게시자는 메시지를 게시할 때마다 메시지와 함께 라우팅 키도 지정합니다.

## 2. RabbitMQ 동작 원리

게시자가 메시지를 게시하면 먼저 교환에서 메시지를 받습니다. 그런 다음 교환은 교환 유형에 따라 메시지를 큐로 전달합니다.

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

* **Fan-out Exchange:** Exchange에 연결되 모든 queue에 메세지를 전달 합니다.
* **Direct Exchange:** Binding Key가 게시자가 지정한 Routing Key와 정확히 동일한 Exchange에 연결된 Queue로 전달합니다.
* **Topic Exchange:** Binding Key와 Routing Key가 부분 일치를 기반으로 메세지 전달합니다.
* **Header Exchange:** AMQP 메시지의 구조를 활용하며 AMQP 메시지의 헤더(사용자 지정 헤더 포함)를 기반으로 복잡한 라우팅을 수행할 수 있습니다. AMQP를 통해 전송된 각 메시지에는 헤더라는 메타데이터가 첨부되어 있습니다.

