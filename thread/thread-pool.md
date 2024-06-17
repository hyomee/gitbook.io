# 자바 Thread Pool

Thread Pool은 미리 할당된 스레드 집합에서 스레드를 가지고 와서 실행 후 종료되면 반납하는 것으로 용 프로그램이 스레드가 필요할 때마다 새 스레드를 만들지 않으므로 리소스 소비를 크게 최소화할 수 있다. 또한 스레드 풀은 작업에 큐를 사용할 수 있으므로 사용 가능한 스레드 부족으로 인해 중요한 작업이 지연되지 않도록 할 수 있다.



## 1. 스레드 풀을 사용해야 하는 이유

스레드를 만들면 오버헤드가 발생하여 많은 양의 데이터를 처리할 때 메모리 사용량이 증가하고 실행 속도 느려질 수 있고 스레드 간을 전환할 때마다 컨텍스트 전환, CPU 캐시 플러시 및 다른 스택을 메모리에 로드하는 등 백그라운드에서 진행되는 여러 가지 다른 작업으로 인해 성능이 저하된다는 점을 고려해야 한다. 이런 문제를 해결하기 위해 스레드 풀을 사용하여 개발자가 주어진 시간에 사용되는 스레드 수를 제어하는 동시에 수명 주기를 관리하여 방지할 수 있으며 다음과 같은 장점이 있다.

* 스레드를 재사용하고 작업이 실행될 때마다 새 스레드를 만드는 오버헤드를 방지하여 응용 프로그램의 성능을 향상시키 수 있다.
* 작업을 큐에 대기시키고 스레드를 사용할 수 있게 되는 즉시 실행하여 작업이 적시에 실행시킬 수 있다.
* 작업을 동시에 실행할 수 있도록 하고 지정된 시간에 활성 상태인 스레드 수를 제어하는 메커니즘을 제공하여 응용 프로그램 성능을 향상시킬 수 있다.
* 응용 프로그램에 필요한 스레드 수를 크게 줄일 수 있으므로 리소스 소비를 줄이고 성능을 크게 향상시킬 수 있다.

## Thread Pool 이란

<figure><img src="../.gitbook/assets/image (216).png" alt=""><figcaption></figcaption></figure>

동시에 여러 작업을 효율적으로 실행 및 관리하기 위해 서버에서 만드는 스레드의 모음으로 새로운 Thread를 만들지 않고 미리 생성된 Thread를 Thread Pool에서 가지고와 재사용하는것으로 성능과 리소스 관리에 도움이 된다

## 2. 스레드 풀을만드는 방법

java.util.concurrent 패키지는 Executors 클래스 및 ThreadPoolExecutor 클래스를 사용하여 스레드 풀을 만들 수 있다.

자바에서 스레드 풀을 생성하고 사용할 수 있도록 java.util.concurrent패키지에서 ExecutorService인터페이스와 Executors클래스를 제공하고 있으며 다음과 같은 기능을 제공하고 있다.

* newFixedThreadPool(int nThreads): 고정된 개수의 쓰레드풀을 생성한다.
* newCachedThreadPool(): 필요한 만큼 쓰레드풀을 생성하고 이미 생성된 쓰레드를 재활용한다.
* newScheduledThreadPool(int corePoolSize): 일정 시간 뒤에 실행되거나 주기적으로 수행되는 작업을 처리할 수 있는 쓰레드풀
* newSingleThreadExecutor(): 쓰레드 1개인 ExecutorService를 리턴합니다. 싱글 쓰레드에서 동작해야 하는 작업을 처리할 때 유용하다

스레드 풀을 종료할 때는 shutdown(), shutdownNow(), awaitTermination() 메소드를 사용합니다. 작업 생성과 처리 요청은 execute(Runnable command) 또는 submit(Runnable task) 메소드를 통해 작업할 수 있으며 ThreadPoolExecutor를 통해서 실행하는 것은 Runnable 또는 Callable 인터페이스의 인스턴스이어야 한다. 작업 처리 결과를 받기 위해 submit 메소드를 사용할 수 있다.

### 2-1. ExecutorService 클래스를 사용

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ExecutorServiceMain {
    public static void main(String[] args) {
        // 3개 fixed-size thread pool 생성
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        // task 실행
        executorService.submit(new Runnable() {
            public void run() {
                System.out.println("run 메서드 실행");
            }
        });
        // executor service shutdown
        executorService.shutdown();
    }
}
```

결과: run 메서드 실행

### 2-2. ThreadPoolExecutor 클래스를 사용

```java
public class ThreadPoolExecutorMain {

    public static void main(String[] args)
    {
        Runnable myTask = ()-> {
            String thame = Thread.currentThread().getName();
            System.out.println(thame + "실행 ...");
        };

        // 2개 fixed-size thread pool 생성
        ThreadPoolExecutor threadPoolExecutor =
                (ThreadPoolExecutor) Executors.newFixedThreadPool(2);

        for (int i = 1; i <= 5; i++)
        {
            threadPoolExecutor.execute(myTask);
        }
        
        threadPoolExecutor.shutdown();
    }
}
```

* 결과

```
pool-1-thread-2실행 ...
pool-1-thread-1실행 ...
pool-1-thread-2실행 ...
pool-1-thread-1실행 ...
pool-1-thread-2실행 ...
```

### 2-3. 4개의 쓰레드를 갖는 고정된 쓰레드풀 생성

{% code lineNumbers="true" %}
```java
public void excutorServiceSample() throws InterruptedException {
    // 4개의 쓰레드를 갖는 고정된 쓰레드풀 생성
    ExecutorService executor = Executors.newFixedThreadPool(4);

    // 작업 예약
    executor.submit(() -> {
        String threadName = Thread.currentThread().getName();
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        System.out.println("Job1 " + threadName);
    });
    executor.submit(() -> {
        String threadName = Thread.currentThread().getName();
        System.out.println("Job2 " + threadName);
    });
    executor.submit(() -> {
        String threadName = Thread.currentThread().getName();
        System.out.println("Job3 " + threadName);
    });
    executor.submit(() -> {
        String threadName = Thread.currentThread().getName();
        System.out.println("Job4 " + threadName);
    });

    // 더 이상 작업을 추가할 수 없도록 설정
    executor.shutdown();

    // 작업이 모두 완료될 때까지 대기
    if (executor.awaitTermination(2, TimeUnit.SECONDS)) {
        System.out.println("All jobs are terminated");
    } else {
        System.out.println("Some jobs are not terminated");
        // 모든 Task를 강제 종료
        executor.shutdownNow();
    }
}
```
{% endcode %}

* 32 Line: 작업이 모두 완료 될 때 까지 2초 대기 설정 (executor.awaitTermination() 함수 사용)
* 8\~12 Line: 3초 대기&#x20;
* 결과: 작업이 모두 완료 될 때까지 2초 대기 후 초과하면 조건식이 만족하지 않아 모든 Task를 종료한다.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

*   8\~12 Line: 1초 대기로 변경 하면 다음과 같은 결과를 얻는다.\


    <figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>


*   32 \~ 38 Line: 다음과 같이 여러 가지 상황을 만들어서 확인해 보자\


    ```java
    // 작업이 모두 완료될 때까지 대기
    if (executor.awaitTermination(2, TimeUnit.SECONDS)) {
        System.out.println("All jobs are terminated");
    } else {
        System.out.println("Some jobs are not terminated");
        // 모든 Task를 강제 종료
        // executor.shutdownNow();
    }

    System.out.println("executor.isTerminated : " + executor.isTerminated());
    // 모든 Task가 종료 될 떄 까지 대기 
    while (!executor.isTerminated()) { }
    System.out.println("executor.isTerminated : " + executor.isTerminated());
    executor.close();
    ```



