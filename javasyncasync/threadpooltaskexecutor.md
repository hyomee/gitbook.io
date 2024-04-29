# ThreadPoolTaskExecutor

자바에서 동시성 프로그램을 만들기 위해서는 Runnable객체를 만든 다음 Thread 객체륽 만들어 실행을 해야 하는데 이것은 Thread를 만드는데 비용이 많이들고 새 Thread Instance를 만들기 때문에 성능이 저하된다. 이런 이유로 Thread Pool을 만들어 Thread Instance에 대한 문제를 해결하고자 한다,

## 1. Thread Pool 이란

<figure><img src="../.gitbook/assets/image (216).png" alt=""><figcaption></figcaption></figure>

동시에 여러 작업을 효율적으로 실행 및 관리하기 위해 서버에서 만드는 스레드의 모음으로 새로운 Thread를 만들지 않고 미리 생성된 Thread를 Thread Pool에서 가지고와 재사용하는것으로 성능과 리소스 관리에 도움이 된다. 특히 병려처리는Thread 개수가 증가하여 Thread 생성과 스케줄링으로 인해 CPU, 메모리 사용량이 늘어나는 문제가 있으므로 Thread Pool을 사용해야 한다.

자바에서 스레드 풀을 생성하고 사용할 수 있도록 java.util.concurrent패키지에서 ExecutorService인터페이스와 Executors클래스를 제공하고 있으며 다음과 같은 기능을 제공한다.

* newFixedThreadPool(int nThreads): 고정된 개수의 쓰레드풀을 생성한다.
* newCachedThreadPool(): 필요한 만큼 쓰레드풀을 생성하고 이미 생성된 쓰레드를 재활용한다.
* newScheduledThreadPool(int corePoolSize): 일정 시간 뒤에 실행되거나 주기적으로 수행되는 작업을 처리할 수 있는 쓰레드풀
* newSingleThreadExecutor(): 쓰레드 1개인 ExecutorService를 리턴합니다. 싱글 쓰레드에서 동작해야 하는 작업을 처리할 때 유용하다

스레드 풀을 종료할 때는 shutdown(), shutdownNow(), awaitTermination() 메소드를 사용합니다. 작업 생성과 처리 요청은 execute(Runnable command) 또는 submit(Runnable task) 메소드를 통해 이루어지며 작업은 Runnable 또는 Callable 구현 클래스로 표현하며, 작업 처리 결과를 받기 위해 submit 메소드를 사용할 수 있다.

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

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

*   8\~12 Line: 1초 대기로 변경 하면 다음과 같은 결과를 얻는다.\


    <figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>


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
