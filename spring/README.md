# Spring

## 1. Spring **Framework 이란**

Java 애플리케이션을 빌드하기 위한 포괄적인 프로그래밍 및 구성 모델을 제공한 Java용 오픈 소스 애플리케이션 개발 프레임워크로  데이터 액세스 추상화, 트랜잭션 관리, 보안 등 프로그램 개발에서 공통으로 필요한 기능을 제공 합니다.

<figure><img src="../.gitbook/assets/image (55).png" alt="" width="465"><figcaption><p>Spring Framework Runtime</p></figcaption></figure>

Spring 프레임 워크는 코어, 빈, 컨텍스트, 표현식 언어, AOP, Aspects, Instrumentation, JDBC, ORM, OXM, JMS, Transaction, Web, Servlet, Struts 등과 같은 많은 모듈로 구성됩니다. 이러한 모듈은 다음 다이어그램과 같이 테스트, 코어 컨테이너, AOP, 측면, 계측, 데이터 액세스/통합, 웹(MVC/원격)으로 그룹화됩니다.

## 2. Spring **Framework** 디자인 철학

Spring은 객체 지향 프로그래밍(OOP)을 기반으로 한 경량화된 자바 프레임워크로 어플리케이션을 보다 간결하게 작성할 수 있게 해주고(간결성),  쉽게 모듈화 개발을 할 수 있도록 해주며 외부 프레임워크 또는 라이브러리와 쉽게 통합할 수 있도록 한다.&#x20;

이를 위해 **IoC(Inversion of Control), DI(Dependency Injection), AOP(Aspect-Oriented Programming)** 등의 기술을 사용하여  구성 요소 간의 느슨한 결합을 촉진하여 코드를 보다 간결하게 작성되고 보다 모듈화되고 유연하며 유지 관리하기 쉽게 만든다.&#x20;

**간결성,  모듈화, 유연성 및 확장성으로 다른 프레임워크와 쉽게 통합하는 할 수 있게 해주는 오픈소스 프레임워크**로  Spring Framework 6.0부터는 Java 17 이상이 필요하다.

Spring Framework은 모듈로 구성이 되어 있으며 구성(configuration) 모델과 종속성 주입(dependency injection)을 코어 모듈을 중심으로 메세징, 트랜잭션 데이터 및 지속성(persistence), 서블릿 기반의 Spring MVC 웹 Framework, Spring WebFlux 반응형 웹 프레임워크을 포함한 것을 의미한다.

Spring Framwork는 다음 요소를 포함 하고 있다.

* Servlet API([JSR 340](https://jcp.org/en/jsr/detail?id=340))
* WebSocket API([JSR 356](https://www.jcp.org/en/jsr/detail?id=356))
* Concurrency Utilities([JSR 236](https://www.jcp.org/en/jsr/detail?id=236))
* JSON Binding API([JSR 367](https://jcp.org/en/jsr/detail?id=367))
* Bean Validation([JSR 303](https://jcp.org/en/jsr/detail?id=303))
* JPA ([JSR 338](https://jcp.org/en/jsr/detail?id=338))
* JMS ([JSR 914](https://jcp.org/en/jsr/detail?id=914))
* Dependency Injection ([JSR 330](https://www.jcp.org/en/jsr/detail?id=330))&#x20;
* Common Annotations ([JSR 250](https://jcp.org/en/jsr/detail?id=250))&#x20;

## 3. Spring **Framework 장점**

* **단순화된 개발** : 개발에 대한 응집력 있는 모듈식 접근 방식을 제공하고사용 가능한 구성 요소를 제공하여 상용구 코드의 양을 줄이고 개발을 단순화하여 결과 개발 주기가 빨라지고 생산성이 향상된다.&#x20;
* **의존성 주입 및 제어의 반전** : 의존성 주입 (DI)을 통해 구성 요소 간의 느슨한 결합으로 보다 모듈화하고 유지 관리 및 테스트가 용이하고, 제어의 반전 (IoC)을 사용하여 객체 생성 및 수명 주기를 프레임워크가 관리하여 개발자의 부담을 줄일 수 있다.
* **모듈식 및 경량화** : Spring Framework에서 제공하는 구성 요소 중 필요한 모듈만 선택 사용하여 경량화 할 수 있다.
* **테스트 용이성** : 구성 요소를 쉽게 분리하고 격리하여 테스트할 수 있어 단위 테스트를 용이하게 하고 애플리케이션의 신뢰성을 보장할 수 있다.
* **AOP(관점 지향 프로그래밍)를 지원** : 로깅, 보안 및 트랜잭션 관리와 같은 교차 편집 문제를 핵심 애플리케이션 로직에서 분리 할 수 있다
* **다른기술과의 통합** : ORM 프레임워크(Hibernate, JPA), 메시징 시스템 등과 같은 다양한 기존 기술 및 프레임워크와 원활하게 통합할 수 있다.
* **강력한 트랜잭션 관리** : 프로그래밍 방식 및 선언적 트랜잭션 관리를 모두 지원하므로 개발자가 데이터베이스 트랜잭션을 효율적이고 안정적으로 처리할 수 있다.&#x20;
* 보안 : Spring Security 모듈을 통해 강력한 보안 기능을 제공하여. 인증, 권한 부여 및 보안 통신 기능을 제공하여 보안 애플리케이션을 보다 쉽게 구현할 수 있도록 한다.

## 4. Spring **Framework 단점**

* 가파른 학습 곡선
* 구성 복잡성
* 런타임 오버 헤드



## **5. 참고 :** 학습 관련 사이트&#x20;

1\. Soring Boot : [Spring Boot 튜토리얼 (howtodoinjava.com)](https://howtodoinjava.com/series/spring-boot/)

2\. Java : [자바 튜토리얼 - 자바 프로그래밍 배우기 - HowToDoInJava](https://howtodoinjava.com/series/java-tutorial/)

3\. Hibernate : [Hibernate 튜토리얼 (howtodoinjava.com)](https://howtodoinjava.com/series/hibernate-tutorials/)

4\. SpringBoot 속성 : [스프링 속성](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html)

5\. eugenp : 참고 : [https://github.com/eugenp/tutorials](https://github.com/eugenp/tutorials)&#x20;



교육 자료 :   &#x20;

{% file src="../.gitbook/assets/Spring 교육 자료 (1).pdf" %}



&#x20;
