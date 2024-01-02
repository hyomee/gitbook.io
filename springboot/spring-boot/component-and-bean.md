# @Component & @Bean

## 1. @Component

@Component는  Bean을 자동 감지하여 자동으로 구성 하기 위해서 사용되는 Annotation으로 개발자가 직접 작성한 클래스를 Bean으로 등록하기 위해 사용됩니다.

@Componet 클래스가 기본 패키지 아래에 있거나 Spring이 스캔 할 다른 패키지를 감지하여 각각의  클래스를  새로운 Bean을 생성 하여  ApplicationContext에 등록합니다.

* 빈 선언&#x20;
  * @ Component를 선언하는 경우 기본적으로 메서드명을 빈 이름으로 등록&#x20;
  * @Component(”beanName”)와 같이 빈 이름을 직접 명시하면 명시한 이름으로 등록
* Bean과 Component는 일대일, 즉 클래스 당 하나의 Bean으로 매핑됩니다.
* @Controller & @RestController, @Service, @Repository 등이 클래스 레벨의 Annotation 입니다

### &#x20;2. Bean

@Bean은  단일 been을 선언하는 데 사용 되어지고 클래스에서 been의 선언을 분리하여 구성 합니다. 즉 메서드 레벨에서 사용이 되어 지고 @Configuration가 필요 합니다.

* 개발자가 직접 제어가 불가능한 외부 라이브러리 등을 Bean으로 만들려고 할 때 사용됩니다.
* (name = "") 옵션이 있고, 해당 옵션을 사용하지 않는다면 method 이름을 camelCase로 변경한 것이 bean id로 등록됩니다.



## 3.  @ComponentScan

스프링에서는 설정 정보(AppConfig.class) 없이 자동으로 스프링 빈을 등록하는 컴포넌트 스캔 기능으로 탐색 위치에 @Component가 붙은 모든 클래스를 스프링 빈으로 등록합니다,

#### 3-1. 컴포넌트 스캔의 기본 대상

* @Component&#x20;
* @Controller & @RestController&#x20;
* @Service&#x20;
* @Repository&#x20;
* @Configuration&#x20;

#### 3-2. 속성&#x20;

* 탐색 범위 :&#x20;
  * 탐색 범위를 지정하지 않으면 @ComponentScan이 붙은 설정 정보 클래스의 패키지가 시작 위치가 됩니다.
  * basePackages : 탐색할 패키지의 시작 위치를 설정하고, 해당 패키지부터 하위 패키지까지 모두 탐색 합니다.&#x20;
  * basePackageClasses : 클래스가 속한 패키지를 탐색 시작 위치로 지정할 수 있습니다.
* filter&#x20;
  * 컴포넌트 스캔의 대상 범위를 지정하는데 사용됩니다.
  * includeFilters : 컴포넌트 스캔 대상을 추가로 지정합니다.&#x20;
  * excludeFilters : 컴포넌트 스캔에서 제외할 대상을 지정합니다.
  * ilterType 옵션
    * ANNOTATION : 기본값으로 어노테이션으로 인식해서 동작.
    * ASSIGNABLE\_TYPE : 지정한 타입과 자식 타입을 인식해서 동작
    * ASPECTJ : AspectJ 패턴을 사용
    * REGEX : 정규표현식&#x20;
    * CUSTOM : TypeFilter라는 인터페이스를 구현해서 처리.

## &#x20;4. @Component 예제

#### 4-1.  @Component 선언을 하지 않아서 오류 발생 &#x20;

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption><p>Spring Bean 등록 오류</p></figcaption></figure>

* 원인 :  Spring Bean 을 등록 하지 않아서 에러 발생&#x20;

#### 4-2. @Component 를 클래스에 삽입&#x20;

<figure><img src="../../.gitbook/assets/image (192).png" alt="" width="563"><figcaption></figcaption></figure>

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/users")
    List<UserDTO> getAllUsers() {
        return userService.getListOfUser();
    }
}

@Service
public class UserService {

    private final List<UserDTO> userList = new ArrayList<>();

    @PostConstruct
    private void init() {
        UserDTO userDTO = UserDTO.builder()
                .id(1)
                .name("홍길동")
                .address("서울")
                .build();

        userList.add(userDTO);
    }

    public List<UserDTO> getListOfUser() {
        return this.userList;
    }

}

```

* Service, RestController 는 다음과 같은 구조를 가집니다.

<figure><img src="../../.gitbook/assets/image (5).png" alt="" width="563"><figcaption></figcaption></figure>

* @Service는 @Component를 @RestController는 @Controller, @Component를 가지고 있어서 @SpringBootApplication는 @ComponentScan을 가지고 있어서 SpringBoot가 시작 될 때 자동으로 Bean으로등록되어 ApplicationContext에 할당 됩니다. ( Spring에서 Bean 생명 주기를 관리 하게 됩니다.)&#x20;

```java
@SpringBootApplication
public class SpringBootComponentVsBeanApplication {

    public static void main(String[] args) {
       SpringApplication.run(SpringBootComponentVsBeanApplication.class, args);
    }

}
```

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption><p>Bean Graph</p></figcaption></figure>
