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

### 4-1. _Thread.start()_&#x20;

### 4-2. _ExecutorService_&#x20;

### 4-3. _ScheduledExecutorService_

### 4-4. _CompletableFuture_&#x20;

### 4-5. 가상 스레드로 실행



