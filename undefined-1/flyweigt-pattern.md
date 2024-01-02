# Flyweigt Pattern

Flyweight Pattern은  객체를 사용 할 때마다 new 로 객체를 인스턴스 화 하여 생성 하지 않고 한번 생성한 객체를 이용하여 공유하는 패턴으로 구조적 패턴에 속한다. 즉 인스턴스의 생성을 최소화 하여 메모리 사용을 절약할 수 있다

<figure><img src="https://blog.kakaocdn.net/dn/FJhyo/btrDPkAsEqI/C7mvnaNWApbCq3ir0fC8vk/img.png" alt=""><figcaption></figcaption></figure>

* client : 실행을 트리거 하는 객체
* FlyweightFatory : Flyweight 객체를 생성하기 위한 Factory&#x20;
* Flyweight : 재사용하고자 하는 객체

***

예제&#x20;

<figure><img src="https://blog.kakaocdn.net/dn/pjPNr/btrDRiVJptP/gFPTJrzQeYXwrVVxDvX8L1/img.jpg" alt=""><figcaption></figcaption></figure>

1\. 재 사용하고자 하는 객체&#x20;

```java
public interface Flyweight {
  String operation(int extrinsicState);
}

public class Flyweight1 implements Flyweight{
  private String intrinsicStatus;

  public Flyweight1(String intrinsicStatus){
    this.intrinsicStatus = intrinsicStatus;
  }

  @Override
  public String operation(int extrinsicState) {
    return String.format("Flyweight1 내부상태(intrinsicStatus) = %s, 외부 상태 = %d ",
            intrinsicStatus,
            extrinsicState);
  }
}
```

2\. Flyweight 를 생성 하기 위한 Factory

```java
public class FlyweightFactory {

  // 공유 pool
  private Map<String, Flyweight> flyweightMaps = new HashMap<String, Flyweight>();

  public Flyweight getFlyweight(String key) {
    if (flyweightMaps.containsKey(key)) {
      System.out.println("\n 공유 객체 flyweight key = " + key);
      return flyweightMaps.get(key);
    } else {
      System.out.println("\n 생성 객체 flyweight  key = " + key);
      Flyweight flyweight = new Flyweight1(key);
      flyweightMaps.put(key, flyweight);
      return flyweight;
    }
  }

  public int getSize() {
    return flyweightMaps.size();
  }

}
```

3\. Client&#x20;

```java
public class Client {
  public static void main(String[] args) {

    // Factory 생성
    FlyweightFactory flyweightFactory = new FlyweightFactory();

    Flyweight flyweight0 = flyweightFactory.getFlyweight("A");
    System.out.println(flyweight0.operation(100));

    Flyweight flyweight1 = flyweightFactory.getFlyweight("A");
    System.out.println(flyweight1.operation(200));

    Flyweight flyweight2 = flyweightFactory.getFlyweight("B");
    System.out.println(flyweight2.operation(300));

    System.out.println(String.format("\n *** 생성된 Flyweight1 갯수 %d ***",
            flyweightFactory.getSize()));

    System.out.println(String.format("\n *** flyweight0 주소 = %s ***",
            System.identityHashCode(flyweight0)));

    System.out.println(String.format("\n *** flyweight1 주소 = %s ***",
            System.identityHashCode(flyweight1)));

    System.out.println(String.format("\n *** flyweight2 주소 = %s ***",
            System.identityHashCode(flyweight2)));
  }
}
```

결과 : flyweight0 , flyweight1의 주소가 같은 것을 확인 할 수 있다,

> 생성 객체 flyweight key = A\
> Flyweight1 내부상태(intrinsicStatus) = A, 외부 상태 = 100\
> \
> &#x20;공유 객체 flyweight key = A Flyweight1\
> 내부상태(intrinsicStatus) = A, 외부 상태 = 200\
> \
> &#x20;생성 객체 flyweight key = B Flyweight1\
> 내부상태(intrinsicStatus) = B, 외부 상태 = 300\
> \
> \*\*\* 생성된 Flyweight1 갯수 2 \*\*\*\
> \
> \*\*\* flyweight0 주소 = 687780858 \*\*\*\
> \*\*\* flyweight1 주소 = 687780858 \*\*\*\
> \*\*\* flyweight2 주소 = 1734161410 \*\*\*

&#x20;

싱클톤 패턴과 차이점&#x20;

싱클톤 패턴은 가변(mutable)으로 Application내에 클래스를 하나의 인스턴스로 제한하는 것입니다. 클래스를 하나의 인스턴스로 제한한다는 것은 한번만 메모리 할당 하고 그 이후엔 그 인스턴스를 받아서 사용하는 패턴으로 꼭 하나만 존재하고자 할때 사용한다. 그러나 Flyweight는 불변(immutable)로 여러 종류의 단 하나를 가질 수 있다 ( 예: 사과, 배는 과일이다 )

```java
  static void flyweightAsSingleton() {

    FlyweightFactory flyweightFactory01 = new FlyweightFactory();
    FlyweightFactory flyweightFactory02 = new FlyweightFactory();

    System.out.println(String.format("\n *** flyweightFactory01 주소 = %s ***",
            System.identityHashCode(flyweightFactory01)));
    System.out.println(String.format("\n *** flyweightFactory02 주소 = %s ***",
            System.identityHashCode(flyweightFactory02)));

    flyweightFactory01.setStr("안녕");
    System.out.println(String.format("\n flyweightFactory01  str = %s \n flyweightFactory02  str = %s",
            flyweightFactory01.getStr(), flyweightFactory02.getStr()));

	// 싱글톤
    FlyweightSingletonFactory flyweightSingletonFactory01 = FlyweightSingletonFactory.getInstance();
    FlyweightSingletonFactory flyweightSingletonFactory02 = FlyweightSingletonFactory.getInstance();
    System.out.println(String.format("\n *** flyweightSingletonFactory01 주소 = %s ***",
            System.identityHashCode(flyweightSingletonFactory01)));
    System.out.println(String.format("\n *** flyweightSingletonFactory02 주소 = %s ***",
            System.identityHashCode(flyweightSingletonFactory02)));

    flyweightSingletonFactory01.setStr("안녕");
    System.out.println(String.format("\n flyweightSingletonFactory01  str = %s \n flyweightSingletonFactory02  str = %s",
            flyweightSingletonFactory01.getStr(), flyweightSingletonFactory02.getStr()));

  }
```

> \*\*\* flyweightFactory01 주소 = 1364614850 \*\*\*\
> \*\*\* flyweightFactory02 주소 = 1211076369 \*\*\*\
> \
> flyweightFactory01 str = 안녕\
> flyweightFactory02 str = null \*\*\* \*\
> \
> \*\* flyweightSingletonFactory01 주소 = 459296537 \*\*\*\
> \*\*\* flyweightSingletonFactory02 주소 = 459296537 \*\*\*\
> flyweightSingletonFactory01 str = 안녕\
> flyweightSingletonFactory02 str = 안녕 \*\*\*

flyweightSingletonFactory는 객체가 한 번만 생성 되므로 동일 주소를 가지고 맴버 변수의 값이 변경 되면 공유 된다.\
\-멀티 쓰레드에서 주의 필요 (synchronized 를 이용해서 thread-safe 해야한다.)&#x20;

&#x20;

전체 소스 : [https://github.com/hyomee/javapattern/tree/main/Pattern/src/main/java/Flyweight](https://github.com/hyomee/javapattern/tree/main/Pattern/src/main/java/Flyweight)



참고  : [https://en.wikipedia.org/wiki/Flyweight\_pattern#Immutability\_.26\_Equality\
\
](https://en.wikipedia.org/wiki/Flyweight\_pattern#Immutability\_.26\_Equality)
