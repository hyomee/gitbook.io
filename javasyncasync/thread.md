# 자바 Thread

자바에서 Thread를 생성하는 방법에는 다음과 같이 2가지 방법이 있다,.



## 1.  Thread 클래스 사용

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

## 2. Runnable  인터페이스를 사용

```java
public class DemoRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("DemoRunnable .. Run ...");
    }
}

// run : new Thread(new DemoRunnable()).start();
```

## 3. 람다 표현식&#x20;

```java
Runnable subTask = () ->
{
    System.out.println("subTask started...");
};

//  run  new Thread(subTask).start();
```

## 4. Thread 시작

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

