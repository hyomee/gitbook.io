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

## 2. Kafka 구성 요소

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption><p>Kafka 개념도</p></figcaption></figure>



* Broker: 데이터를 수신, 전달(Consumer의 요구에 따라 응답)&#x20;
* Message: 데이터의 최소 단위, key/value 구조, 전송 시 Partition 이용&#x20;
* Producer: 데이터 생산자, broker에 Message 전달&#x20;
  * 레코드를 프로듀스할 때 어느 토픽의 어느 파티션에 할당할 지를 결정한다&#x20;
* Consumer: 메시지 얻음&#x20;
* Topic: 메시지 종류별로 Broker에서 관리&#x20;
  * 카프카 안에는 여러 레코드 스트림이 있을 수 있다.&#x20;
  * 하나의 토픽에 대해 여러 Subscriber가 붙을 수 있음

## 3. Kafka 시스템 구성

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Kafka 구성</p></figcaption></figure>





* **API**: Producer, Consumer개발을 위한 API&#x20;
* **ZooKeeper**: 분산 처리를 위한 관리 도구&#x20;
  * 분산 메시징의 메타 데이터 ( Topic, Partition .. )를 관리&#x20;
  * 카프카 클러스터의 리더(Leader)를 발탁하는 방식도 주키퍼가 제공하는 기능&#x20;
* **Kafka Admin**: Kafka 관리&#x20;
* **Kafka Cluster 구성 방법**
  1. **수동 설치 및 설정**:
     * **Zookeeper 설치**: Kafka는 Zookeeper와 함께 실행되므로 먼저 Zookeeper를 설치해야 합니다. Zookeeper는 Kafka의 메타데이터와 상태 정보를 관리합니다.
     * **Kafka 설치**: Kafka를 다운로드하고 각 노드에 설치합니다. 설치된 Kafka 노드는 Zookeeper와 연결됩니다.
     * **Kafka Broker 설정**: 각 Kafka 브로커의 `server.properties` 파일을 수정하여 브로커 ID, 포트, 로그 디렉토리, Zookeeper 연결 정보 등을 설정합니다.
     * **Kafka 브로커 실행**: Zookeeper와 Kafka 브로커를 실행합니다.
  2. **Docker Compose를 사용한 설치**:
     * Docker Compose를 사용하면 간편하게 Kafka Cluster를 구성할 수 있으며. Docker Compose 파일에 Zookeeper와 Kafka 노드를 정의하고 실행하면 됩니다

## 4. 5. Kafka 활용 사례

* **IoT (사물 인터넷)**: 초당 수백만 개의 데이터 포인트를 처리할 수 있어 대규모 데이터를 다루는 IoT 환경에 적합합니다.
* **전자상거래**: 웹 사이트 활동 트래킹, 주문, 장바구니, 재고 등 다양한 데이터를 처리할 수 있습니다.
* **IT 운영**: 모니터링, 로그 관리, 데이터 수집 등 IT 운영팀의 업무에 활용됩니다.
