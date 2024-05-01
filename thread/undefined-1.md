# 스레드 오류 유형

자바에서 멀티 스레딩은 개발자에게 스레드가 서로 간섭하지 않고 예기치 않은 결과를 생성하지 않도록 동기화를 관리해야 하는 책임이 주어지며 다음과 같은 유형에 대해서 개발시 고려 해야 한다.

* 경합 조건  (Race Conditions)
* 교착 상태 (Deadlocks)
* 권한 실패 (Starvation)
* 데이터 불일치  (Data Inconsistency)

## 1. 경합 조건  (Race Conditions)

스래드 실행이 실행 순서에 따라 예측할 수 없는 결과와 데이터 손상이 발생하는 현상&#x20;

{% code lineNumbers="true" %}
```java
public class RaceConditionMain {
    private static int counter = 0;

    public static void increment() {
        for (int i = 0; i < 10000; i++) {
            counter++;
        }
    }

    public static void main(String[] args) {

        Thread thread1 = new Thread(RaceConditionMain::increment);
        Thread thread2 = new Thread(RaceConditionMain::increment);

        thread1.start();
        thread2.start();

        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Counter: " + counter);

    }
}
```
{% endcode %}

* 결과 : Counter: 13001 \[ 실행할 때 마다 변경됨 ]
* 원인:  Thread1, thread2 가 동시에 수행이 되면서 counter 변수를 공유하고 있어 서로 경합이 발생하여 어떤 값을 예측할 수 없다.&#x20;
* 해결: **synchronized** 키워드를 사용하여 경합이 발생 하지 않게 한다. 개발자는 공유 데이터를 사용하는 경우 공유 데이터에 대한 분석을 하여 동기화 메서드나 블럭과 같은 기능을 사용하여 공유 데이터를 보호해야 한다.

```java
public static synchronized void increment() {
    for (int i = 0; i < 10000; i++) {
        counter++;
    }
}
```

* 결과: Counter: 20000

## 2. 교착 상태 (Deadlocks)

두 개 이상의 스레드가 영구적으로 차단되어 각 스레드가 다른 스레드가 잠금을 해제할 때까지 대기할 때 발생하는 현상이다.

```java
public class DeadlockMain {
    private static final Object lockObj1 = new Object();
    private static final Object lockObj2 = new Object();

    public static void main(String[] args) {

        Thread thread1 = new Thread(() -> {

            synchronized (lockObj1) {
                System.out.println("Thread 1: lockObj1 Start ");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.println("Thread 1: lockObj2 wait");
                synchronized (lockObj2) {
                    System.out.println("Thread 1: lockObj1, lockObj2 Process");
                }
            }
        });

        Thread thread2 = new Thread(() -> {

            synchronized (lockObj2) {
                System.out.println("Thread 2: lockObj2 Start");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.println("Thread 2: lockObj1 wait");
                synchronized (lockObj1) {
                    System.out.println("Thread 2: lockObj2, lockObj1 Process");
                }
            }

        });

        thread1.start();
        thread2.start();

    }
}
```

* 결과&#x20;

```
Thread 1: lockObj1 Start 
Thread 2: lockObj2 Start
Thread 2: lockObj1 Wait
Thread 1: lockObj2 Wait
```

* 문제:  Thread 1은 lockObj1 를 가지고 있고 lockObj12를 기다리고 Thread 2은 lockObj2 를 가지고 있고 lockObj11를 기다리게 되면 두 Thread 모두 진행할 수 없는 교착상태가 발생 한다.&#x20;
* 해결:  교착상태를 방지하기 위해서는 동일한 순서로 잠금를 획득햐어 한다. 즉 synchronized가 필요한 경우 Thread는 일관된 순서를 사용해야 한다. Thread 2를 Thread 1과 동일한 순서로 수정 한다.

```java
 Thread thread2 = new Thread(() -> {

    synchronized (lockObj1) {
        System.out.println("Thread 2: lockObj1 Start");
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Thread 2: lockObj2 wait");
        synchronized (lockObj2) {
            System.out.println("Thread 2: lockObj1, lockObj2 Process");
        }
    }

});
```

* 결과

```
Thread 1: lockObj1 Start 
Thread 1: lockObj2 Wait
Thread 1: lockObj1, lockObj2 Process
Thread 2: lockObj1 Start
Thread 2: lockObj2 wait
Thread 2: lockObj1, lockObj2 Process
```

* Thread 1 에 의해서 lockObj1 이 잠겨 있어서 Thread 2는 대기 하게 되고 Thread 1이 완료 후 Thread 2가 실행하게 된다.
* 교착스상태는레드 간의 순환 종속성을 방지하기 위해 항상 일관된 순서로 잠금을 획득하므로 해결 할 수 있다.

## 3.   권한 실패 (Starvation)

스레드가 공유 리소스에 대한  액세스 권한을 얻을 수 없고 진행할 수 없을 때 발생하는 것으로 우선 순위가 낮은 스레드가 우선 순위가 높은 스레드에 의해 지속적으로 선점될 때 발생한다.

```java
public class StarvationMain {

    private static final Object lock = new Object(); 

    public static void main(String[] args) {

        Thread highThread = new Thread(() -> {
            while (true) {
                synchronized (lock) {
                    System.out.println("높은 순위 highThread 실행");
                }
            }

        });

        Thread lowThread = new Thread(() -> {
            while (true) {
                synchronized (lock) {
                    System.out.println("낮은 순위 lowThread 실행");
                }
           }
        }); 


        highThread.setPriority(Thread.MAX_PRIORITY);
        lowThread.setPriority(Thread.MIN_PRIORITY);

        highThread.start();
        lowThread.start();


    }
}
```

* 결과:

```
높은 순위 highThread 실행
높은 순위 highThread 실행
높은 순위 highThread 실행
높은 순위 highThread 실행
```

* 문제: lowThread는 highThread 보다 우선 순위가 낮기 때문에 우선 순위가 높은 Thread가 겹합에 의해 우선 처리 되어서 발생 한다.
* 해결: 스레드 우선 순위를 조정을 위해 **ReentrantLock**를 사용해서 해결할 수 있다**.**

```java
public class StarvationMain {

    private static final Lock lock = new ReentrantLock(true);

    public static void main(String[] args) {  
    
        Thread highThread = new Thread(() -> {
            while (true) {
                lock.lock();
                try  {
                    System.out.println("높은 순위 highThread 실행");
                } finally {
                    lock.unlock();
                }
            }

        });

        Thread lowThread = new Thread(() -> {
            while (true) {
                lock.lock();
                try  {
                    System.out.println("낮은 순위 lowThread 실행");
                } finally {
                    lock.unlock();
                }
            }
        });

        highThread.setPriority(Thread.MAX_PRIORITY);
        lowThread.setPriority(Thread.MIN_PRIORITY);

        highThread.start();
        lowThread.start();


    }
}
```

* 결과:

```
높은 순위 highThread 실행
높은 순위 highThread 실행
높은 순위 highThread 실행
낮은 순위 lowThread 실행
높은 순위 highThread 실행
낮은 순위 lowThread 실행
높은 순위 highThread 실행
낮은 순위 lowThread 실행
```

**ReentrantLock**은 가장 오래 대기하는 스레드가 잠금을 가져오도록 조정하도록 한다.

## 4. 데이터 불일치  (Data Inconsistency)

여러 스레드가 적절한 동기화 없이 공유 데이터에 액세스할 때 발생하며, 이로 인해 예기치 않은 잘못된 결과가 발생하는 것으로 **synchronized** 키워드를 통해서 해결할 수 있다.

예제:  자바스레드안전성 / 자바 동기화 사용 (synchronized)

## 5. 자바에서 멀티 스레드 개발시 고려 사항&#x20;

* **경합/교착 상태 방지:** 멀티 스레드를 사용할 때는 스레드가 순서적으로 작동이 하도록 하여 공유 리소스에 대한 공유를 하지 말도록 **synchronized** 키워드를 적절히 사용하도록 하여 동시에 접근 하지 않도록 헤야 하며 wait() 및 notify()는 올바르게 사용하지 않으면 교착 상태가 발생 하므로 동기적으로 코드를 작성하고 wait() 및 notify()는 사용하지 말아야 한다.
* **Thread Pool사용 :** 자바에서 스레드는 자원 소모가 많은 작업으로 스레드 풀을 사용하여 오버해드를 줄일  수 있다. 스레드 풀을 사용할 때는 풀 크기는 불필요한 스레드 생성을 피하면서 최대 부하를 처리하기 위해 풀의 크기를 적절하게 조정해야 한다.
* &#x20;**synchronized** **사용**:  교착 상태가 발생하지 않도록 다른 스레드에서 먼저 액세스할 가능성이 가장 높은 개체를 기반으로 해야 한다. 또한 동기화 블록은 최대 성능과 확장성을 위해 가능한 한 작게 유지해야 하므로 비용이 많이드는 I/O 호출 같은 기능에는 사용하지 많아야 한다.
* **휘발성 필드 사용:** 휘발성 필드를 사용하는 경우는 volatile 키워드를 사용하여 모든 스레드가 가장 최근 값을 보도록  휘발성 필드에서 읽을 때 모든 읽기는 모든 스레드에서 가장 최근의 쓰기를 반환하도록 보장해야 한다,
* **스레드 지역 변수 사용 금지:** 반드시 필요한 경우가 아니면 스레드 로컬 변수를 사용하지 말아야 한다.
* **Executor 프레임워크 사용:** 자바 Thread 작업은 스레드 생성 및 삭제와 같은 스레드 실행 관리와 관련된 복잡성이 너무 많고 리소스 사용(CPU, 메모리등)에 대해서 스레드 수등을 고려하고 작업을 해야 하는데 Executor Framework를 사용히여 단순화할 수 있다. _Executors_ 클래스에서 사용할 수 있는 많은 정적 메서드중 대표적인 메서드는 다음과 같으며 _ExecutorService_ 개체를 반환한다.
  * **newCachedThreadPool():** 필요에 따라 새 스레드를 만들지만 이전에 생성된 사용 가능한 스레드를 빠르게 다시 사용할 수 있고 스레드 풀은 작업 부하에 따라 축소 및 확장될 수 있다.
  * **newFixedThreadPool (int nThreads):** 이 메서드로 만든 스레드 풀에는 전달된 매개 변수에 의해 설정된 고정 크기가 있어 지정된 시간에 풀에는 최대 nThread(스레드 수)가 있을 수 있다.
  * **newSingleThreadExecutor ():**  모든 작업을 실행할 스레드가 하나만 있다. 이 스레드가 예기치 않게 종료되면 새 스레드가 만들어지지만 주어진 시간에 단일 스레드가 있다는 보장된다.
  * _ExecutorService:_ _ExecutorService_ 인터페이스는 _Executor_를 확장하고 스레드의 질서 있는 종료_를 시작하는 shutdown()_ 메서드와 같이 스레드 실행을 관리하는 데 필요한 메서드를 제공하며 스레드 예약을 지원하기 위해 _ExecutorService_를 확장하는 _ScheduledExecutorService_라는 또 다른 인터페이스있다.
* #### 스레드로부터 안전한 라이브러리 를 사용해야 하고 원자성 개체 사용한다.

