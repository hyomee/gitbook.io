# 애플리케이션 만들기

스프링 부트 애플리케이션을 만들기 위해서는 Maven, Grade을 사용해서 구성을 하거나 [Spring initializr](https://start.spring.io/)을 사용해서 만들수 있다.

1. Spring initializr 프로젝트 생성 및 실행
2. @Component, @Bean
3. @Component, @Service

## **1. Spring initializr 프로젝트 생성 및 실행** <a href="#id-1-spring-initializr" id="id-1-spring-initializr"></a>

* Project : Gradle - Groovy
* Language : Java
* Spring Boot : 3.1.3
* Project Metadata :
  * Packaging : Jar
  * Java : 17
* Dependencies
  * Spring Configuration Processor DEVELOPER TOOLS

![Spring initializr](https://hyomee.github.io/doc/01\_spring/000\_SpringBoot\_%EA%B8%B0%EC%B4%88/img/springinitiakizr\_01.png)

을 선택 하고 Generate를 클릭 해서 demo.zip 파일을 다운 받는다.

### **1-1. 다운 로드 파잉 압축을 푼다** <a href="#id-1-1" id="id-1-1"></a>

![Spring initializr](https://hyomee.github.io/doc/01\_spring/000\_SpringBoot\_%EA%B8%B0%EC%B4%88/img/application.png)

### **1-2. 개발 Tool 작업** <a href="#id-1-2-tool" id="id-1-2-tool"></a>

IntelliJ IDEA 2022.3.3 (Community Edition)에서 오픈 하면 자동으로 의존성 파일을 다운로드 받는다. 아래 화면은 IntelliJ에서 HELP.md 파일의 내용이다.

![Spring initializr](https://hyomee.github.io/doc/01\_spring/000\_SpringBoot\_%EA%B8%B0%EC%B4%88/img/readme.png)

* build.gradle : 의종성 관리
* src/main/com/example/demo/DemoApplication : 스프링 부트 Main 애플리케이션 파일

### **1-3. 소스 코드로 변경하고 Application Run** <a href="#id-1-3-application-run" id="id-1-3-application-run"></a>

```
@SpringBootApplication
public class DemoApplication {
	public static void main(String[] args) {
		var applicationContext = SpringApplication.run(DemoApplication.class, args);

        
		var beanNames = applicationContext.getBeanDefinitionNames();

        // bean 이름 순으로 정렬
        Arrays.sort(beanNames);

        // bean 이름 출력
		Arrays.asList(beanNames).forEach(System.out::println);
	}
}
```

> 실행 결과 :

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.1.3)

2023-09-07T23:36:36.188+09:00  INFO 19432 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication using Java 17.0.6 with PID 19432 (D:\PRJJ\demo\demo\build\classes\java\main started by ASUS in D:\PRJJ\demo\demo)
2023-09-07T23:36:36.194+09:00  INFO 19432 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to 1 default profile: "default"
2023-09-07T23:36:37.403+09:00  INFO 19432 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 1.868 seconds (process running for 2.711)
applicationAvailability
applicationTaskExecutor
demoApplication
forceAutoProxyCreatorToUseClassProxying
lifecycleProcessor
org.springframework.aop.config.internalAutoProxyCreator
org.springframework.boot.autoconfigure.AutoConfigurationPackages
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration


```

스프링 부트 애플리케이션이 동작하는 것을 확인 할 수 있다.

### **1-4. @SpringBootApplication 어노테이션** <a href="#id-1-4-springbootapplication" id="id-1-4-springbootapplication"></a>

@SpringBootConfiguration, @EnableAutoconfiguration, @ComponentScan 어노테이션을 합쳐둔 어노테이션으로 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성이 모두 자동으로 설정된다. 아래 코드는 @SpringBootApplication의 소스이다.

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
}
```

* @EnableAutoConfiguration – 설정 자동 등록
  * spring-boot-autoconfigure > META-INF > spring.factories에 등록한 Bean 등록
* @ComponentScan – 빈 등록하기
  * 스프링에서 관리 하는 Bean 등록
    * @Component
    * @Configuration
    * @Repository
    * @Service
    * @Controller
    * @RestController
* @SpringBootConfiguration - @Configuration
  * @Configuration : 스프링 에 빈 팩토리를 위한 오브젝트를 설정을 담당하는 클래스라고 인식 할 수 있도록 알려주는 어노테이션으로 외부 라이브러를 스프링에서 관리하도록 등록할 때 사용한다.

## **2. @Component와 @Bean** <a href="#id-2-component-bean" id="id-2-component-bean"></a>

@Component는 클래스를 스프링 빈으로 등록하도록 알려주는 어노테이션이고. @Bean은 메서드에 사용되며 해당 메서드가 반환하는 객체를 스프링 빈으로 등록한다.

> @Component : @ComponentScan의 대상으로 자동으로 스프링 컨테이너에 스프링 빈을 등록하는 기능으로 @Service, @Controller, @Repository등에 포함되어 있다.

> @Bean : 수동으로 스프링 컨테이너에 등록할 스프링 빈을 직접 등록하는 기능으로 @Configuration에 @Bean으로 스프링 컨테이너에 자바 객체를 등록한다.

## **3. @Component와 @Service** <a href="#id-3-component-service" id="id-3-component-service"></a>

@Service는 @Component와 같은 기능을 하지만 비즈니스 로직을 담당하는 서비스 클래스임을 나타내는 어노테이션이다
