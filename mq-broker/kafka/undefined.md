---
description: >-
  Apache Kafka는 고성능 데이터 파이프 라인, 스트리밍 분석, 데이터 통합 ​​및 미션 크리티컬 애플리케이션을 위해 사용하는 오픈
  소스 분산 이벤트 스트리밍 플랫폼
---

# 개요

## 1. 일반적인 메세지 모델

### **1-1. Queuing Model**

여러 Consumer가 분산 처리로 메시지를 소비하는 모델로  프로듀서에서 메세지가 큐에 담기고 컨슈머가 큐에서 메세지를 추출하는 방법으로 추출한 메세지는 컨슈머 중 하나가 처리합니다. &#x20;

<figure><img src="../../.gitbook/assets/image (275).png" alt=""><figcaption></figcaption></figure>

### **1-2. Pub/Sub Model**

여러 Subscriber에 동일한 메시지를 전달 하고, Topic 기반으로 전달 내용을 변경하는 모델로 다음과 같은 특징이 있습니다.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* 퍼블리서에서 발행한 메세지는 브로커의 토픽에 보관되며 발행한 메세지의 소비는 퍼블리서에서 관심이 없습니다.
* 서브스크라이버는 여러개의 토픽 중에 하나를 선택하여 받아 소비합니다.&#x20;
* 서비스크라이버는 관심이 있는 토픽만 구독 할 수 있어 여러 서비스크라이버는 동일한 토픽을 구독하여 동일한 메세지를 소비 할 수 있습니다.

## 2. Kafka 메세지 모델&#x20;

카프카 메세지 모텔은 프로듀서(Producer), 브로커(Broker), 컨슈머(Consumer)로 구성이 되면 두가지 모델이 있으며 Consumer Group을 도입하여 컨슈머를 확장할 수 있습니다.

* 여러 컨슈머다 분산 처리 하는 모델 (**Queuing Model**)&#x20;
* 토픽기반으로 여러 서비스클라이버가 동일한 매세지를 받는 모델 (**Pub/Sub Model**)

## 3. Kafka API

다양한 제품과 연동을 위해 제공하는 API로 Connect API와 Stream API 을 제공하고 있으며 각각 다음과 같은 역활을 합니다.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### 3-1. Kafka Producer API

&#x20;Java 클라이언트는 Kafka 클러스터에 데이터 스트림을 게시하는 데 사용됩니다. 이 API를 사용하여 애플리케이션은 하나 이상의 Kafka 토픽으로 레코드 스트림을 전송할 수 있습니다. 이제 몇 가지 주요 기능을 살펴보겠습니다.:

1. 스레드 안전한 프로듀서 인스턴스: 단일 프로듀서 인스턴스를 여러 스레드와 공유하는 것은 일반적으로 속도를 높입니다.&#x20;
2. &#x20;비동기 send() 메서드: 호출하면 레코드를 보류 중인 전송 버퍼에 추가하고 바로 반환합니다, 이는 효율적인 레코드 배치를 가능하게 합니다.&#x20;
3. &#x20;acks 설정: 요청 완료 기준을 제어합니다. "all" 설정은 레코드의 완전한 커밋을 기다리는 느린 방식이지만 가장 안정적입니다.&#x20;
4. &#x20;버퍼 관리: 각 파티션에는 미전송 레코드를 위한 버퍼가 있으며, 이의 크기는 batch.size 설정에 따라 결정됩니다.&#x20;
5. Idempotent 프로듀서 모드: 최소 한 번 이상 정확히 한 번까지의 전송을 보장하며, 중복 전송을 방지합니다.​

참고: [https://kafka.apache.org/documentation/#producerapi](https://kafka.apache.org/documentation/#producerapi)

### 3-2. Kafka Consumer API

Kafka 클러스터에서 데이터를 읽는 역할로이 API를 사용하여 애플리케이션은 하나 이상의 토픽을 구독하고 해당 토픽에 저장된 스트림(메세지)을 가져와 애플리케이션에 필요한 처리를 수행합니다. 즉 실시간으로 데이터를 처리하거나 과거의 레코드를 입수하여 처리할 수 있습니다.

참고: [https://kafka.apache.org/documentation/#consumerapi](https://kafka.apache.org/documentation/#consumerapi)

### 3-3. Kafka Consumer API

Apache Kafka의 구성 요소로서 데이터 파이프라인을 간소화하는 역할로 다른 데이터 시스템 간의 테이터 가져오기/내보내기를 쉽게 해주는 API로 Kafka와 통합되는 외부 시스템 및 응용 프로그램에서 이벤트 스트림을 소비(읽기)하거나 생성(쓰기)할 수 있습니다.

1. **Source Connectors**:
   * Source 커넥터는 외부 시스템에서 Kafka 토픽으로 데이터를 가져오는 역할을 합니다.
   * 예를 들어, **JDBC Source Connector**는 관계형 데이터베이스에서 데이터를 읽어 Kafka 토픽으로 전송할 수 있습니다.
2. **Sink Connectors**:
   * Sink 커넥터는 Kafka 토픽에서 데이터를 가져와 외부 시스템으로 전송하는 역할을 합니다.
   * 예를 들어, **Elasticsearch Sink Connector**는 Kafka 토픽에서 데이터를 읽어 Elasticsearch 클러스터로 색인할 수 있습니다.
3. 그외 Transform Connectors, Custom Connectors가 있습니다.

참고: [https://kafka.apache.org/documentation.html#connect](https://kafka.apache.org/documentation.html#connect)

### 3-4. Kafka Stream API

Apache Kafka 개발 프로젝트에서 공식적으로 제공되는 스트림 프로세싱 프레임워크입니다. 이 Java로 구현되어 있으며, 카프카 클러스터 내의 토픽에 저장된 데이터를 실시간으로 처리, 변환 및 분석할 수 있도록 도와주는 것으로 스트림 프로세싱을 간편하게 구현하고, 카프카의 서버 사이드 클러스터 기술과 결합하여 확장성, 탄력성, 분산 처리, 고가용성 등을 제공하여 실시간 데이터 처리를 위한 강력한 도구로 활용할 수 있습니다 다음과 같은 특징이 있습니다.

1. 간단하고 가벼운 클라이언트 라이브러리: 기존 자바 애플리케이션에서 쉽게 사용할 수 있습니다.
2. 시스템이나 카프카에 대한 의존성 없음: Kafka Streams는 카프카 클러스터 내의 데이터를 처리하므로 별도의 시스템이나 카프카에 대한 의존성이 없습니다.
3. 이중화된 로컬 상태 저장소 지원: Stateful한 어플리케이션을 구현할 때 RocksDB와 같은 로컬 데이터 스토어를 사용하여 낮은 대기 시간을 유지합니다[2](https://www.devkuma.com/docs/apache-kafka/strems/).
4. 1번만 처리되는 보장: 카프카 브로커나 클라이언트에 장애가 생기더라도 스트림에 대해선 1번만 처리되는 것을 보장합니다.
5. 토폴로지 기반 API: 스트림 처리를 하는 프로세스들이 서로 연결되어 있는 토폴로지를 만들어서 처리할 수 있습니다

참고: [https://kafka.apache.org/37/documentation/streams/](https://kafka.apache.org/37/documentation/streams/)

### 3-4. Kafka Admin API

Kafka 클러스터를 관리하고 관리하는 데 사용되는 Kafka API로 이 API를 통해 개발자는 프로그래밍 방식으로 Kafka 리소스를 생성, 삭제, 설명 및 수정할 수 있습니다. 주요 작업은 다음과 같습니다:

1. **토픽 관리**: 토픽 생성, 삭제, 설명, 수정 등을 수행합니다.
2. **브로커 관리**: 브로커 정보를 조회하고 수정합니다.
3. **구성 관리**: Kafka 구성 항목을 관리합니다.
4. **ACL(접근 제어 목록) 관리**: 접근 권한을 설정하고 관리합니다.

**Admin API**를 사용하면 Kafka 클러스터를 프로그래밍 방식으로 효율적으로 관리할 수 있습니다

참고: [https://kafka.apache.org/documentation/#adminapi](https://kafka.apache.org/documentation/#adminapi)

## 2. Kafka 구성 요소

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Kafka 개념도</p></figcaption></figure>



* Broker: 데이터를 수신, 전달(Consumer의 요구에 따라 응답)&#x20;
* Message: 데이터의 최소 단위, key/value 구조, 전송 시 Partition 이용&#x20;
* Producer: 데이터 생산자, broker에 Message 전달&#x20;
  * 레코드를 프로듀스할 때 어느 토픽의 어느 파티션에 할당할 지를 결정한다&#x20;
* Consumer: 메시지 가지고 온다.
* Topic: 메시지 종류별로 Broker에서 관리&#x20;
  * 카프카 안에는 여러 레코드 스트림이 있을 수 있다.&#x20;
  * 하나의 토픽에 대해 여러 Subscriber가 붙을 수 있음

## 3. Kafka 시스템 구성

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>Kafka 구성</p></figcaption></figure>





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

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption><p>카프카 분산 메세지 구조</p></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>카프카 물리적 구조</p></figcaption></figure>

카프카 클러스터는 다중 브로커로 구성이 되며 클러스터에 대한 쓰기/읽기 작업의 부하 분산을 도와주고 있으며 각 브로커의 상태는 주키퍼를 사용 합니다.

#### 4-2-1. 리더와 팔로워

카프카는 장애 대응을 위해 메인 브로커가 다운되더라도 리플리케이션되고 있는 브로커가 메인 허브로서의 역할을 수행하는데 이것을 리더와 팔로워라 합니다. 즉 각각의 토픽 파티션에는 리더(leader)로 할동하는 브로커가 하나씩 있고, 0개 이상의 팔로워(flower)를 갖는다.&#x20;

<figure><img src="../../.gitbook/assets/image (269).png" alt=""><figcaption><p>카프카 복제(리더와 팔로워)</p></figcaption></figure>



