# GC ( Garbage Collection )

자바는 메모리를 GC라는 알고리즘을 통해서 관리하기 메모리를 관리를 개발자가 하지 않았고 절대 만들지 말아야 합니다.

### 1. Garbage Collection : JVM의 Heap 영역에서 동적으로 할당했던 메모리 영역 중 필요 없게 된 메모리 영역을 주기적으로 삭제하는 프로세스

JVM Runtime Data Area은 다음과 같습니다.

| <img src="../.gitbook/assets/image (20).png" alt="" data-size="original"> | <p></p><ul><li>Method Area </li></ul><ul><li>Heap Area </li></ul><ul><li>JVM Stack <br>- StackOverflowError<br>- OutOfMemoryError</li></ul><ul><li>Program Counter Register  </li></ul><ul><li>Runtime Constant Pool  </li></ul><ul><li>Native Method Stack  </li></ul> |
| ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |



이 영역 중 GC가 발생하는 곳은 Heap영역으로 GC는 다음의 역활을 합니다.

* 메모리 할당
* 사용 중인 메모리 인식
* 사용하지 않는 메모리 인식

지정한 메모리 영역이 꽉 차서 JVM에 행(Hang)이 걸리거나 지장한 메모리 보다 더 많음 메모리를 할당 하려고 하거나 하면 OutOfMemoryError가 발생 하여 JVM이 다운 될 수 있습니다. 이것을 방지 하기 위해  GC가 메모리를 관리하여 합니다.

<figure><img src="../.gitbook/assets/image (21).png" alt="" width="520"><figcaption></figcaption></figure>

Young, Old, Perm 영역으로 나뉘는데 Java 1.8부터는 Perm 영역이 사라졌서,  **Young 영역(Eden, Survivor 1, Survisor 2) , Old 영역**으로 나뉘어 진다&#x20;

### 2. GC 대상



<figure><img src="../.gitbook/assets/image (29).png" alt="" width="563"><figcaption></figcaption></figure>

### 3. **메모리에 객체가 생성 되면**&#x20;

**3-1. Eden 영역에 객체가 지정됨**&#x20;

<figure><img src="../.gitbook/assets/image (23).png" alt="" width="563"><figcaption></figcaption></figure>

**3-2. Eden 영역애 객체가 꽉 차면 Survivor 영역으로 객채가 옯겨짐**&#x20;

* Survivor 영역 중 하나는 비어 있어여 함&#x20;
* 비어 있는 Survivor 영역은 GC 후에 살아 남아 있는 객체들이 이동 해야 하므로

<figure><img src="../.gitbook/assets/image (24).png" alt="" width="563"><figcaption></figcaption></figure>

**3-3.  GC 가 발생 하면 Eden 영역에 있는 객체가 Survivor 1, Survivor 2 영역으로 이동**&#x20;

* Survivor 1, Survivor 2 영역에서 욌다 갔다 하던 객체가 Old 영역으로 이동&#x20;
* Old 영역으로 바로 가는 객체는 크기가 큰 경우 Survivor  15M인데 객체가 18M이면&#x20;

<figure><img src="../.gitbook/assets/image (25).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (26).png" alt="" width="563"><figcaption></figcaption></figure>

**3-4.   Survivor 1영역에 있는 객체가 Old영역으로 이동**&#x20;

<figure><img src="../.gitbook/assets/image (27).png" alt="" width="563"><figcaption></figcaption></figure>

**3-5.   Major GC 발생**&#x20;

<figure><img src="../.gitbook/assets/image (28).png" alt="" width="563"><figcaption></figcaption></figure>

## 1. GC 종류

GC가 발생하거나 객체가 각 영역에서 다른 영역으로 이동할 때 애플리케이션의 병목이 발생하면서 성능에 영향을 주게 된게 되는 HotSpot VM에서는 Thread-Local Allocation Buffers라는 것을 사용하여 다른 스레드에 영역을 주지 않는 메모리 할당 작업이 가능해 집니다.

* Minor GC : Young 영역에서 발생 하는 GC
* Major GC : Old 영역에서 발생 하는 GC

## 2. GC 방식

1. Serial Collector&#x20;
   * Young 영역과 Old 영역이 시리얼 하게 하나의 CPU를 사용
   * 애플리게이션 수행이 멈춘다.
   * JVM Option : -XX:+UseSerialGC
2. Parallel Collector
   * Throughput Collector 라고 함&#x20;
   * 다른 CPU가 대기 상태로 남아 있는 것을 최소화
   * Young 영역에서의 Collection을 병렬로 처리
   * 많은 CPU를 사용 하기 때문에 GC의 부하를 줄이고 애플리케이션의 처리량 증가
   * JVM Option : -XX:+UseParallelGC
3. Parallel Compacting Collector
   * Young 영역에서의 Collection을 병렬로 처리
   * Old 영역은&#x20;
     * 표시 단계 : 살아 있는 객체 표시
     * 종합 단계 : 이전에 GC수행하여 컴팩션된 영역에 살아 있는 객체의 위치 조사
     * 컴팩션 단계 : 컴팩션을 수행하는 단계, 수행 이후에는 컴팩션된 영역과 비어있는 영역으로 나뉨
   * JVM Option : -XX:+UseParallelThreads=n, -XX:+UseParallelOldGC
4. Concurrent Mark-Sweep Collector
   * Low-Latency Collector라고도 함&#x20;
   * 힙 메모리 영역이 클 때 적합&#x20;
   * JVM Option : -XX:CMSIniiatingOccupancyFraction=n
5. Garbage First Collector
   * stop-the-world에 의한 일시 중지 시간을 줄이는 것을 목표로한다
   * Heap영역을 체스판처럼 여러 영역으로 나누어 관리한다
   * JVM 7에서 동시 마크 스윕 콜렉터(CMS)를 대체하기 위해 계획되었으며 Java 9에서 기본값으로 설정되었습니다.&#x20;
   * &#x20;메모리가 큰 다중 처리기를 대상으로 하는 서버 스타일 가비지 수집기로, 높은 처리량을 달성하면서 높은 확률로 소프트 실시간 목표를 충족합니다.
   * 참고 : [https://docs.oracle.com/en/java/javase/20/gctuning/garbage-first-g1-garbage-collector1.html](https://docs.oracle.com/en/java/javase/20/gctuning/garbage-first-g1-garbage-collector1.html)

## 3. JVM 옵션

GC를 사용하여 성능을 올리는 방법은 Option을 잘 조정 하여 사용해야 한다. New영역의 크기를 적절히 조절하여 효과를 볼수 있으며 Full GC 시간을 줄여 성능을 향상 할 수 있다.

| 구분         | 옵션                 | 설명                      |
| ---------- | ------------------ | ----------------------- |
| Heap 영역 크기 | --Xms              | JVM 시작 시 힙 영역 크기        |
| Heap 영역 크기 | --Xmx              | 최대 힙 영역 크기              |
| New 영역 크기  | --XX:NewRatio      | New 영역과 Old 영역의 비율      |
| New 영역 크기  | --XX:NewSize       | New 영역의 크기              |
| New 영역 크기  | --XX:SurvivorRatio | Eden영역과 Survivor 영역의 비율 |





1\.&#x20;

