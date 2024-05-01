# Thread

자바에서 쓰레드는 프로세스 내에서 실행되는 작업 단위로 JVM(Java Virtual Machine)에 의헤 스케줄되며 코드 블록으로 구성된다. 자바는 Main Thread가 main() 메소드를 실행하면서 시작된다, 즉 Main Thread안에서 Multi Thread는 필요에 작업 Thread를 만들어 병렬로 코드를 실행할 수 있다.

## 1. 자바 스레드 라이프 사이클

<figure><img src="../../.gitbook/assets/image (217).png" alt=""><figcaption><p>Java Thread LifreCycle</p></figcaption></figure>

*   New Thread: 클래스를 인스턴스화하거나 인터페이스를 구현하고 인스턴스에 전달하여 새 스레드 생성\


    ```java
    Thread newThread = new Thread(() -> {
          System.out.println("새로운 스레드 생성);
    });
    ```


*   Runnable/Running: start() 메서드를  통해 시작하며  Runnable 상태의 스레드는 다른 스레드와 동시에 실행될 수 있고 Runnable 상태의 스레드는 스케줄러가 프로세서 시간을 할당할 때 Running 상태로 전환된다. ( run() )\


    ```java
    runnableThread.start();
    System.out.println("스레드 상테   :" + runnableThread.getState());

    ================================================================================
    public class HelloWorldRunnableExample implements Runnable {
        public void run() {
            System.out.println("Hello from a thread!");
        }
    }
    ```


*   Waiting: 여러 가지 이유로 차단되거나 대기 상태로 전환 될 수 있으며 매소드를 사용하여 스레드를 명지적으로 일시중지 할 수 있다 (Thread.sleep(), Object.wait()) \


    ```java
     Thread.sleep(3000);
    ```


*   Dead: 스레드는 메서드 실행이 완료되거나 처리되지 않은 예외가 발생할 때 **Terminated** 상태로 전환된다.\
    \


    ```java
    Thread newThread = new Thread(() -> {
      System.out.println("새로운 스레드 생성"); // Thread is executing.
    });
     
    newThread .start();
        
    try {
      terminatedThread.join();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
        
    System.out.println("Is thread newThread ? " + !newThread .isAlive());

    /* 결과 
    새로운 스레드 생성
    thread terminated? true
    */
    ```



자바에서 Thread를 생성하는 방법에는 다음과 같이 2가지 방법이 있다.

* Thread 클래스로부터 직접 생성: Thread 클래스 객체를 생성한 후 start() 메서드를 통해 다른 스레드에서 할 작업을 할당한다.&#x20;
* Runnable 인터페이스를 구현한 클래스 객체로 생성: 익명 구현 객체를 만들어 간단하게 실행할 수 있다.

### 1-1.  Thread 클래스 사용

서브 클래스(상속)으로 실행하려는 클래스를 지정하고 run() 메서드를 구현해야 한다.

```java
public class DemoThread extends Thread{
    public DemoThread() {
        super("DemoThread");
    }
    public void run() {
        System.out.println("DemoThread .. Run ...");
    }
}

// run : new DemoThread().start()
```

### 1-2. Runnable  인터페이스를 사용

스레드에서 실행하려는 클래스를 지정하는데 사용되며 run() 메서드를 제정의 해야 한다.&#x20;

```java
public class DemoRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("DemoRunnable .. Run ...");
    }
}

// run : new Thread(new DemoRunnable()).start();
```

### 1-3. 람다 표현식&#x20;

```java
Runnable subTask = () ->
{
    System.out.println("subTask started...");
};

//  run  new Thread(subTask).start();
```

## 2. Thread 시작

### 4-1. _Thread.start()_

가상 Thread를 제외한 새로운 Thread의 시작애 사용하는 메서드로 스케줄러에 Thread를 등록하고 리소스 할당과 같은 수행하는데 필요한 모든 활동을 담당한다.

```java
new DemoThread().start()
new Thread(new DemoRunnable()).start();
new Thread(subTask).start();
```

### 4-2. _ExecutorService_&#x20;

이미 생성된 Thread를 사용하기 위해서 제공 Thread Pool 생성 및 관리의 핵심이 되는 인터페이스로 _**Runnable**_ 또는 _**Callable**_ 작업을 사용하여 Thread Pool에서 하나의 Thread를 사용하여 작업을 실행한다.

```java
Runnable subTask = () ->
{
    System.out.println("subTask started...");
};

ExecutorService executor = Executors.newFixedThreadPool(4);
executor.execute(subTask);

// 람다 표현식으로 변경 
executor.execute(()-> {
    System.out.println("executor.execute ..");
});
```

### 4-3. _ScheduledExecutorService_

_일정 시간 후에 작업을 실행하거나 주기적으로 작업을 실해아흔 경우 사용하는 인터페이스로 다음 예제는 3초 후  예약된 작업이 수행 되는 예제이다._

{% code lineNumbers="true" %}
```java
public void scheduledExecutorServiceDemo() 
     throws ExecutionException, InterruptedException {
    
    // ScheduledExecutorService ThreadPool 생성 
    ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
    System.out.println("Thread is  : " + LocalTime.now());
    
    ScheduledFuture<String> result = executor.schedule(() -> {
        System.out.println("Thread is ScheduledFuture Start :" + LocalTime.now());
        Thread.sleep(2000);
        System.out.println("Thread is ScheduledFuture End :" + LocalTime.now());
        return "completed";
    }, 3, TimeUnit.SECONDS);

    System.out.println(result.get());

    if (!result.state().equals(Future.State.RUNNING)) {
        System.out.println("Thread is executing the job :" + LocalTime.now());
        executor.shutdownNow();
    }
}
```
{% endcode %}

### 4-4. _CompletableFuture_&#x20;

Java 8에 도입된 Future API의 확장으로 Future, CompletionStage\<T>인터페이스의 구현체로  여러 _Future를_ 생성, 연결 및 결합하는 방법을 제공한다. runAsync()는반환  결과 없이 실행 할 떄 반환 값이 있을 때는  supplyAsync()를 사용한.

```java
public void completableFutureDemo() 
    throws ExecutionException, InterruptedException {
    // 반환 값이 없는 runAsync
    CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
        System.out.println("Thread is runAsync executing");
    });

    // 반환 값이 있는 supplyAsync
    CompletableFuture<String> result = CompletableFuture.supplyAsync(() -> "Thread is executing");

    // 반환 값 출력 
    System.out.println(result.get());
}
```

### 4-5. 가상 스레드로 실행

java 19이후 추가된 것으로 높은 처리량의 동시 애플리케이션을 작성하는 데 도움이 되는 JVM 관리 경량 스레드이다.

```java
    public void virtualThreadDemo() {
        Runnable runnable = () -> System.out.println("가상 쓰레드 Runnable");
        Thread.startVirtualThread(runnable);


//        Thread.startVirtualThread(() -> {
//            //Code to execute in virtual thread
//            System.out.println("Inside Runnable");
//        });

        Thread.Builder builder = Thread.ofVirtual().name("Virtual-Thread");
        builder.start(runnable);
    }
```

