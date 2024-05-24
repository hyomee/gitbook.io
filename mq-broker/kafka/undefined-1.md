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

## 3. 파티서닝

