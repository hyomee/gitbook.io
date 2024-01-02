# lombok + @Qualifier 문제

## 1. 문제점

Lombok @RequiredArgsConstructor와 Spring @Qualifier을 이용해서 Bean주입을 하는 경우 다음과 오류가 발생한다.

아래는SvcService를 상속받은 구현체가 2개 있고 productChg, subscriber로 설정한 소스이다.

```java
@Slf4j
@RequiredArgsConstructor
@Service("productChg")
public class SvcProductChgServiceImpl implements SvcService{
    @Override
    public void process(String ctn) {
        log.debug("SvcProductChgServiceImpl :: " + ctn);
    }
}

@RequiredArgsConstructor
@Service("subscriber")
public class SvcServiceImpl implements SvcService { 

    @Override
    public void process(String ctn) {
        log.debug(ctn); 
    }
}
```

@Qualifier를 사용하고  Lombok의@RequiredArgsConstructor을  사용한 소스로.

```java
@RequiredArgsConstructor
@Service
public class SubscriberServiceImpl implements SubscriberService {

     @Qualifier( "productChg")
     private final SvcService productChgSvcService;


     @Qualifier( "subscriber")
     private final SvcService subscriberSvcService;

}
```

<mark style="color:orange;">**빌드시 다음과 같은 오류가 발생 한다**</mark>.

```
Parameter 0 of constructor in OOOOOImpl required a single bean, but 2 were found:
	- X00001defined in file [....... \X00001Impl.class]
	- X00002defined in file [....... \X00002Impl.class]

Action:
Consider marking one of the beans as @Primary, updating the consumer to accept multiple beans, or using @Qualifier to identify the bean that should be consumed
```

## 2. 문제 해결

### 2-1. 원인

Lombok @RequiredArgsConstructor은 생성자를 자동 생성 할 때 final로 선언된 변수만 인수로 받는 생성자를 생성하는데 이때 생성자의 인수로 @Qualifier로 선언한 Bean이 생성되지 않기 때문이다

### 2-2. 조치&#x20;

#### 2-2-1. Lombok 설정 파일을 만들어 사용한다.

src/main/java 폴더에 lombok.config을 만들고 다음 내용을 작성 하고 빌드 하면 정상적으로 원하는 결과를 얻을 수 있다.

```
lombok.copyableAnnotations += org.springframework.beans.factory.annotation.Qualifier
```

#### 2-2-2. Resource 사용

Lombok를 사용하지 않고 java에서 제공하는 jakarta.annotation인 @Resource를 사용한다.

Lombok의@RequiredArgsConstructor을 사용하지 않고 final로 변수를 선언하지 않고 작성을 한다,&#x20;

```java
@Service
public class SubscriberServiceImpl implements SubscriberService {
   

    @Resource(name = "productChg")
    private SvcService productChgSvcService;

    @Resource(name = "subscriber")
    private SvcService subscriberSvcService;

    public void processEntr() {
        productChgSvcService.process("01093698787");
        subscriberSvcService.process("01094115479");
    }

}

```

## 3. @Autowired, @Inject, @Resource 차이점&#x20;

모두 의존관계를 자동으로 연결해주는 기능으로 @Autowired는 Spring이 제공해 주는 어노테이션이고 @Inject, @Resource는 자바에서 제공해주는 어노테이션이다,

### 3-1. @Resource&#x20;

* 자바에서 제공하는어노테이션
* Feld, Setter Method에 사용되며  생성자에는 사용 불가능 하다.
* 참조 변수와 이름이 동일한 빈을 주입하며, name 속성을 사용해서 주입 빈을 지정 할 수 있다.
* 이름으로 빈을 찾지 못하면 타입을 기준으로 의존성을 주입합한다.

### 3-2. @Inject

* 자바에서 제공하는어노테이션
* Field, Setter Method, Constructor에 사용 가능 하다.
* @Resource와 틀린점은 기본적으로 타입 기준으로 의존성을 주입 하며 동일한 타입이 여러개 존재 하면 참조 변수의 이름과 같은 빈을 주입한다.
* &#x20;@Qualifier,  @Named, @Primary을 사용하여 빈을 주입 한다.

### 3-2. @Autowired

* 스피링에서 제공하는 어노테이션
* &#x20;@Named을 제외하고 @Inject와 동일한 기능을 한다.

