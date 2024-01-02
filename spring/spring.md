# Spring 기본 개념

* Spring Application Context라는 Container가 Application Component을 생성하고 관리 한다.&#x20;
* Dependency Injection(DI) 패턴을 기반으로 Bean의 상호 연결을 수행 한다. : Application Component에 의존(사용)하는 다른 빈의 생성과 관리를 별도의 Container가 해주며, Component를 필요로 하는 Bean에 주입 한다.

<figure><img src="../.gitbook/assets/image (10).png" alt="" width="521"><figcaption></figcaption></figure>

## 1. Configuration Lifecycle

<figure><img src="../.gitbook/assets/image (11).png" alt="" width="479"><figcaption></figcaption></figure>

## 2. Spring IoC(Inversion of Control) Container

### 2-1. IoC (inversion of control)&#x20;

프로그램의 실행 흐름이나 객체의 생명 주기를 개발자가 직접 제어 하는 것이 아니라 컨테이너로 제어권이 넘어가는 것

<figure><img src="../.gitbook/assets/image (13).png" alt="" width="456"><figcaption></figcaption></figure>

### 2-2. Bean&#x20;

* 컨테이너가 관리 하는 객체를 의미 하며 기본적으로 싱클턴
* 기본적으로 네가지 애너테이션을 사용하여 Class를 자동으로 Bean으로 등록
  * @Controller : Presentation Layer에서 Controller명시&#x20;
  * @Service : Business Layer에서 Service 명시&#x20;
  * @Repository : Persistence Layer 에서 DAO 명시&#x20;
  * @Component : 기타 자동 등록 하고 싶은 것
  * &#x20;@Bean : 외부 라이브러리의 객체를 Bean으로 만들떄

<figure><img src="../.gitbook/assets/image (14).png" alt="" width="544"><figcaption></figcaption></figure>

## 3. Spring Bean Life Cycle

* 인터페이스 구현 : Spring 에 종속적&#x20;
* Bean 정의 시 메소드 지정 : Spring 에 종속적&#x20;
* JSR-250 어노테이션 지정

<figure><img src="../.gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

```java
public class BSimpleClass {
    
    @PostConstruct
    public void inPostConstructit(){
        System.out.println("BEAN 생성 및 초기화 : init() 호출됨");
    }
 
    @PreDestroy
    public void destroy(){
        System.out.println("BEAN 생성 소멸 : destroy 호출됨");
    }
}

```

* @PostConstruct 어노테이션을 지정한 메소드를 Bean생성과 properties의존성 주입 후 콜백으로 호출
* @PreDestroy 어노테이션을 지정한 메소드를 Bean 소멸 전 콜백으로 호출
