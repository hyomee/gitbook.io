---
description: >-
  Apache Kafka는 고성능 데이터 파이프 라인, 스트리밍 분석, 데이터 통합 ​​및 미션 크리티컬 애플리케이션을 위해 사용하는 오픈
  소스 분산 이벤트 스트리밍 플랫폼으로 시스템과 시스템을 연결하는 역활을 합니다
---

# Kafka

<figure><img src="../../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

## 1. 이벤트 스트리밍&#x20;

ㅇ

## 1. Kafka 특징

Kafka는 대량의 데이터를 높은 처리량과 실시간 처리를 위한 오픈 소스로 다음과 특징이 있이 있습니다.

* **확장성:** 데이터 양에 따라서 시스템 확장이 가능합니다.
* **연속성** 수신 데이터를 디스크에 저장 하기 때문에 언제라도 데이터를 읽을 수 있습니다.
* **유연성**: 다른 제품이나 시스템을 연결 하는 허브 역할을 합니다.
  * Connect API -> Kafka Connect 제공&#x20;
  * Streams API -> Kafka Streams 제공&#x20;
*   **신뢰성**: 메시지 전달 보증으로 데이터 상실은 허용 하지 않습니다.\


    <table data-header-hidden><thead><tr><th width="166"></th><th width="168"></th><th width="74"></th><th width="69"></th><th></th></tr></thead><tbody><tr><td>종류</td><td>개요</td><td>재전송</td><td>중복삭제</td><td>비고</td></tr><tr><td>At Most Once</td><td>1회는 전달 시도</td><td>X</td><td>X</td><td>메시지 중복 없음, 상실 있음</td></tr><tr><td>At Least Once</td><td>적어도 1회는 전달</td><td>O</td><td>X</td><td><p>메시지 중복 가능, 상실 없음</p><p>Ack, Offset Commit</p></td></tr><tr><td>Exactly Once</td><td>1회만 전달</td><td>O</td><td>O</td><td><p>메시지 중복 없음, 상실 없음, 성능 저하</p><p>Ack, Offset Commit</p><p>트랜잭션 Abort/Timeout</p></td></tr></tbody></table>

## 5. Kafka 활용 사례

* **IoT (사물 인터넷)**: 초당 수백만 개의 데이터 포인트를 처리할 수 있어 대규모 데이터를 다루는 IoT 환경에 적합합니다.
* **전자상거래**: 웹 사이트 활동 트래킹, 주문, 장바구니, 재고 등 다양한 데이터를 처리할 수 있습니다.
* **IT 운영**: 모니터링, 로그 관리, 데이터 수집 등 IT 운영팀의 업무에 활용됩니다.







