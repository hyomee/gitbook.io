---
description: Inversion of Control
---

# DIP & DI

## 1. 의존성 ( Dpendency )

의존성이란 클래스 A가 클래스 B를 맴버 변수 또는 로컬 변수로 가지고 있거나 혹은 파라미터로 전달되거나 클래스 B의 메서드를 호출하는 것을 말한다.&#x20;

클래스 A가 클레스 B에 의존이 높으면 클래스 B의 변경에 의해 컴파일 오류가 나거나 예상치 못한 동작을 하는 등의 영향을.받게 되는면 결합도가 높다고 이야기 한다. 즉 의존성이 높으면 결합도가 높아져서 수정이 용이 하지 않으며 테스트 범위가 증가 하게 된다.

<figure><img src="../.gitbook/assets/image (57).png" alt="" width="266"><figcaption><p>클래스 A는 B에 의존함</p></figcaption></figure>

이 문제를 해결하기 위한 패턴이 Dependency Inversion Principle (DIP)이다.

## 2. DIP ( Dependency Inversion Principle)

> '고차원 모듈은 저차원 모듈에 의존하면 안된다. 이 모듈 모두 다른 추상화된 것에 의존해야 한다. 추상화 된 것은 구체적인 것에 의존하면 안 된다. 구체적인 것이 추상화된 것에 의존해야 한다.'\
> \- Martin, Robert C. -

즉 의존 관계를 맺을 때 변화하기 쉬운 것에 의존하기보다는, 변화하지 않는 것에 의존하라는 원칙으로 DI(종속성 주입)는 응용 프로그램을 쉽게 관리하고 테스트할 수 있도록 프로그래밍 코드에서 종속성을 제거하는 디자인 패턴입니다. -> 느슨한 결합

참고 : [의존 관계 역전 원칙](https://ko.wikipedia.org/wiki/%EC%9D%98%EC%A1%B4%EA%B4%80%EA%B3%84\_%EC%97%AD%EC%A0%84\_%EC%9B%90%EC%B9%99)

<figure><img src="../.gitbook/assets/image (56).png" alt="" width="266"><figcaption><p>Dependency 제거</p></figcaption></figure>

#### 다음 코드는 리소스를 가져 오는 접근 방법 입니다.

```java
// 1. new 키워드로 resource(A 클래스의 인스턴스)를 직접 가져옴
A obj = new AImpl();   

// 2. 정적 팩토리 메소드getA()를 호출하여 리소스 (A 클래스의 인스턴스)를 가져옴
2. A obj = A.getA();  

// 3. DI (Java Naming Directory Interface)로 리소스를 가져옴
Context ctx = new InitialContext();  
Context environmentCtx = (Context) ctx.lookup("java:comp/env");  
A obj = (A)environmentCtx.lookup("A");  

```

#### 종속성 조회 : 리소스에 대한 접근 방법으로 다음과 같은 문제가 있습니다.

* **긴밀한 결합** 종속성 조회 접근 방식은 코드를 긴밀하게 결합되어  있고 리소스가 변경되면 코드에서 많은 수정을 수행해야 합니다.
* **테스트하기 쉽지 않음** 이 접근 방식은 특히 블랙 박스 테스트에서 응용 프로그램을 테스트하는 동안 많은 문제를 일으킵니다.

### 2-1. 고차원 모듈은 저차원 모듈에 의존하면 안된다

<figure><img src="../.gitbook/assets/image (58).png" alt="" width="375"><figcaption><p>고차원 ( Abstract ),  저차원 ( Class B )</p></figcaption></figure>

Class A는 Abstract를 참고 하지만 Abstract는 Class B를 의존하면 안된다. 즉  추상화 된 것은 구체적인 것에 의존하면 안 된다

### 2-2. 구체적인 것이 추상화된 것에 의존해야 한다

<figure><img src="../.gitbook/assets/image (59).png" alt="" width="375"><figcaption></figcaption></figure>

Class B가 Abstract를 상속(Inherit)하도록 하여 의존(Dependency)를 Inversion한다. 즉 A Interface를 Package A에 정의하고 Package B에서 구현하는 것이 DIP이다.

<figure><img src="../.gitbook/assets/image (60).png" alt="" width="347"><figcaption><p>Dependency Inversion</p></figcaption></figure>

## 3. DI ( Dependency Injection )&#x20;

의존관계를 외부에서 결정하고 주입하는 것이 DI(의존관계 주입으로 Constructor Injection, Interface Injection, Method Injection이 있다.

* **Constructor Injection** : 객체 생성자를 사용한 주입으로 가장 많이 사용한다.
* **Method Injection** :  setter메서드의 파라메터로 주입을 하는 방법이다.
* **Interface Injection** : 인터페이스를 사용하여 Method Injection로 주입하는 방법이다.

## 4. 예제

자동차 판매를 위한 프로그램이 있다고 가정해 본다.

### 4-1. 일반적인 구현 ( 의존 )

```java
// 자동차 판매를 위한 Client 코드 
public class  CarApplication {
  private CarService carService = new CarService();
  public void saleCar(String manufacturer, String carType){
    carService.sale(manufacturer, carType);
  }
}

// 자동차 판매을 위한 Class로 판매하는 로직 수행 
class CarService {
  public void saleCar(String manufacturer, String carType) {
    System.out.println("제조사 : "+manufacturer+ " 차종 ="+carType+ "판매힙니다.");
  }
}
```

#### 4-1-1.  문제점

1. CarApplication에서 CarServie를 초기화하고 있어 종속성이 발생하고 있어 다른 기능을 추가 할려고 하면 CarApplication에 수정이 발생한다.&#x20;
2. 자동차 제조 회사가 추가 되거나 제조사별 판매를 옵션을 변경 하려면 다른 응용 프로그램을 작성해야 한다.

#### 4-1-2. DI 적용

<figure><img src="../.gitbook/assets/image (160).png" alt="" width="375"><figcaption><p>DI 적용</p></figcaption></figure>

**1. 생성자 주입을 통한 DI**

<pre class="language-java"><code class="lang-java"><strong>// 자동차 판매 어플리케이션 
</strong><strong>public class CarApplication {
</strong>  private CarService carService  ;
  
  public CarApplication(CarService carService) {
    this.carService = carService;
  }
  public void saleCar(String manufacturer, String carType){
    carService.sale(manufacturer, carType);
  }
}

// 자동차 판매를 위한 인터페이스 
public interface CarService {
  void sale(String manufacturer, String carType) ;
}

// 현대 자동차 판매를 위한 서비스 
<strong>public class HyundauiCarServiceImpl implements CarService {
</strong>  @Override
  public void sale(String manufacturer, String carType) {
    System.out.println("제조사 : "  +manufacturer + " 차종 : " + carType + " 판매힙니다.");
  }
}
</code></pre>

클라이언트에서 자동차 판매를 위해서 CarApplication을 생성 할 때 재조사별 서비스를 생성하므로 CarApplication은 독립적으로 재사용 할 수 있다.

```java
public class  CarClient {

  public static void main(String[] args){
    // CarApplication 생성 시점에 HyundauiCarServiceImpl 주입 
    CarApplication carApplication = new CarApplication(new HyundauiCarServiceImpl());
    carApplication.saleCar("현대", "그랜저");
  }
}
```

현대 자동차 판매 서비스 생성부분에 CarService를 상속 받은 기아자동차판매 구현 서비스가 있으면 CarApplication 생성시 기아자동차판매 구현 서비스를 생성 하면 CarApplication의 수정 없이 판매를 확장 할 수 있다. 즉 CarApplication는 하나의 독립적인 컴포넌트가 된 것 이다.

**2. Method 주입을 통한 DI**

```java
public class CarApplication {
  private CarService carService  ;
  public void saleCar(CarService carService, String manufacturer, String carType){
    this.carService = carService;
    this.carService.sale(manufacturer, carType);
  }
}
```

saleCar 메서드의 파라메터로 CarService를 주입 받아서 처리 하는 것으로 CarApplication는 하나의 독립적인 컴포넌트가 된다.

```java
public class  CarClient {

  // saleCar 메서드 호출시 HyundauiCarServiceImpl 주입 
  public static void main(String[] args){
    CarApplication carApplication = new CarApplication();
    carApplication.saleCar(new HyundauiCarServiceImpl(), "현대", "그랜저");
  }
}
```

**3. Interface 주입을 통한 DI**

CarServiceInject 인터페이스를 통해서 생성된 객체   CarSaleConsumer에서 사용되는 서비스를 받아 사용하기 위해 인터페이스 주입을 사용한 예제로 CarService는 수정 없이 재사용 한다.

1. 자동차 판매에 대한 판매를 담당하는 인터페이스 선언 : CarSaleConsumer
2. 객체  생성을   위한 인터페이스 선언:   CarServiceInject
3. CarServiceInject에 대한 구현체 생성 : HyundauiCarServiceInjectImp
4. CarApplication 생성 : CarSaleConsumer 상속 받아 판매을 위한 구현으로 "생성자 주입을 통한 DI"을 사용 객체를 주입 한다.

```java
// 자동차 판매에 대한 판매를 담당하는 인터페이스 선언 
interface CarSaleConsumer {
  void saleCar(String manufacturer, String carType);
}


// 자동차 Application은 자동차 판매 인터페이스 상속 하여 판매에 대한 구현 
class CarApplication implements CarSaleConsumer {
  private CarService carService  ;

  public CarApplication(CarService carService) {
    this.carService = carService;
  }

  @Override
  public void saleCar(String manufacturer, String carType) {
    carService.sale(manufacturer, carType);
  }
}

// 객체 생성을 컴파일 시점에 하지 않고 런타임 시점에 하기 위한 클래스를 반환 하는 인터페이스 
interface CarServiceInject {
  CarSaleConsumer getCarService();
}

// 런타임 시점에 현대 자동차 주입을 통합 객체 생성 구현체 
// 다른 자동차 회사도 동알하게 작성 
class HyundauiCarServiceInjectImpl implements CarServiceInject {
  @Override
  public CarSaleConsumer getCarService() {
    return new CarApplication( new HyundauiCarServiceImpl() );
  }
}
```

Interface 주입을 통한 DI을 통한 Client 예제

```java
public class  CarClient {
  public static void main(String[] args){
    // 
    CarServiceInject carServiceInject = new HyundauiCarServiceInjectImpl();
    CarSaleConsumer carSaleConsumer = carServiceInject.getCarService();
    carSaleConsumer.saleCar("현대", "그랜저");
  }
}
```

## 5. 장단점&#x20;

### 5.1. 종속성 주입 장점

* 관심사 분리 및 응용 프로그램을 쉽게 확장할 수 있다.
* 종속하는 객체의 초기화가 인젝터에 의해 진행 되므로  애플리케이션 클래스의 코드 감소

### 5.2.  종속성 주입 단점

* 컴파일 타임에 발견되었을 런타임 오류를 런타임 시점에 알 수 있어서 유지보수 어려움 발생 할 수 있다.

## 6. Spring DI

### 6-1. DI ( Dependency Injection ) - XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xmlns:context="http://www.springframework.org/schema/context"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">    
    
    <bean id="entrServcice"
        class="co.kr.abacus.spring.xml.entr.service.EntrServiceImpl" >
         <constructor-arg ref="entrBySvcService" />
    </bean>

    <bean id="entrBySvcService"
         class="co.kr.abacus.spring.xml.entrsvc.service.EntrBySvcServiceImpl" />    
</beans>
```

<figure><img src="../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

### 6-2. DI ( Dependency Injection ) - annotation

컴포넌트 스캔 설정 ( component-scan )

스프링 설정 파일에 Application에서 사용할 Bean을 등록 하지 않고 자동 설정-\<context:component-scan> element 정의-Class에 @Component 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xmlns:context="http://www.springframework.org/schema/context"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">    
    
   <context:component-scan 
        base-package="co.kr.abacus.spring.annotation"></context:component-scan>
	
    
</beans>

```

<figure><img src="../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

### 6-3. DI ( Dependency Injection ) – JAVA Config

자바 설정 기반

스프링 설정 파일에 Application에서 사용할 Bean을 등록 하지 않고 자동 설정-@Configuration 사용

```java
@Configuration
public class ServiceConfig {

   @Bean
   public EntrService entrService() {
      return new EntrServiceImpl(entrBySvcService());
   }
	
   @Bean
   public EntrBySvcService entrBySvcService() {
      return new EntrBySvcServiceImpl();
   }	}
}

```

<figure><img src="../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>

### 6-4. DI ( Dependency Injection ) – 의존성 주입

Spring에서 의존성 주입을 지원하는 어노테이션은 @Autowired, @Inject, @Qualifier, @Resource 있음, Spring에서는 @Autowired, @Qualifier제공 함

<figure><img src="../.gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>
