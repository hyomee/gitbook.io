# 개념

RabbitMQ는 AMQP(Advanced Message Queuing Protocol)를 구현한 오픈소스 메시지 브로커로 생산자(Producer)가 메시지를 보내면 소비자(Consumer)에게 전달해주는 역할을 하는 메시지 큐로 MPL 1.1에 따라 라이선스가 부여된 오픈 소스 메시지 브로커입니다. Pivotal software Inc(현재 VMware 소유)의 일부로  거의 99.999%의 가동 시간을 제공하는 분산되고 내결함성이 있는 소프트 실시간 시스템을 위해 설계된 Erlang으로 작성되었습니다.

<details>

<summary>얼랭 (Erlang)</summary>

범용 병렬 프로그래밍 언어로 에릭슨(Ericsson)사에서 스위칭 소프트웨어에서 사용하기 위해 개발되었지만, 1998년에 오픈 소스로 공개되었습니다. 흔히 Erlang이라는 이름이 "Ericsson Language"에서 따온 것이라고 생각하지만, 실제로는 통신 이론을 연구한 덴마크의 수학자 Agner Krarup Erlang의 이름에서 따온 것입니다.

얼랭은 다음과 같은 특징을 가지고 있습니다:

* **병렬 프로그래밍**: Erlang은 병렬 처리를 위해 설계되었습니다. 특히 통신 시스템, 은행, 전자 상거래, 컴퓨터 전화 및 즉시 메시징과 같은 실시간 시스템에서 사용됩니다.
* **함수형 프로그래밍**: Erlang은 함수형 프로그래밍 언어로, 함수를 중심으로 프로그래밍합니다. 이로써 코드의 가독성과 유지 보수성이 향상됩니다.
* **동시성, 분산 및 내결함성 지원**: Erlang의 런타임 시스템은 동시성, 분산 처리 및 내결함성을 내장하고 있습니다. 이를 통해 안정적이고 확장 가능한 시스템을 구축할 수 있습니다.
* **메세지 전달 방식:** 프로세스 간 통신은 메시지 패싱(Message Passing)을 통해 이루어 집니다.
  1. **프로세스(Process)**:
     * Erlang은 가볍고 독립적인 프로세스를 생성하는 데 강점이 있습니다. 이러한 프로세스는 운영체제의 프로세스가 아닌 Erlang 가상 머신(VM) 내에서 실행됩니다.
     * 각 프로세스는 고유한 프로세스 ID(PID)를 가지며, 메모리를 공유하지 않습니다. 따라서 하나의 프로세스가 실패하더라도 다른 프로세스에 영향을 주지 않습니다.
  2. **메시지 패싱**:
     * 프로세스 간 통신은 메시지를 보내고 받는 방식으로 이루어집니다.
     * 한 프로세스는 다른 프로세스에게 메시지를 보낼 수 있습니다. 메시지는 특정 프로세스의 메일박스에 전달됩니다.
     * 메시지를 받은 프로세스는 메시지를 처리하거나 다른 프로세스에게 전달할 수 있습니다.
  3. **프로세스 간 통신의 장점**:
     * **경량성**: Erlang 프로세스는 가볍고 빠르게 생성되며, 많은 수의 프로세스를 동시에 실행할 수 있습니다.
     * **내결함성**: 프로세스 간 통신을 통해 하나의 프로세스가 실패하더라도 다른 프로세스가 계속 작동할 수 있습니다.
     * **분산 시스템 지원**: Erlang은 분산 시스템을 구축하기 위한 강력한 메커니즘을 제공합니다.

</details>

<figure><img src="../../.gitbook/assets/image (308).png" alt=""><figcaption><p>얼랭으로 구현된 RabbitMQ</p></figcaption></figure>

<details>

<summary>실시간시스템(Real-Time System)</summary>

요구사항에 맞퉈 특정한 이벤트를 전달 받았을 때 반드시 응답을 반환하는 하드웨어 플랫폼 또는 소프트웨어 플랫폼이나 두가지를 조합한것을 말합니다.

</details>

## 1. RabbitMQ 장점

1. **신뢰할 수 있는 메시징(Reliable Messaging)**: 시스템의 일부가 실패하더라도 메시지가 손실 없이 전달되도록 합니다.
2. **유연한 라우팅(Flexible Routing)**: 다양한 방식으로 메시지를 라우팅할 수 있으므로 다양한 요구 사항에 맞게 조정할 수 있습니다.
3. **확장성(Scalable)**: 많은 리소스를 추가하여 증가하는 메시지 및 사용자 수를 처리할 수 있습니다.
4. **여러 프로토콜 지원**:  AMQP, STOMP, MQTT, HTTP, 웹소켓, SMTP와 같은 다양한 메시징 프로토콜지원 합니다.
   * **AMQP (Advanced Message Queuing Protocol):** RabbitMQ에서 사용하는 기본 프로토콜로 신뢰할 수 있는 배달, 라우팅 및 보안과 같은 강력한 메시징 기능을 제공합니다
   * **AMQP 1.0:** 서로 다른 시스템에서 상호 운용이 가능한 다른 버전의 AMQP로 다양한 AMQP 1.0 클라이언트와의 호환성을 보장합니다.
   * **STOMP (Simple (or Streaming) Text Oriented Messaging Protocol):** 간단하고 구현하기 쉬운 텍스트 기반 프로토콜로 웹 기반 메시징 시스템 및 응용 프로그램과의 통합에 자주 사용됩니다.
   * **MQTT (Message Queuing Telemetry Transport):** 경량 메시징 프로토콜로 IoT(사물 인터넷) 장치 및 모바일 애플리케이션과 같은 제한된 환경에 이상적입니다
   * **HTTP:** 메시징을 위한 RESTful HTTP API를 지원하기 위해 플러그인을 사용할 수 있습니다
   * **WebSockets:** 수명이 긴 단일 TCP 연결을 통해 전이중 통신 채널을 제공하여 실시간 웹 응용 프로그램에 유용하며 웹 브라우저에서 메시지를 보내고 받을 수 있습니다.
   * **SMTP (Simple Mail Transfer Protocol):** 플러그인 또는 사용자 지정 통합을 통해 이메일 시스템과 상호 작용할 수 있습니다
5. **사용하기 쉬움**: 사용자 친화적인 인터페이스와 다양한 기술문서가 있어 개발자가 쉽게 코드를 작성할 수 있습니다.

## 2. RabbitMQ 기능 및 이점

* **오픈 소스** – Mozilla Public License 1.1에 따라 출시되었습니다.
* **다중 메시지 프로토콜** – AMQP, MQTT, STOMP, HTTP.
* **경량–** 단일 인스턴스는 40MB 미만의 RAM에서 실행할 수 있습니다.
* **클라이언트 라이브러리 지원** – Java, Python, JavaScript, Erlang 등과 같은 모든 최신 프로그래밍 언어에는 RabbitMQ 클라이언트 라이브러리가 있습니다.
* **타사 플러그인 지원** – 타사 플러그인을 위한 유연한 플러그인 시스템을 제공합니다. &#x20;
* **확장성이 뛰어난 아키텍처** – RabbitMQ의 클러스터를 쉽게 배포할 수 있습니다.
* **엔터프라이즈 및 클라우드 지원** – 온프레미스 인프라 또는 클라우드 인프라에 배포할 수 있습니다.
* **관리 및 모니터링** – HTTP-API, 명령줄 도구 및 관리 및 모니터링을 위한 UI.
* **도구 지원** – 주요 CI/CD 도구와 함께 작동하며 BOSH, Chef, Docker 및 Puppet과 함께 배포할 수 있습니다.

## 3. RabbitMQ의 주요 개념

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* **Producer**: 메시지를 보내는 주체로, 보내고자 하는 메시지를 Exchange에 publish합니다.
* **Consumer**: Producer로부터 메시지를 받아 처리하는 주체입니다.
* **Exchange**: Producer로부터 전달받은 메시지를 어떤 Queue로 보낼지 결정하는 장소로 다양한 Exchange 타입이 있으며, 라우팅 키와 규칙에 따라 적절한 Queue로 메시지를 전달합니다.
* **Queue**: Consumer가 메시지를 소비하기 전까지 보관하는 장소입니다.
* **Binding Key**: Exchange와 Queue의 관계를 정의하며, 특정 Exchange가 특정 Queue를 binding하도록 설정합니다.
* **Routing Key:** 게시자는 메시지를 게시할 때마다 메시지와 함께 라우팅 키도 지정합니다.

## 4. RabbitMQ 동작 원리

게시자가 메시지를 게시하면 먼저 교환에서 메시지를 받습니다. 그런 다음 교환은 교환 유형에 따라 메시지를 큐로 전달합니다.

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* **Producer (생산자 or Publisher)**: 메시지를 발행하는 주체로. Producer는 보내고자 하는 메시지를 Exchange에 publish합니다.
*   **Exchange (교환기)**: Producer로부터 전달받은 메시지를 어떤 Queue로 보낼지 결정하는 장소로  RabbitMQ에서는 다음과 같은 Exchange Type을 지원합니다.

    <figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

## 5. RabbitMQ에서RPC&#x20;

**RabbitMQ는 AMQP 메시지 브로커로서, 코어 서버 통신의 거의 모든 부분에서 RPC 패턴을 사용하여 통신합니다**. 따라서 RPC에 대해 알아보는 것이 중요합니다.

RPC (원격 프로시저 호출)는 프로세스 간 통신을 위한 방법 중 하나입니다. 이 기술은 별도의 원격 제어 코딩 없이도 다른 주소 공간에 있는 함수나 프로시저를 실행할 수 있게 해줍니다. 분산 컴퓨팅 환경에서 자원을 효율적으로 사용하기 위해 개발되었으며, 프로세스는 원칙적으로 자신의 주소 공간 내에 있는 함수만을 호출하여 실행할 수 있습니다.

<figure><img src="../../.gitbook/assets/image (312).png" alt="" width="563"><figcaption><p>RPC</p></figcaption></figure>

* RPC의 장점
  * 고유 프로세스 개발 집중 가능 (하부 네트워크 프로토콜에 신경쓰지 않아도 되기 때문)
  * 프로세스간 통신 기능을 비교적 쉽게 구현하고 정교한 제어가 가능
* RPC의 단점
  * 호출 실행과 반환 시간이 보장되지 않음 (네트워크 구간을 통하여 RPC 통신을 하는 경우, 네트워크가 끊겼을 때 치명적 문제 발생)
  * 보안이 보장되지 않음
* RPC의 대표적인 구현체는?
  * [`ProtocolBuffer` by `Google`](https://developers.google.com/protocol-buffers/)
  * [`Thrift` by `Facebook`](https://thrift.apache.org/)
  * [`Finalge` by `Twitter`](https://twitter.github.io/finagle/)

## 5. RabbitMQ 흐름

<figure><img src="../../.gitbook/assets/image (313).png" alt=""><figcaption></figcaption></figure>



## 6. Binding

Exchange와 Queue를 연결하는 관계로 Exchange 타입과 binding 규칙에 따라 적절한 Queue로 전달됩니다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* API 참고: [https://www.rabbitmq.com/client-libraries/java-client](https://www.rabbitmq.com/client-libraries/java-client)

Exchange는메세지를 받고 받은 매새지를 큐로 전달하는 요소로 Exchange가 어떤 Queue로 메시지를 전달하는지 결정하는 라우팅 알고리즘은 Exchange Type과 Binding 규칙에 의해 결정됩니다 즉 Exchange와 Queue를 적절하게 설정하여 메시지를 효율적으로 라우팅할 수 있습니다.

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

Exchage는 관리 UI 또는 프로그램 방식을 통해서 만들 수 있습니다.&#x20;

Add a new queue 영역에서 큐 정보를 입력 후 생성 합니다.

큐 생성 및 삭제는 다음 메서드를 사용 합니다.

exchange와 queue 에 대한 코드를 작성하였습니다. 즉 메세지는 큐로 직접 전달이 되지 않는 라우팅 설정 입니다. Exchange는 RabbitMQ 서버 내의 가상 호스트(vhost)에 상주하는 메시지 라우팅 에이전트로 메세지를 큐로 보내기 위해서는 바인딩 규칙을 설정 해야 합니다.



참고: [https://www.rabbitmq.com/](https://www.rabbitmq.com/), [https://www.rabbitmq.com/tutorials](https://www.rabbitmq.com/tutorials)

참고: [www.rabbitmq.com/getstarted.html](http://www.rabbitmq.com/getstarted.html)

