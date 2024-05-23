# 병렬수행

Spring Batch는 순착적으로 실행된다. 즉 기본이 단일 스레드에서 실행이 되는데 병렬 처리를 통해 성능을 개선 할 수 있다.&#x20;

* 주의사항&#x20;
  * 서버 자원(CPU/Memory 등)이 풍족해야 한다. 단일 스레드에서 서버 자원에 문제가 있으면 멀티 스레드로 하여도 성능 향상을 기대할 수는 없다.
  * 사용하는 Reader/Writer이 스레드에 안전(thread-safe)해야 한다.
  * 만약스레드에 안전하지 않으며 [SynchronizedItemStreamReader](https://docs.spring.io/spring-batch/docs/current/api/org/springframework/batch/item/support/SynchronizedItemStreamReader.html)을 이용해 **thread-safe**로 변환해서 사용해야 한다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

## 1. 사용 가능한 TaskExecutor

* [SimpleAsyncTaskExecutor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/core/task/SimpleAsyncTaskExecutor.html)
  * 새로운 스레드를 각 작업마다 생성하여 비동기적으로 실행하는 것으로 스레드를 재사용하지 않고 매번 새로운 스레드를 만든다.
  * 기본적으로 동시 스레드는 무제한이어서 concurrencyLimit 속성을 통해 동시 실행 스레드 수를 제한하여 사용해야 한다.
  * <mark style="color:purple;">대량 작업은 피해야 하며 스레드 풀링 TaskExecutor, JDK 21에서는 setVirtualThreads를 true로 설정하여 가상 스레드를 고려해야 한다.</mark>
* [ThreadPoolTaskExecutor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html):&#x20;
  * 스레드 풀에서 작업을 실행하며 비동기적으로 실행하여 응답 시간을 최적화 할 수 있다.
  * corePoolSize, maxPoolSize, queueCapacity 등의 설정을 통해 동시 실행 스레드 수를 제어할 수 있다.

## 2. 병렬 수행 방법&#x20;

Spring Batch에서 병렬처리 옵션은 다음과 같다.

* **Thread Safe한 PagingItemReader**&#x20;
  * Paging 방식으로 시작 Row와 PageSize(가져올 Row) 만큼 데이터를 읽는 방식으로 Cusor방식 처럼 순차적으로 데이터를 가지고 않기 때문에 사용가는한데 Thread Safe한 PagingItemReader를 사용해야 멀티 스레드 환경에서 안전하게 사용할 수 있다.
  * JpaPagingItemReader**를 사용햐여 한다,** MyBatis, JDBC 관련 Reader**는** Thread Safe하지 않다.
* **Partitioning**
  * **Partitioner**를 사용하여 분할된 범위를 각 Step에 전달하여 병렬처리하는 방식으로  Reader 자원을 공유하지 않으므로 synchronized에 영향을 받지 않**고** 처리할 수 있다.&#x20;
  * Partitioner을 어떻게 나누어야 할지에 대해서 고민을 해야 한다. 즉 데이터의 분포와 작업량에 따라 적절한 **GridSize**를 설정해야 한다.



