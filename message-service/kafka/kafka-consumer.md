# Kafka-Consumer

Kafka 브로커에서 메시지를,읽어와서  소비하는 역할을 하는구것으로  지정돤 토픽의 데이터(메시지)를 레코드 단위로 가져와 소비합니다. 메시지를 읽을 때, 부하 분산과 가용성 및 처리량 증가에 도움이 되는 파티션을 사용하고, 오프셋을 통해 메시지를 추적합니다. 다음은 Kafka Consumer에 관한 주요 사항입니다.

<figure><img src="../../.gitbook/assets/image (344).png" alt=""><figcaption></figcaption></figure>

1. **데이터 가져오기(Kafka Consumer)**:
   * Consumer는 브로커에게 가져올 파티션을 지정하는 “fetch” 요청을 보냅니다.
   * 각 요청은 로그 오프셋을 지정하며 해당 오프셋 위치부터 로그의 일부를 수신합니다.
   * Consumer는 위치를 제어하고 필요한 경우 데이터를 다시 소비할 수 있습니다.
2. **토픽(Topic) 및 파티션(Partition):**
   * **Topic**: 메세지 범주&#x20;
   * **Partition:** Topic에서 가장 작은 저장 단위로 Topic에 대한 파티션 수를 구성할 수 있습니다.
3. **Consumer 그룹**:
   * Consumer는 **데이터를 소비하기 위해 그룹을 형성**할 수 있습니다.
   * Consumer 그룹은 소비자 구성에서 group.id 속성을 설정하여 정의됩니다.
   * &#x20;group.id가 지정되지 않으면 무작위로 생성됩니다.
   * 모든 토픽의 파티션은 그룹 내의 소비자에게 분배됩니다.
   * 새 멤버가 가입하거나 나가면 파티션은 비례적으로 재할당됩니다.
   * 한 브로커가 그룹 코디네이터 역할을 하며 멤버와 할당을 관리합니다.
   * 코디네이터는 내부 오프셋 토픽(\_\_consumer\_offsets)의 리더 중에서 선택됩니다.
   * Consumer 그룹은 동일한 토픽에서 메시지를 병렬로 처리할 수 있도록 합니다.
4. **오프셋 관리**:
   * 할당을 받은 후 Consumer는 각 파티션에 대한 초기 위치를 결정합니다.
   * Consumer는 로그 내에서 위치를 조정할 수 있습니다.
   * Kafka는 동일한 토픽에 여러 Consumer 애플리케이션이 동시에 구독할 수 있도록 합니다.

Kafka Consumer는 확장 가능하고 내결함성 있는 데이터 처리 파이프라인 구축에 중요한 역할을 합니다.

## 1. 간단한 Use Case를 통해서 보는 Consumer 그룹

<figure><img src="../../.gitbook/assets/image (345).png" alt=""><figcaption></figcaption></figure>







참고: [https://docs.confluent.io/platform/current/clients/consumer.html](https://docs.confluent.io/platform/current/clients/consumer.html)
