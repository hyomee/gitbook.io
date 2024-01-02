---
description: Inversion of Control
---

# IoC

IoC(Inversion of Control)는 프로그램의 객체 또는 일부에 대한 제어를 컨테이너 또는 프레임 워크로 이전하는 소프트웨어 엔지니어링의 원칙으로 프레임워크가 프로그램 흐름을 제어하고 개발자가 구현한 코드(지정한 코드)를 호출하게 하는 것으로 객체의 생성, 생명 주기의 관리까지 모든 객체에 대한 제어권을 개발자가 관리하는 것이 아니라 프레임워크(컨테이너)가 관리 한다는 의미이다.

Strategy design pattern, Service Locator pattern, Factory pattern, and Dependency Injection (DI)을와 같은 전략 패턴으로 IoC(Inversion of Control)를 구현 할 수 있다.

## 1. 장점&#x20;

* 프로그램  실행과 구현을 분리 할 수 있다.
* 서로 다른 구현 간에 쉽게 전환할 수 있습니다.
* 프로그램의 모듈성 향상 시킬 수 있다.
* 구성 요소를 격리하거나 종속성을 조롱하고 구성 요소가 계약을 통해 통신할 수 있도록 하여 프로그램을 더 쉽게 테스트할 수 있습니다.

## 2. Spring IoC **Container**

참고 : [The IoC Container](https://docs.spring.io/spring-framework/reference/core/beans.html)

스프링 프레임워크에서  객체의 생성, 생명 주기 관리를 책임지고 의존성을 관리해주는 컨테이너로  **BeanFactory , ApplicationContext** 가 IoC Container이다.

<figure><img src="../../.gitbook/assets/image (51).png" alt="" width="563"><figcaption><p>Spring IoC Container</p></figcaption></figure>

* **BeanFactory** : 객체의 생성과 객체 사이의 런타임 관계를 DI 관점에서 볼 때 컨테이너를 BeanFactory라고 한다.
* **ApplicationContext** : BeanFactory에 여러가지 컨테이너 기능을 추가한 **ApplicationContext** 가 있다.

## 3. Spring IoC **Container 필수 단계**

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption><p>IoC Container 단계</p></figcaption></figure>

## 4. Spring IoC **Container** 종류

<figure><img src="../../.gitbook/assets/image (161).png" alt=""><figcaption></figcaption></figure>

### 4-1. **BeanFactory**&#x20;

Spring 애플리케이션에서 Bean을 작성, 관리 및 구성하는 작업을 담당하는 컨테이너로  Bean을 위한 팩토리 역할을 한다. XML 파일, 어노테이션 또는 Java 기반 구성 클래스의 양식일 수 있는 구성 메타데이터를 읽고 해당 메타데이터를 기반으로 Bean을 인스턴스화 한다.&#x20;

* **Bean을 인스턴스화** : 지연 로드 동작은 시작 시간과 메모리 소비를 줄여 애플리케이션 성능을 향상 한다.
* **Bean의 라이프사이클도 관리** : Bean의 초기화 및 소멸을 처리하여 더 이상 필요하지 않을 때 제대로 초기화되고 해제한다.
* **종속성 주입** : 종속성(의존성)은 사용되는 구성 접근 방식에 따라 생성자 삽입, setter 주입 또는 필드 삽입을 통해 지정할 수 있다.
* **국제화 및 자원 로딩을 지원** : 역화된 메시지 리소스를 로드하고 관리

Spring BeanFactory Container 는 느슨한 결합, 모듈식 설계 및 유지 관리 가능한 코드를 달성하기 위한 견고한 기반을 제공하며 지연 로딩, 종속성 주입, 다중 범위 및 국제화를 지원을 통해 개발자가 유연하고 확장 가능하며 고도로 구성 가능한 애플리케이션을 빌드할 수 있도록 한다.

### 4-2. **ApplicationContext**&#x20;

BeanFactory 컨테이너의 고급 확장으로 Spring 애플리케이션에서 Bean, 해당 종속성 및 구성을 관리하기 위한 중앙 허브 역할을 한다.

* **국제화 및 메시지 리소스 처리**
* **계층적 컨텍스트에 대한 지원**
* **이벤트 처리에 대한 지원** : Bean이 이벤트를 발행하고 다른 Bean이 해당 이벤트를 수신하고 응답할 수 있도록 하는 유연하고 강력한 이벤트 메커니즘을 제공한다.
* **다양한 Spring 모듈 및 확장과 원활하게 통합** : Spring MVC, Spring Security, Spring Data 등과 같은 구성 요소를 자동으로 감지하고 구성한다.
* **속성 확인 메커니즘을 지원** : 환경 변수, 시스템 속성, 구성 파일 및 데이터베이스를 포함한 다양한 소스의 속성 값을 확인 할 수 있다.&#x20;

## 5. BeanFactory와 ApplicationContext의 차이점

BeanFactory 및  ApplicationContext 인터페이스는 IoC 컨테이너 역할하는 것으로Spring의 AOP와의 간단한 통합, 메시지 리소스 처리 (I18N 용), 이벤트 전파, 웹 응용 프로그램을위한 응용 프로그램 계층 별 컨텍스트 (예 : WebApplicationContext)와 같은 BeanFactory 보다 몇 가지 추가 기능을 추가합니다. 따라서 BeanFactory 보다 ApplicationContext 를 사용하는 것이 좋습니다.

## 6. 예제

일반적인 Java에서 종속성 예시는 다음과 같다.

<pre class="language-java"><code class="lang-java"><strong>public class CarSale{
</strong>    private CarServie carService;
 
    public CarSale() {
        this.carService= new HandaiCarServiceImpl();    
    }
}
</code></pre>

### 6-1. DI 적용&#x20;

```java
public class CarSale{
    private CarServie carService;
 
    public CarSale(CarServie carService ) {
        this.carService = carService ;    
    }
}
```

참고 : [Dependencies](https://docs.spring.io/spring-framework/reference/core/beans/dependencies.html),  [Dependency Injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)

### 6-2. Spring 생성자 기반 종속성 주입

```java
@Configuration
public class AppConfig {

    @Bean
    public CarServie carService() {
        return new HandaiCarServiceImpl();
    }

    @Bean
    public CarSale carSale() {
        return new CarSale(carService());
    }
}
```

_@Configuration_ 어노테이션은 클래스가 Bean 정의의 소스임을 표시 한다. 메메서드에 _@Bean_ 어노테이션을 사용하여 Bean을 정의한다. 사용자 지정 이름을 지정하지 않으면 Bean 이름은 기본적으로 메서드 이름으로 설정된다.

기본 싱글 톤 범위를 가진 빈의 경우, Spring은 먼저 빈의 캐시 된 인스턴스가 이미 존재하는지 확인하고, 그렇지 않은 경우에만 새 인스턴스를 작성한다. 프로토타입 범위를 사용하는 경우 컨테이너는 각 메서드 호출에 대해 새 Bean 인스턴스를 반환한다.

### 6-3. Spring **Setter** 기반 종속성 주입

```java
@Bean
public CarSale carSale () {
    CarSale carSale = new CarSale ();
    carSale.setCarService(HandaiCarServiceImpl());
    return carSale ;
}
```

Bean을 인스턴스화하기 위해 인수 없는 생성자 또는 인수 없는 정적 팩토리 메서드를 호출한 후 클래스의 setter 메서드를 호출한다.

### 6-4. Spring _@Autowired_ 주석

```java
public CarSale carSale () {
    @Autowired
    private CarServie carService;
}
```

CarSale 오브젝트를 생성하는 동안 CarServie Bean을 삽입할 생성자 또는 setter 메소드가 없는 경우 컨테이너는 리플렉션 을 사용하여 CarServie 을 CarSale 에 삽입한다.

```
public CarSale carSale () {
    @Autowired
    @Qualifier("handaiCarServiceImpl")
    private CarServie carService;
}
```

동일한 유형의 Bean이 두 개 이상 있는 경우 _@Qualifier_ 주석을 사용하여 이름으로 Bean을 참조할 수 있다.
