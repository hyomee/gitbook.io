# Callable, Future

**동시성(Concurrency)**은 하나의 쓰레드에서 여러 Task를 관리하므로 동시에 처리하는 것처림 보이게 하는 것이다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**멀티태스팅(Multitasking**)은 하나의 시스템이 여러 작업을 동시에 처리하는 것처럼 동작하는 하는 것으로 동시성과 개념이 비슷하지만 멀티태스팅은 주로 운영 체계에서 제공된다.&#x20;

**병렬성(Parallelism)**은 여러 작업을 실제로 동시에 처리하는 것이다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<table><thead><tr><th width="145">구분</th><th>동시성</th><th>병렬성</th></tr></thead><tbody><tr><td>개념</td><td>동시에 처리하는 것처럼 보이게 하는 것</td><td>여러 작업을 실제로 동시에 처리하는 것</td></tr><tr><td>사용 코어 수</td><td>싱글 코어</td><td>멀티 코어</td></tr><tr><td>동작 방식</td><td>싱글 코어에서 멀티 쓰레드(Multi thread)를 동작 시키는 방식</td><td>멀티 코어에서 멀티 쓰레드(Multi thread)를 동작시키는 방식</td></tr><tr><td>개념적 차이</td><td>논리적인 개념</td><td>물리적인 개념</td></tr></tbody></table>

자바 버전에 따른 동시성 변천&#x20;

<table><thead><tr><th width="142">버전</th><th>사용 방법</th></tr></thead><tbody><tr><td>Java 5 이전</td><td>Runnable과 Thread를 이용하여 구현</td></tr><tr><td>Java 5</td><td>ExecutorService, Callable&#x3C;T>, Future&#x3C;T></td></tr><tr><td>Java 7</td><td>Fork/Join 그리고 RecursiveTask</td></tr><tr><td>Java 8</td><td>Stream, CompletableFuture</td></tr><tr><td>Java 9</td><td>분산 비동기 프로그래밍은 명시적으로 지원 (발행 구독 프로토콜 지원 Flow AP)</td></tr></tbody></table>

## 1. Runnable

인수가 없는 단일 메서드 run() 재정의 하며 반환 값도 없다.

```java
@FunctionalInterface
public interface Runnable {
    /**
     * Runs this operation.
     */
    void run();
}
```

{% code title="RunnableMain,java" lineNumbers="true" %}
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class RunnableMain {
    public static void main(String[] args) {
        ExecutorService service = Executors
                .newSingleThreadExecutor();
        DemoRunnable task = new DemoRunnable(5);
        service.execute(task);
        service.shutdown();
    }
}
```
{% endcode %}

{% code title="DemoRunnable.java" lineNumbers="true" %}
```java
public class DemoRunnable implements Runnable {

    private int num = 0;
    public DemoRunnable(int num){
        this.num = num;
    }

    @Override
    public void run() {

        int prod = 1;
        for (int i = 2; i <= num; i++) {
            prod *= i;
        }
        System.out.println("DemoRunnable ::" + prod);
    }
}
```
{% endcode %}

결과 : DemoRunnable ::120

## 2. Callable

**Callable**:  java.util.concurrent.Callable는 **동시에 실행 할 수 있는 작업을 나타내고 결과를 반환**하는인터페이스이다. java.lang.Runnable 인터페이스와 유사하지만 값을 반환하고 확인된 예외를 발생시킬 수 있다.

* _call():_ 인수가 없는 단일 메서드를 재정의
* 값을 반환하고 확인된 예외를 throw할 수 있다는 점을 제외하고는 _Runnable_ 인터페이스의 _run()_ 메서드와 유사

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

* Callable / Runnable 차이점

| Callable\<V>                                                     | Runnable                           |
| ---------------------------------------------------------------- | ---------------------------------- |
| Java 1.5 이후 _java.util.concurrent_ 패키지의 일부                       | Java 1.0 이후 _java.lang_ 패키지의 일부    |
| _Callable\<V_와 같은 매개 변수가 있는 인터페이스                                | 매개 변수화되지 않은 인터페이스                  |
| 확인 된 예외 throw                                                    | 체크된 예외 Throw                       |
| 정의된 인터페이스 매개 변수 Type과 동일한 반환 Type _V_가 있는 _call()_이라는 단일 메서드를 포함 | _void_를 반환하는 _run()_이라는 단일 메서드를 포함 |

{% code title="CallableTaskMain.java" lineNumbers="true" %}
```java
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableTaskMain {

    public static void main(String[] args) 
            throws ExecutionException, InterruptedException {
        ExecutorService service = Executors
                .newSingleThreadExecutor();
        CallableTask task = new CallableTask(5);
        Future<Integer> f = service.submit(task);
        Integer val = f.get();
        System.out.println("CallableTask :: " + val);
        service.shutdown();
    }
}
```
{% endcode %}

* 10 Line: ExecutorService를 생성 한다.
* 13 Line: 반환값을 받아야 하므로 Future로 반환 받기 위해 submit() 메서드로 실행을 한다.
* 14 Line: Thread 실헹이 완료될 때 까지 대기하다가 완료되면 결과를 읽어 온다.
* 16 Line: ExecutorService를 종료 한다.

{% code title="CallableTask.java" lineNumbers="true" %}
```java
import java.util.concurrent.Callable;

public class CallableTask  implements Callable<Integer> {
    private int num = 0;
    public CallableTask(int num){
        this.num = num;
    }

    @Override
    public Integer call() throws Exception {
        int prod = 1;
        for (int i = 2; i <= num; i++)
            prod *= i;
        return prod;
    }
}
```
{% endcode %}

결과 : CallableTask :: 120



## 3. Future&#x20;

**Future**:  java.util.concurrent.Future는 **비동기 계산의 결과를 나타내는 제네릭 인터페이스** 이다. 즉 비동기 작업으로 아직 되지 않았지만 나중에 완료될 수 있는 작업의 결과를 나타내는 유형으로 다음과 같은 주요 메서드가 있다.

* get() : 결과를 얻는 것으로 결과를 얻을 수 없는 경우 블록(block)된다.
* isDone() : 호출자가 완료되었는지 여부 확인 . 논 블러킹(Non Blocking)
* cancel() : 완료되기 전에 취소 한다.

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

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>실행 결과</p></figcaption></figure>

Java 8에 도입된 _CompletableFuture_ 클래스를 사용하면 _Runnable_을 비동기적으로 실행하여 값을 반환하지 않는 작업과 값을 반환하는 _Supplier_ 작업을 수행할 수 있다. _Supplier_ 는 _get ()_ 이라는 인수가없는 단일 메소드를 포함하고 _Callable_ 과 같은 결과를 반환하는 기능적 인터페이스이다.

