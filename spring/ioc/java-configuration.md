---
description: Java-based Container Configuration
---

# Java 기반 Configuration

#### Bean 인스턴스화(**Instantiation) 방법 중 Java 기반 컨테이터 구성**

## 1. 기본 개념&#x20;

@Configuration, @Bean은 Spring의 Java기반 핵심 어노테이션으로 Spring IoC 컨테이너에서 관리할 새 개체를 초기화 한다.

<figure><img src="../../.gitbook/assets/image (163).png" alt="" width="563"><figcaption></figcaption></figure>

## 2. 프로그램 방식 -  스프링 컨테이너 생성

AnnotationConfigApplicationContext()를 사용해서 프로그램 방식으로  스프링 컨테이너를 생성 할 수 있다.

### 2-1. 일반 자바 스프링 컨테이너 생성

CarService를 상속받는 HyundauiCarServiceImpl.java, SamsungCarServiceImpl.java 예제

<figure><img src="../../.gitbook/assets/image (164).png" alt=""><figcaption><p>읿반 java </p></figcaption></figure>

HyundauiCarServiceImpl.class, SamsungCarServiceImpl.class는 일반 자바 객체로 스프링이 관리하는 컨테이너에 AnnotationConfigApplicationContext()를 사용 예제로 프로그램에서 필요에 따라서 스프림 컨테이너에 등록 해서 사용 할 수 있다.

````java
```java
@Component
public class DemoApplicationRun implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
         ApplicationContext ctx = new AnnotationConfigApplicationContext(
              HyundauiCarServiceImpl.class,
              SamsungCarServiceImpl.class);
          CarService hyundauiService = ctx.getBean(HyundauiCarServiceImpl.class);
          CarService samsungService = ctx.getBean(SamsungCarServiceImpl.class); 
          hyundauiService.sale("현대", "그랜저");
          samsungService.sale("삼성", "SM3");
    }

}
```
````

### 2-2. @Configuration을 스프링 컨테이너 생성

아래 코드는 스프링 @Configuration으로 선언한 클래스로 스프링에서 관리 할 수 있는 @Bean 주석을 사용한 예제 이다

```java
@Configuration
public class AppConfig {

    @Bean
    public HyundauiCarServiceImpl hyundauiCarService() {
        return new HyundauiCarServiceImpl();
    }

    @Bean
    public SamsungCarServiceImpl samsungCarService() {
        return new SamsungCarServiceImpl();
    }
}
```

#### 2-2-1.  register

AnnotationConfigApplicationContext()를 사용해서 프로그래밍 방식으로 스프링 컨텍스트에 등록 하고 사용하는 예시이다.

```java
@Component
public class DemoApplicationConfigRun implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        try (AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext()) {
            ctx.register(AppConfig.class);
            ctx.refresh();
            CarService hyundauiService = ctx.getBean(HyundauiCarServiceImpl.class);
            CarService samsungService = ctx.getBean(SamsungCarServiceImpl.class); 
            hyundauiService.sale("현대", "그랜저");
            samsungService.sale("삼성", "SM3");
        }
    }
}
```

AppConfig.java&#x20;

```java
@Configuration
public class AppConfig {

    @Bean
    public HyundauiCarServiceImpl hyundauiCarService() {
        return new HyundauiCarServiceImpl();
    }

    @Bean
    public SamsungCarServiceImpl samsungCarService() {
        return new SamsungCarServiceImpl();
    }
}
```

#### 2-2-2. scan

구성 요소 스캔을 통한 스프림 컨테이터 생성 할 때 사용하는 것으로 basePackages로 선언한 디렉토리에 있는@Configuration 주석이 있는 파일을 찾아서 @Bean 주석이 있는 것을 AnnotationConfigApplicationContext 에 등록한다.

```java
@Component
public class DemoApplicationScanRun implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
       try (AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext()) {
            ctx.scan("com.hyomee.config");
            ctx.refresh();
            CarService hyundauiService = (CarService) ctx.getBean("Handaui");
            CarService samsungService = ctx.getBean(SamsungCarServiceImpl.class); 
            System.out.println("************************");
       
            hyundauiService.sale("현대", "그랜저");
            samsungService.sale("삼성", "SM3");
        }
    }
}
```

CarService hyundauiService = (CarService) ctx.getBean("Handaui"); 은 @Bean선언시 이름을 부여 한 것을 찾기 위해서 사용한 것으로 Object로 반환 하므로 캐스딩을 해야 한다.

<pre class="language-java"><code class="lang-java">package com.hyomee.config;

<strong>@Configuration
</strong>public class AppConfig {
    @Bean("Handaui")
    public CarService hyundauiCarService() {
        return new HyundauiCarServiceImpl();
    }

    @Bean("Sansung")
    public CarService samsungCarService() {
        return new SamsungCarServiceImpl();
    }
}
</code></pre>

## 3.  컴포넌트 스캔 - 스프링 컨테이너 생성

수동으로 스프링 컨데이너 생성은 개발자가 직접 코드를 작성해야 하는 붚편함을 덜어주기 위해 주석기반으로 생성할 수 있다.&#x20;

<pre class="language-java"><code class="lang-java">package com.hyomee.config;

<strong>@Configuration
</strong>@ComponentScan(basePackages = "com.hyomee.config")
public class AppConfig {
    @Bean("Handaui")
    public CarService hyundauiCarService() {
        return new HyundauiCarServiceImpl();
    }

    @Bean("Sansung")
    public CarService samsungCarService() {
        return new SamsungCarServiceImpl();
    }
}
</code></pre>

&#x20;

참고 : [Java-based Container Configuration](https://docs.spring.io/spring-framework/reference/core/beans/java.html)

