# Future

**동시성(Concurrency)**은 하나의 쓰레드에서 여러 Task를 관리하므로 동시에 처리하는 것처림 보이게 하는 것이다.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

**멀티태스팅(Multitasking**)은 하나의 시스템이 여러 작업을 동시에 처리하는 것처럼 동작하는 하는 것으로 동시성과 개념이 비슷하지만 멀티태스팅은 주로 운영 체계에서 제공된다.&#x20;

**병렬성(Parallelism)**은 여러 작업을 실제로 동시에 처리하는 것이다.

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<table><thead><tr><th width="145">구분</th><th>동시성</th><th>병렬성</th></tr></thead><tbody><tr><td>개념</td><td>동시에 처리하는 것처럼 보이게 하는 것</td><td>여러 작업을 실제로 동시에 처리하는 것</td></tr><tr><td>사용 코어 수</td><td>싱글 코어</td><td>멀티 코어</td></tr><tr><td>동작 방식</td><td>싱글 코어에서 멀티 쓰레드(Multi thread)를 동작 시키는 방식</td><td>멀티 코어에서 멀티 쓰레드(Multi thread)를 동작시키는 방식</td></tr><tr><td>개념적 차이</td><td>논리적인 개념</td><td>물리적인 개념</td></tr></tbody></table>

자바 버전에 따른 동시성 변천&#x20;

<table><thead><tr><th width="142">버전</th><th>사용 방법</th></tr></thead><tbody><tr><td>Java 5 이전</td><td>Runnable과 Thread를 이용하여 구현</td></tr><tr><td>Java 5</td><td>ExecutorService, Callable&#x3C;T>, Future&#x3C;T></td></tr><tr><td>Java 7</td><td>Fork/Join 그리고 RecursiveTask</td></tr><tr><td>Java 8</td><td>Stream, CompletableFuture</td></tr><tr><td>Java 9</td><td>분산 비동기 프로그래밍은 명시적으로 지원 (발행 구독 프로토콜 지원 Flow AP)</td></tr></tbody></table>

## 1. Java Future, Callable

**Future**:  java.util.concurrent.Future는 비동기 계산의 결과를 나타내는 인터페이스 이다. 즉 비동기 작업으로 아직 되지 않았지만 나중에 완료될 수 있는 작업의 결과를 나타내는 유형으로 다음과 같은 주요 메서드가 있다.

* get() : 결과를 얻는 것으로 결과를 얻을 수 없는 경우 블록(block)된다.
* isDone() : 호출자가 완료되었는지 여부 확인 . 논 블러킹(Non Blocking)
* cancel() : 완료되기 전에 취소 한다.

**Callable**:  java.util.concurrent.Callable는 동시에 실행 할 수 있는 작업을 나타내고 결과를 반환하는인터페이스.이다. java.lang.Runnable 인터페이스와 유사하지만 값을 반환하고 확인된 예외를 발생시킬 수 있다.

Future 또는 Callable를 사용하기 위해서는 동시에 작업을 실행하는 역할을 담당하는 Executor 또는 ExecutorService가 필요한데  java.util.concurrent 팩키지에 ThreadPoolExcutor, ForkJoinPool과 같은 인퍼페이스의 구현체를 제공하고 있다.

다음 예제는 Future와 Callable를 사용한 비동기 예제이다.

{% code lineNumbers="true" %}
```java
@Component
public class FutureCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        Future<String> future = executorService.submit(
                new Callable<String>() {
                    @Override
                    public String call() throws Exception {                    
                        System.out.println("Callable ...  call" );
                        Thread.sleep(1000);
                        return "안녕 Callable";
                    }
                }
        );

        System.out.println("결과를 기다리는 중 ...  isDone :: " + future.isDone()  ");
        String result = future.get();
        System.out.println(result);
        executorService.shutdown();
    }
}
```
{% endcode %}

* 5 \~ 15 Line:  비동기 작업을 위해 ExecutorService를 단일 쓰레드 생정자를 생성 하고 submit() 메서드를 사용하여 비동기 작업인 Callable를 생성 하여 실행 하여 결과를 Future 객체로 반환한다.
* 18 Line: 비동기 작업이 실행 완료될 때 까지 대기한다.
* 19 Line: 비동기 작업 결과 값을 출력 한다.
* 20 Line: 비동기 작업을 종료 한다.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>실행 결과</p></figcaption></figure>



## 2. Future

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>



* cancel:&#x20;
