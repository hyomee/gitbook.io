---
description: >-
  Apache Kafka는 고성능 데이터 파이프 라인, 스트리밍 분석, 데이터 통합 ​​및 미션 크리티컬 애플리케이션을 위해 사용하는 오픈
  소스 분산 이벤트 스트리밍 플랫폼
---

# 개요

Kafka는 대량의 데이터를 높은 처리량과 실시간 처리를 위한 오픈 소스로 다음과 특징이 있이 있습니다.

* **확장성:** 데이터 양에 따라서 시스템 확장이 가능합니다.
* **연속성** 수신 데이터를 디스크에 저장 하기 때문에 언제라도 데이터를 읽을 수 있습니다.
* **유연성**: 다른 제품이나 시스템을 연결 하는 허브 역할을 합니다.
  * Connect API -> Kafka Connect 제공&#x20;
  * Streams API -> Kafka Streams 제공&#x20;
*   **신뢰성**: 메시지 전달 보증으로 데이터 상실은 허용 하지 않습니다.\


    <table data-header-hidden><thead><tr><th width="166"></th><th width="168"></th><th width="74"></th><th width="69"></th><th></th></tr></thead><tbody><tr><td>종류</td><td>개요</td><td>재전송</td><td>중복삭제</td><td>비고</td></tr><tr><td>At Most Once</td><td>1회는 전달 시도</td><td>X</td><td>X</td><td>메시지 중복 없음, 상실 있음</td></tr><tr><td>At Least Once</td><td>적어도 1회는 전달</td><td>O</td><td>X</td><td><p>메시지 중복 가능, 상실 없음</p><p>Ack, Offset Commit</p></td></tr><tr><td>Exactly Once</td><td>1회만 전달</td><td>O</td><td>O</td><td><p>메시지 중복 없음, 상실 없음, 성능 저하</p><p>Ack, Offset Commit</p><p>트랜잭션 Abort/Timeout</p></td></tr></tbody></table>

## 1. Kafka 메세지 모델

* **Queuing Model**:  여러 Consumer가 분산 처리로 메시지를 소비하는 Queuing Model&#x20;
* **Pub/Sub Model**: 여러 Subscriber에 동일한 메시지를 전달 하고, Topic 기반으로 전달 내용을 변경하는 Pub/Sub Model -> Consumer Group 개념 도입

## 2. Kafka 구성 요







