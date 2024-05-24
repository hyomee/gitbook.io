---
description: Apache Kafka는 반드시 한 메시지 단위로 송수신 하는 기능과 처리량을 높이기 위해서 메시지를 축척하여 처리 하는 기능을 제공합니다.
---

# 메세지 송수신

카프카에서 메세지는 한 개 메세지 단위로 송수신하는 기능과 어느정보 데이터를 축척해서 배치 처리로 송수신하는 기능을 제공합니다.

## 1. 프로듀서 메세지 송신

<figure><img src="../../.gitbook/assets/image (278).png" alt=""><figcaption></figcaption></figure>

* **하나의 메세지 송신**: 기본 설정 값으로 하나의 메세지를 송신하는 방법
* **배치 처리**: 처리량 향상을 위해 프로듀서에서 일정량의 메세지를 모아서 송신하는 방법으로 프로듀서의 메모리를 사용하며 송신 데이터는 설정한 크기(batch.size),  지정된 시간(linger.ms) 까지 축척 후 송신합니다.&#x20;

## 2. 컨슈머  메세지 수신(취득)

토픽과 파티션에 대해서 Current Offset 위티에서 마지막으로 취득한 메시지 부터 브로커에 요청 하여 브로커에 보관 되어 있는 최신 메시지까지 수신하므로 브로커 요청 간격이 길수록 모인 메시지가 많아 집니다.

<figure><img src="../../.gitbook/assets/image (279).png" alt=""><figcaption></figcaption></figure>

<mark style="color:purple;">모아서 받는 경우 프로듀서 송신과 컨슈머 수신의 지연 시간이 발생 할 수 있으므로 처리령과 대기 시간의 트레이드 오프를 고려한 설계를 해야 합니다.</mark>

### 2-1. 컨슈머 장애 대응

Offset Commit의 구조를 이용해 컨슈머 처리 실패, 고장 시 롤백 메시지 재 취득을 하여야 하는데 오류에 대한 처리는 Consumer API를 이용한 애플리케이션에서 고려해야 합니다.

* Commit Offset update 직전 고장의 경우: 중복 메시지가 수신 될 수 있으므로 고려 해야 함 (At Least Once)

<figure><img src="../../.gitbook/assets/image (280).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (282).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (283).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (284).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (285).png" alt=""><figcaption></figcaption></figure>

## 3.  프로듀서 파티서닝

토픽 당 데이터를 분산 처리하는 단위로 토픽 안에 파티션을 나누어 그 수대로 데이터를 분산 처리하는 것으로 <mark style="color:purple;">프로듀서에서  송신하는 메세지를 어떻게 파티션해서 보낼지 결정하는 파티셔닝을 메시지에 포함된 key의 명시적인 지정 여부에 따라 두가지 패턴이 제공</mark>됩니다.

*   **Key 해시 값을 사용한 송신**: 메시지 key를 명시적으로 지정 함으로 Key에 따라서 파티션을 결정 하는 로직으로 **동일한** <mark style="color:orange;">Key를 가진 메시지는 동일한 ID를 가진 파티션에 송신</mark> 됩니다.\


    <figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
*   **라운드 로빈에 의한 송신**: 메시지 Key를 지정 하지 않고 null로 하면 여러 파티션에 라운드 로빈 방식으로 송신 됩니다.\


    <figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

    1. DefaultPartitioner Class이용해서 구현합니다.
    2. Producer API에서 제공 하는 Partitioner 인터페이스를 구현함으로써 Key,Value 값에 따라서 송신 로직을 커스텀으로 구현합니다.



{% hint style="info" %}
파티션

* 각 토픽마다 데이터를 여러개의 파티션에 나누어서 저장/처리
* 토픽 사이즈가 커질 경우 파티션을 늘려서 스케일아웃을 할 수 있다&#x20;
* 일반적으로 브로커 하나당 파티션 하나&#x20;
* 컨슈머 인스턴스 수는 파티션 갯수를 넘을 수 없다. 그러므로 병렬처리의 수준은 파티션 수에 의해 결정된다. 즉 파티션이 많을 수록 병렬처리 정도가 높아진다.&#x20;
* 각 파티션마다 Publish되는 레코드에 고유 오프셋을 부여한다. 때문에 레코드는 파티션 내에서는 유니크하게 식별된다. 하지만 파티션간에는 순서를 보장하지 않는다.&#x20;
* 전체 순서를 보장하고 싶으면 파티션을 하나만 두는 수 밖에 없는데, 이러면 데이터량이 많아져도 스케일아웃이 안되고, 컨슈머 인스턴스도 하나만 둘 수 있으므로 병렬 컨슈밍도 안된다.
{% endhint %}

## 4. 브로커 데이터 보관 기간

카프카는 수신된 메시지를 디스크에 보관(영속화)하고 컨슈머는 보관되어 있는 메시지를 읽어 들이는 구조로 보관된 데이터는 메시지 취득 후 경과 시간, 데이터 크기 두가지 정책으로 설정합니다.

* **데이터 취득 후 경과 시간을 트리거 하는 경우**
  * 시간, 분, 밀리초 등으로 지정, 시정한 시간 보다 오래된 데이터 삭제
  * 기본 168시간 (1주)
* **데이터 크기를 트리거 하는 경우**
  * 지정한 데이터 크기보다 커진 경우 데이터 삭제&#x20;
  * 기본 -1 ( 크기 제한 없음 )
* 압축 : 최신 Key의 데이터를 남겨두고 중복하는 Key의 오래된 메시지 삭제
* cleanup.policy : delete 또는 compact 값을 이용해서 설정합니다.



