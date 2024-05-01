# 자바 스레드 안전성

**동시성이란** 여러 작업을 동시에 수행하는 것으로 여러 프로그램 또는 프로그램의 여러 부분을 병렬로 실행하는 기능을 의미한다.

자바는 처음 부터 동시 프로그래밍을 지원하도록 설계되어 Java 프로그래밍 언어와 Java 클래스 라이브러리 전반에 걸쳐 기본 동시성을 지원한다.&#x20;

* **프로세스**: 실행중에 있는 프로그램으로 최소 하나의 Thread를 가지고 있으면 Thread단위로 스케불링을 한다.실행을 위해서 메모리 할당이 이루어지고, 할당된 메모리 공간으로 바이너리 코드가 올라가게 된는데 이 순간 부터 프로세스라 한다.
  * 프로세스 간의 통신은 파이프와 소켓을 포함하는 _IPC(Inter Process Communication)_ 리소스를 통해 이루어 진다. &#x20;
  * 대부분의 JVM(Java Virtual Machine) 구현은 단일 프로세스로 실행되지만 Java 애플리케이션은 ProcessBuilder **오**브젝트를 사용하여 추가 프로세스를 작성할 수 있다.
* 스레드: 프로세스 내에 존재하며, 이는 모든 프로세스에 하나 이상의 스레드가 있음을 의미한다.
  * 프로세스 내의 모든 스레드는 메모리 및 열려 있는 파일을 포함한 리소스를 공유하므로 스래드를 효율적으로 관리하지 않으면 문제가 될수 있다.

자바에서 스레드 안전성을 지키기 위한 방법

* 동기화 사용 (synchronized)&#x20;
* 원자 변수 사용 (**Atomicxxx**)
* volatile 키워드 사용
* final 키워드를 사용

## 1. 자바 동기화 사용 (synchronized)&#x20;

한 번에 하나의 스레드만 실행 하여 해당 작업이 완료 될 떄까지 다른 작업을 하지 못하게 하는 것으로 여러 스레드가 동시에 동일한 리소스에 액세스하는 것을 방지하여 불일치 문제를 해결할 수 있다. 이때 사용하는 키워드가 **synchronized** 키워드이다.

{% code lineNumbers="true" %}
```java
public class FourArithmetic{
  void add(int num) {
    // Thread Instance 생성
    Thread t = Thread.currentThread();
    for (int i = 1; i <= 5; i++) {
      System.out.println(t.getName() + " : " + (num + i));
    }
  }
}

public class FourArithmetic2 extends Thread {
  FourArithmetic math= new FourArithmetic();
  public void run() {
    math.add(10);
  }
}


public class DemoApp{

  public static void main(String[] args) {
    FourArithmetic2 maths = new FourArithmetic2 ();
    
    Thread t1 = new Thread(maths);
    Thread t2 = new Thread(maths);
    
    t1.setName("Thread 1");
    t2.setName("Thread 2");
    
    t1.start();
    t2.start();
  }
}
```
{% endcode %}

위 코드를 실행하면 다음과 같이 두 스레드가 섞여서 출력되는 것을 확인할 수 있다,

```
Thread 1 : 11
Thread 1 : 12
Thread 1 : 13
Thread 2 : 11
Thread 1 : 14
```

이것을 하나싹 실행 하도록 하기 위헤서 **synchronized** 키워드 사용하는 코드로 변경한다.

{% code lineNumbers="true" %}
```java
public class FourArithmetic{
  synchronized void add(int num) {
    // Thread Instance 생성
    Thread t = Thread.currentThread();
    for (int i = 1; i <= 5; i++) {
      System.out.println(t.getName() + " : " + (num + i));
    }
  }
}
```
{% endcode %}

변경 후 실행하면 다음과 같이  Thread1이 완료 된 후 Thread2가 실행 된다,

```
Thread 1 : 11
Thread 1 : 12
Thread 1 : 13
Thread 1 : 14
Thread 1 : 15
Thread 2 : 11
Thread 2 : 12
Thread 2 : 13
Thread 2 : 14
Thread 2 : 15
```

## 2. 원자 변수 사용 (**Atomicxxx**)

원자 변수는 동기화를 최소화하고 메모리 일관성 오류를 방지하는 개발자가 변수에 대해 원자성 연산을 수행헐 수 있다.

원자 변수: **AtomicInteger**, **AtomicLong**, **AtomicBoolean** , **AtomicReference**

```java

import java.util.concurrent.atomic.AtomicInteger;

public class Counter {
  AtomicInteger counter = new AtomicInteger();
  
  public void increment() {
    counter.incrementAndGet();
  }
}


public class DemoAPP{
  public static void main(String[] args) throws Exception {
  
    Counter c = new Counter();
    
    Thread t1 = new Thread(new Runnable() {
      public void run() {
        for (int i = 1; i <= 1000; i++) {
          c.increment();
        }
      }
    });
    
    Thread t2 = new Thread(new Runnable() {
        public void run() {
          for (int i = 1; i <= 1000; i++) {
            c.increment();
          }
        }
    });
    
    t1.start();
    t2.start();
    
    t1.join();
    t2.join();
    
    System.out.println(c.counter);
  }
}
```

결과 : 2000

## 3. volatile 키워드 사용

**volatile** 키워드: 동시에 여러 스레드에서 개체를 사용할 수 있도록 하는 필드 수정자

```java

public class DemoApp{
  static volatile int int1 = 0, int2 = 0;
  
  static void methodOne() {
    int1++;
    int2++;
  }
  
  static void methodTwo() {
    System.out.println("int1=" + int1 + " int2=" + int2);
  }
  
  public static void main(String[] args) {
  
    Thread t1 = new Thread() {
      public void run() {
        for (int i = 0; i < 5; i++) {
          methodOne();
        }
      }
    };
    
    Thread t2 = new Thread() {
      public void run() {
        for (int i = 0; i < 5; i++) {
          methodTwo();
        }
      }
    };
    
    t1.start();
    t2.start();
  }
}
```

결과: 두 변수는 두 번째 스레드가 값을 출력하기 전에 첫 번째 스레드에 의해 완전히 증가한다.

```
int1=5 int2=5
int1=5 int2=5
int1=5 int2=5
int1=5 int2=5
int1=5 int2=5
```

## 4. final 키워드를 사용

_Final 변수는_  일단 할당되면 객체에 대한 참조가 다른 객체를 가리킬 수 없는 특성을 사용한 코드는 Java에서 항상 스레드로부터 안전하며 에디터에서 오류 발생한다.

```java
public class DemoApp{
    final String aString = new String("즉시");

    void someMethod() {
        aString = "new value"; // 에로 표시 됨 
    }
}
```
