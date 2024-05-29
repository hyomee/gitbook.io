# AMQP

AMQP(Advanced Message Queuing Protocol)는 메시지 지향 미들웨어를 위한 개방형 표준 응용 계층 프로토콜입니다. 이 프로토콜은 메시지 지향, 큐잉, 라우팅(P2P 및 발행-구독), 신뢰성, 보안 기능을 제공하며 대표적인 소프트웨어로 RabbitMQ, SwiftMQ 등이 있습니다.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>AMQP 구조</p></figcaption></figure>

* 생산자, 메시지 브로커 및 소비자 간의 상호 운용성을 만듭니다.
* 자/게시자는 메시지를 만들어 Exchange로 보냅니다. 그런 다음 Exchange는 관련 바인딩에 따라 메시지를 하나 이상의 큐로 라우팅합니다.&#x20;

참고: [위키백과\_AMQP](https://ko.wikipedia.org/wiki/AMQP)
