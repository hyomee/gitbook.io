---
description: Spring Inversion of Control(IoC) 컨테이너에 의해 관리되는 오브젝트
---

# Spring Beans

참고 :  [Bean Overview](https://docs.spring.io/spring-framework/reference/core/beans.html)&#x20;

Spring에서 Bean은  XML 기반 구성 또는 Java 기반 구성을 사용하여 정의하고개발자는 컨테이너가 Bean을 작성하고 관리하는 데 필요한 정보(해당 클래스, 특성 및 종속성)를 지정한다.&#x20;

Spring Framework에서 Bean을 사용할 때 DI(dependency injection)를 통해 Bean은 컨테이너에서 종속성을 자동으로 해결하고 주입한다. 이는 느슨한 결합이되어 어플리케이션의 유연성과 모듈성을 향상 시킨다.

Spring 컨테이너는 다음과 같은 기능을 제공한다.

* Bean의 수명 주기 단계를 관리하므로 개발자는 Bean이 생성되고 멸될 때 컨테이너에 의해 실행될 초기화 및 소멸 콜백을 지정할 수 있다.&#x20;
* Bean의 라이프사이클과 가시성을 정의하는 다양한 Bean 범위를 지원한다.&#x20;
  * Bean 범위 :  싱글톤, 프로토타입, 요청, 세션, 애플리케이션
* Bean은 모듈식 설계 및 코드 재사용을 가능하게 한다. 즉 특정 기능을 켑술화하는 작은 단위로 나뉘어 모듈화하여 코드베이스를 더 쉽게 이해, 유지 관리 및 확장할 수 있도록 한다.
* AOP(Aspect-Oriented Programming), 트랜잭션 관리, 캐싱 및 보안과 같은 기능을 제공하여 개발자는 추가 기능을 Bean에 쉽게 통합하고 애플리케이션에서 더 높은 수준의 추상화 및 모듈성을 달성할 수 있다.

## 1.  Bean은&#x20;

* Spring Framework의 기본 단위로, 응용 프로그램의 구성 요소를 나타내고
* Spring 컨테이너에 의해 관리되며&#x20;
* 종속성 주입, 수명 주기 관리, 모듈식 설계 및 코드 재사용과 같은 이점을 제공한다.
* Bean을 활용하여 개발자는 Spring Framework를 사용하여 유연하고 유지 관리 가능하고확장 가능한 애플리케이션을 빌드할 수 있다.

## 2. **Beans 수명 주기**

<figure><img src="../../.gitbook/assets/image (162).png" alt=""><figcaption><p>생명 주기</p></figcaption></figure>

#### 2-1. **인스턴스화(Instantiation)**&#x20;

Spring 컨테이너는 XML 기반 구성, JavaConfig 어노테이션 또는 컴포넌트 스캔을 포함한 다양한 메커니즘을 통해   Bean 인스턴스를 생성한다.&#x20;

* **XML 구성 :** XML 기반 구성을 통해 개발자는 XML 파일 내에서 Bean
* **주석 기반 구성 :** @Component, @Service, @Repository 및 @Controller와 같은 주석을 클래스에 적용하여 Bean의 역활응 하게 한다.\
  \
  <mark style="color:orange;">**참고 :**</mark> [<mark style="color:orange;">**Annotation-based Container Configuration**</mark>](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config.html)\

* **JavaConfig :** Spring 3.0에 도입된 JavaConfig를 사용하면 개발자가 일반 Java 클래스를 사용하여 Bean을 구성한다.  구성 클래스는 @Configuration, @Bean 및 @Import와 같은 어노테이션을 사용하여 Bean 및 해당 종속성을 정의한다.\
  \
  <mark style="color:orange;">**참고 :**</mark> [<mark style="color:orange;">**Java-based Container Configuration**</mark>](https://docs.spring.io/spring-framework/reference/core/beans/java.html)\

* **Component Scanning :** Spring은 사전 정의된 기본 패키지 내에서 특정 주석(예: @Component)이 있는 클래스를 자동으로 감지하는 구성 요소 스캔을 제공한다.

#### 2-2. **Dependency Injection**:

Bean 인스턴스가 생성되면 Spring 컨테이너는 Bean에 필요한 종속성을 삽입한다.&#x20;

#### 2-3.  **초기화(Initialization)**

&#x20;종속성이 삽입된 후 컨테이너는 Bean을 초기화한다. 이 단계에는 Bean 내에 정의된 초기화 콜백 또는 메소드 호출이 포함되며, 이를 통해 개발자는 필요한 설정 태스크를 수행할 수 있다.

#### 2-4. **사용 중(In Use)**

Bean은 애플리케이션 내에서 사용할 준비가 되고다른 컴포넌트가 Bean의 기능을 활용하여 Bean과 연결되어 사용된다.

#### 2-4. **소멸(Destruction)**

애플리케이션 또는 컨테이너가 종료되면 Spring 컨테이너가 소멸 단계를 트리거한다.  컨테이너는 Bean에 정의된 소멸 콜백 또는 메소드를 호출하여 정리 태스크, 자원 해제 또는 정상 종료를 한다.

## **3. Beans 장점**

* **종속성 주입 :** Bean은 느슨한 결합을 달성하여 코드를 보다 모듈화하고 유지 관리 및 테스트 가능한다.
* **제어 반전(IoC) :**  Spring 컨테이너가객체 생성 및 수명주기 제어를 담당하게 되어  관심사의 분리를 가능하게 하여 객체 관리를 단순화할 수 있다.
* **구성 유연성** : Beans는 XML, 어노테이션 및 JavaConfig와 같은 다양한 **구성** 옵션을 사용하여 개발자는 프로젝트 요구 사항, 팀 기본 설정 및 유지 관리 고려 사항에 따라 가장 적합한 구성 방법을 선택할 수 있다.
* **모듈식 설계** : Beans는 개별 구성 요소 내에 특정 기능을 캡슐화하여 모듈식 설계를 용이하게 되고 이러한 모듈성은 코드 재사용, 문제 분리 및 유지 관리 용이성을 촉진하여 보다 강력하고 확장 가능한 애플리케이션으로 만들수.있다.
* **테스트 용이성** : 종속성 주입 및 모듈식 설계를 통해 테스트가.용이하게 되어 코드 품질이 향상되고 버그가 줄어들며 전반적인 애플리케이션 안정성이 향상된다.
* **런타임 관리 용이성** : Bean은 Spring 컨테이너에 의해 관리되기 때문에 개발자가 수동 관리 작업을.하지 않아도 되어 응용 프로그램 안정성이 향상되고 리소스 누수 가능성을.줄일 수 있다.
* **AOP(Aspect-Oriented Programming):** Spring Framework는 AOP와 원활하게 통합되어 Bean에 여러 측면을 적용함으로써  개발자는 로깅, 보안 및 트랜잭션 관리와 같은 기능을 Spring 컨테이너가 담당하게 하고 업무 로직에 집중 할 수 있어 개발 생산성이 높아 지고 모듈성 및 유지 관리성을 향상시킬 수 있다.

## **5. Beans 단점**

Spring Framework를 이해하고 적용을 해야 효과를 볼 수 있듯이 덜 익은 감을 먹는 것 처럼 장점을 남용 할 떄는 일반 개발보다 더 시간이 많이 걸리고 복잡성이 높아 지며 Spring에 의존성이 생겨 다른 프레임워크 또는 기술 스택으로 마이그레이션하려면 상당한 코드 변경 및 리팩토링이 필요할 수 있다. 또한 런타임시 Bean 주입이 많아져서 런타임 오버헤드가 발생한다

참고 : [Java-based Container Configuration](https://docs.spring.io/spring-framework/reference/core/beans/java.html)
