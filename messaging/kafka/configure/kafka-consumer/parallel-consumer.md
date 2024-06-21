# Parallel Consumer

Parallel Consumer는 단일 파티션에 여러 컨슈머 스레드를 사용하여 병렬 처리를 가능하게 합니다. 이를 통해 파티션 수를 늘리지 않고도 높은 동시 처리량을 달성할 수 있습니다.

Parallel Consumer는 메시지를 처리하기 전에 오프셋 메타데이터에 있는 `incompleteOffsets` 정보를 확인하여 현재 메시지를 처리할지 여부를 판단합니다. 이를 통해 브로커 파일 시스템 리소스 사용량, 장애에 대한 취약성, 복제 비용 등의 문제를 완화할 수 있습니다.

Parallel Consumer의 내부 구조는 다음과 같습니다:

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

1. **아키텍처**:
   * Parallel Consumer는 Kafka의 기본 Consumer와 유사한 구조를 가지고 있습니다.
   * 메시지 처리를 병렬로 수행하기 위해 여러 컨슈머 스레드를 사용합니다.
   * 오프셋 갱신은 비동기로 수행되어 메시지 처리를 연속적으로 수행할 수 있도록 합니다.
2. **순서 보장 방식**:
   * Parallel Consumer는 누적하여 이전 오프셋들에 대한 처리를 완료한 가장 마지막 오프셋을 커밋합니다.
   * 메시지 처리를 병렬로 수행하면 어떤 오프셋을 처리해야 할지 모호할 수 있습니다.
   * Parallel Consumer는 처리 결과를 임시로 저장해두고 주기적으로 오프셋을 커밋하여 순서를 보장합니다.
3. **성능 비교**:
   * Parallel Consumer를 사용하면 파티션 수를 늘리지 않고도 높은 동시 처리량을 달성할 수 있습니다.
   * 메시지 처리를 병렬로 수행하면 브로커 리소스 사용량을 최적화하고 성능을 향상시킬 수 있습니다.

Parallel Consumer를 활용하여 카프카에서 효율적인 메시지 처리를 구현할 수 있습니다. &#x20;

참고: [https://d2.naver.com/helloworld/7181840](https://d2.naver.com/helloworld/7181840)
