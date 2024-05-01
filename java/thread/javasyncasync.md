# 자바 동기/비동기

작업 A, 작업 B가 있는 경우 **동기(sync)**작업은   작업 A가 실행되고 난 후 작업B을 수행하는 것으로 순차적으로 실행되는 작업을 의미하여 **비동기(async)**작업은 작업A, 작업B가 관계 없이 동시에 수행되는 것을 의미한다. Java 코드를 사용하여 살펴 본다.

## 1. 일반 적인 동기 코드

```java
@Component
public class FutureCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("작업1 시작");
        String rnTaskA = taskA();
        System.out.println("작업1 종료 : " + rnTaskA);
        System.out.println("작업2 시작");
        String rnTaskB = taskB();
        System.out.println("작업2 종료 : " + rnTaskB );
    }

    private String taskA() throws InterruptedException {
        Thread.sleep(1000);
        return "Task A ";
    }

    private String taskB() throws InterruptedException {
        Thread.sleep(1000);
        return "Task B ";
    }
}
```

*   실행 결과 \


    <figure><img src="../../.gitbook/assets/image (214).png" alt=""><figcaption></figcaption></figure>

작업 A(taskA()) , 작업 B(tackB())가 순차적으로 실행되는 것을 확인 할 수 있다.

## 2. Thread로 변경 ( Runnable )



{% code lineNumbers="true" %}
```java
@Component
public class FutureCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        ResultVo resultVo = new ResultVo();

        Thread threadTaskA = new Thread(() -> {
            try {
                resultVo.taskA = taskA();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });

        Thread threadTaskB = new Thread(() -> {
            try {
                resultVo.taskB = taskB();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });

        System.out.println("작업1 시작");
        threadTaskA.start();
        System.out.println("작업2 시작");
        threadTaskB.start();
        threadTaskA.join();
        threadTaskB.join();

        System.out.println("작업1 종료 : " + resultVo.taskA);
        System.out.println("작업2 종료 : " + resultVo.taskB );
    }

    private class ResultVo {
        private String taskA;
        private String taskB;
    }

    private String taskA() throws InterruptedException {
        Thread.sleep(1000);
        return "Task A ";
    }

    private String taskB() throws InterruptedException {
        Thread.sleep(1000);
        return "Task B ";
    }
}
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (215).png" alt=""><figcaption><p>Thread 실행 결과</p></figcaption></figure>

* 7 \~ 21 Line: Thread 생성&#x20;
* 39, 44 Line: TaskA, TaskB 작업 선언&#x20;
* 23\~31 Line: Thread로 생성한 작업 실행 , join 메서드를  사용해서 작업이 종료 될 떄 까지 대기 후 출력 한다.

## 3. Future API 적용

Future에 작업을 등록할 때, 등록되는 작업이 Runnable인지 Callable인지 잘 확인해야 한다. Runnable은 아무것도 리턴하지 않기 때문에 get()을 호출했을 때 null이 나올 수 있다.&#x20;

```java
public void futureApi() throws ExecutionException, InterruptedException {

    ExecutorService executorService = Executors.newFixedThreadPool(2);

    Future<String> threadTaskA = executorService.submit(()-> taskA());
    Future<String> threadTaskB = executorService.submit(()-> taskB());

    System.out.println( threadTaskA.get() + threadTaskB.get());

    executorService.shutdown();
}
```

## 4. FutureTask API 적용

FutureTask는 Runnable과 Future를 합친 RunnableFuture를 상속한 클래스의 객체인데, 둘을 모두 상속한 덕분에 Runnable과 Future의 역할을 객체 하나로 한 번에 수행할 수 있다.&#x20;

```java
public void FutureTask() throws ExecutionException, InterruptedException {

    ExecutorService executorService = Executors.newFixedThreadPool(2);

    FutureTask<String> threadTaskA = new FutureTask<>(()->taskA());
    FutureTask<String> threadTaskB = new FutureTask<>(()->taskB());

    executorService.submit(threadTaskA);
    executorService.submit(threadTaskB);
    
    System.out.println( threadTaskA.get() + threadTaskB.get());

    executorService.shutdown();
}
```

## 5. CompletableFuture API 적용

```java
public void CompletableFutureTask() throws ExecutionException, InterruptedException {

    ExecutorService executorService = Executors.newFixedThreadPool(2);

    CompletableFuture<String> threadTaskA = CompletableFuture.supplyAsync(() -> {
        try {
            return taskA();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }, executorService);

    CompletableFuture<String> threadTaskB = CompletableFuture.supplyAsync(() -> {
        try {
            return taskB();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }, executorService);



    System.out.println( "ThreadTaskA : " + threadTaskA.get());
    System.out.println( "ThreadTaskB : " + threadTaskB.get());

    executorService.shutdown();
}
```



* runAsync: 반환 값이 없는 경우 비동기 작업 실행 (Runnable 타입을 파라미터로 전달하기 때문에 어떤 결과 값을 담지 않는다.)
* supplyAsync: 반환 값이 있는 경우 비동기 작업 실행 (supplier 타입을 넘기기 때문에 반환 값이 존재)
  * &#x20;get() - Future 인터페이스에 정의된 메소드로 checked exception인 InterruptedException과 ExecutionException을 던지므로 예외 처리 로직이 반드시 필요하다.
  * join() - CompletableFuture에 정의되어 있으며, checked  exception을 발생시키지 않는 대신 unchecked CompletionException이 발생된다.
  * join()을 사용하는 것이 권장



<table><thead><tr><th width="199"></th><th></th></tr></thead><tbody><tr><td>thenApply</td><td><p>반환 값을 받아서 다른 값을 반환해주는 콜백</p><pre class="language-java"><code class="lang-java">CompletableFuture&#x3C;String> cf = CompletableFuture
        .supplyAsync(() -> {
            return "hello world!";})
        .thenApply(s -> {
            return s.toUpperCase();});
System.out.println(cf.join()); // HELLO WORLD!
</code></pre></td></tr><tr><td>thenAccept</td><td><p>반환 값을 받아 처리하고 값을 반환하지 않는 콜백</p><pre class="language-java"><code class="lang-java">CompletableFuture&#x3C;Void> cf = CompletableFuture
        .supplyAsync(() -> {
            return "hello world!";})
        .thenAccept(System.out::println); 

cf.join(); // hello world! 
</code></pre></td></tr><tr><td>thenRun</td><td><p>Runnable을 파라미터로 받으며, 반환 값을 받지 않고 그냥 다른 작업을 처리하고 값을 반환하지 않는  콜백</p><pre class="language-java"><code class="lang-java">CompletableFuture&#x3C;Void> cf = CompletableFuture
        .supplyAsync(() -> {
            return "hello world!";})
        .thenRun(System.out::println);

cf.join(); // hello world! 
</code></pre></td></tr><tr><td>thenApplyAsync</td><td><p>별도의 스레드에서 비동기적으로 실행</p><pre class="language-java"><code class="lang-java">CompletableFuture&#x3C;Void> cf = CompletableFuture
        .supplyAsync(() -> {
            return "hello world!";})
        .thenRunAsync(System.out::println);

cf.join(); // hello world! 
</code></pre></td></tr><tr><td>thenRunAsync</td><td><p>앞선 계산의 결과와 상관없이 주어진 작업을 별도의 스레드에서 비동기적으로 실행</p><pre class="language-java"><code class="lang-java">CompletableFuture&#x3C;Void> cf = CompletableFuture
        .supplyAsync(() -> {
            return "hello world!";})
        .thenRunAsync(System.out::println);

cf.join(); // hello world! 
</code></pre></td></tr><tr><td>thenCompose</td><td>두 작업을 이어서 실행하도록 조합하며, 앞선 작업의 결과를 받아서 사용</td></tr><tr><td>thenCombine</td><td>각 작업을 독립적으로 실행하고, 모두 완료되었을 때 결과를 받아서 사용</td></tr><tr><td>allOf</td><td>여러 작업들을 동시에 실행하고, 모든 작업 결과에 콜백을 실행</td></tr><tr><td>anyOf</td><td>여러 작업들 중에 가장 빨리 끝난 하나의 결과에 콜백을 실행</td></tr><tr><td>exceptionally</td><td>발생한 에러를 받아서 예외를 처리</td></tr><tr><td>handle</td><td>(결과값, 에러)를 반환받아 에러가 발생한 경우와 아닌 경우 모두를 처리</td></tr></tbody></table>
