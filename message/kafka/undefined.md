---
description: >-
  Apache Kafka는 고성능 데이터 파이프 라인, 스트리밍 분석, 데이터 통합 ​​및 미션 크리티컬 애플리케이션을 위해 사용하는 오픈
  소스 분산 이벤트 스트리밍 플랫폼
---

# 개요

## 1. Kafka 메세지 모델

* **Queuing Model**:  여러 Consumer가 분산 처리로 메시지를 소비하는 Queuing Model&#x20;
* **Pub/Sub Model**: 여러 Subscriber에 동일한 메시지를 전달 하고, Topic 기반으로 전달 내용을 변경하는 Pub/Sub Model -> Consumer Group 개념 도입

## 2. Kafka 구성 요소

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p>Kafka 개념도</p></figcaption></figure>



* Broker: 데이터를 수신, 전달(Consumer의 요구에 따라 응답)&#x20;
* Message: 데이터의 최소 단위, key/value 구조, 전송 시 Partition 이용&#x20;
* Producer: 데이터 생산자, broker에 Message 전달&#x20;
  * 레코드를 프로듀스할 때 어느 토픽의 어느 파티션에 할당할 지를 결정한다&#x20;
* Consumer: 메시지 가지고 온다.
* Topic: 메시지 종류별로 Broker에서 관리&#x20;
  * 카프카 안에는 여러 레코드 스트림이 있을 수 있다.&#x20;
  * 하나의 토픽에 대해 여러 Subscriber가 붙을 수 있음

## 3. Kafka 시스템 구성

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>Kafka 구성</p></figcaption></figure>





* **API**: Producer, Consumer개발을 위한 API&#x20;
* **ZooKeeper**: 분산 처리를 위한 관리 도구&#x20;
  * 분산 메시징의 메타 데이터 (Topic, Partition .. )를 관리&#x20;
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

## 4. 분산 메세징 구조

### 4-1. 논리적 구조

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>카프카 분산 메세지 구조</p></figcaption></figure>

* **Topic**: 카프카 클러스터에서 여러개 만들 수 있으며 하나의 토픽은 n개 이상의 파티션(Partition)으로 구성되어 있습니다다.
  * 물리적으로 각 토픽은 각각의 토픽에 대해 하나 이상의 파티션을 소유하는 다른 카프카 브로커에게 전달합니다.
* **Partition**: 각 토픽 당 데이터를 분산 처리하는 단위&#x20;
  * Offset: 파티션 단위로 메시지 위치를 나타냄&#x20;
    * Log-End-Offset(LEO): 파티션 데이터의 끝&#x20;
    * Current Offset: 컨슈머가 어디까지 메시지를 읽은 위치 (Consumer Group별)&#x20;
    * Commit Offset: 컨슈머가 어디까지 커밋 했는지를 나타냄 (Consumer Group 별)
* **Consumer Group**: 단일 애플리케이션 안에서 여러 컨슈머가 단일 토픽이나 여러 파티션에서 메시지를 취득 하는 방법&#x20;
  * 컨슈머 그룹 마다 독립적인 컨슘 오프셋을 가진다.&#x20;
  * 컨슈머 그룹 내에서 처리해야할 파티션이 분배된다. 즉 하나의 파티션은 하나의 서버가 처리하고. 그룹에 서버가 추가되면 카프카 프로토콜에 의해 동적으로 파티션이 재분배 된다.&#x20;
  * 하나의 토픽 레코드를 분산 처리하는 구조라면 동일 컨슈머 그룹을 가지게 해야 한다.&#x20;
  * 하나의 토픽 레코드에 각각 별도의 처리를 하는 다른 파이프라인이라면 서로 다른 컨슈머 그룹을 가지게 해야 한다.



### 4-2. 물리적 구조

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption><p>카프카 물리적 구조</p></figcaption></figure>

카프카 클러스터는 다중 브로커로 구성이 되며 클러스터에 대한 쓰기/읽기 작업의 부하 분산을 도와주고 있으며 각 브로커의 상태는 주키퍼를 사용 합니다.

#### 4-2-1. 리더와 팔로워

카프카는 장애 대응을 위해 메인 브로커가 다운되더라도 리플리케이션되고 있는 브로커가 메인 허브로서의 역할을 수행하는데 이것을 리더와 팔로워라 합니다. 즉 각각의 토픽 파티션에는 리더(leader)로 할동하는 브로커가 하나씩 있고, 0개 이상의 팔로워(flower)를 갖는다.&#x20;

<figure><img src="../../.gitbook/assets/image (269).png" alt=""><figcaption><p>카프카 복제(리더와 팔로워)</p></figcaption></figure>



## 5. Kafka 활용 사례

* **IoT (사물 인터넷)**: 초당 수백만 개의 데이터 포인트를 처리할 수 있어 대규모 데이터를 다루는 IoT 환경에 적합합니다.
* **전자상거래**: 웹 사이트 활동 트래킹, 주문, 장바구니, 재고 등 다양한 데이터를 처리할 수 있습니다.
* **IT 운영**: 모니터링, 로그 관리, 데이터 수집 등 IT 운영팀의 업무에 활용됩니다.
