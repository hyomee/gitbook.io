---
description: 'Exchange Type: Direct'
---

# Binding

Exchange와 Queue를 연결하는 관계로 Exchange 타입과 binding 규칙에 따라 적절한 Queue로 전달됩니다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

Exchage는 관리 UI 또는 프로그램 방식을 통해서 만들 수 있습니다.&#x20;

Add a new queue 영역에서 큐 정보를 입력 후 생성 합니다.

큐 생성 및 삭제는 다음 메서드를 사용 합니다.

exchange와 queue 에 대한 코드를 작성하였습니다. 즉 메세지는 큐로 직접 전달이 되지 않는 라우팅 설정 입니다. Exchange는 RabbitMQ 서버 내의 가상 호스트(vhost)에 상주하는 메시지 라우팅 에이전트로 메세지를 큐로 보내기 위해서는 바인딩 규칙을 설정 해야 합니다.

