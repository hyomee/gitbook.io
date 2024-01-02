# AOP

Aspect Oriented Programming의 약자로 관점 지향 프로그래밍으로 로직의 핵심적인 관점, 부가적인 관점으로 나누고 관점을 기준으로 모듈화 하는 것입니다. 즉 어떤 공통된 로직이나 기능을 하나의 단위로 묶는 것 분리하는 것 입니다. ( 예: DB 연결, 로그 출력 등 .. )

## 1. 횡단 관심과 핵심 관심 분리

<figure><img src="../../.gitbook/assets/image (15).png" alt="" width="563"><figcaption></figcaption></figure>

메소드 마다 공통인 로깅, 예외, 트랜잭션과 같은 횡단 관심과 실제 수행 되는 업무 로직을 핵심 관심이라 합니다.

## 2. AOP 용어

<figure><img src="../../.gitbook/assets/image (16).png" alt="" width="357"><figcaption></figcaption></figure>

1. Advice : Aspect가 해야 할 작업으로 언제 무엇을 해야 하는지 정의&#x20;
2. before(이전) : advice 대상 Method가 호출 되기 전&#x20;
3. after(이후) : 결과에 상관없이 advice 대상 Method사 완료 된 후&#x20;
4. after-returning(반환 이후) : advice 대상 Method가 성공적으로 완료 된 후&#x20;
5. after-throwing(예외 발생 이후) : advice 대상 Method가 예외를 던진 후&#x20;
6. around(주위) : advice가 advice 대상 메소드를 감싸서 advice 대상 Method 호출 전과 호출 후 몇가지 기능 제공
7. &#x20;Join point : Advice를 적용 할 수 있는 곳으로 Application 실행에 Aspect를 끼워 넣을 수 있는 지점, 모든 업무 메소드
8. Pointcut : Aspect가 Advice할 Join point의 영역을 좁히는 일을 함, 즉 어디서 할지를 정의 함 각 poiintcut은 Advice가 weaving되어야 하나 이상의 Join point를 정의함&#x20;
9. Aspect : Advice와 Pointcut를 합친 것으로 무엇을 언제 어디서 할지 모든 정보를 정의함&#x20;
10. Introduction : 기존 Class에 코드 변경 없이도 새 Method, Member Variable을 추가 하는 기능&#x20;
11. Weaving : 타깃 객체에 Aspect를 적용해서 새로운 프록시 객체를 생성하는 절차&#x20;
12. compile time : 컴파일 시점에 weaving. 별도의 컴파일러 필요, Aspect 5의 Weaving Compile&#x20;
13. classload time : JVM에 로드 될 떄 weaving. AspectJ 5의 Load-Time Weaving 기능 사용&#x20;
14. runtime : 실행 중에 weaving. Spring AOP Aspect

## 3. Spring의 AOP

프록시 패턴 기반의 AOP 구현체, 프록시 객체를 쓰는 이유는 접근 제어 및 부가기능을 추가하기 위해서 입니다. 스프링 빈에만 AOP를 적용 가능 모든 AOP 기능을 제공하는 것이 아닌 스프링 IoC와 연동하여 엔터프라이즈 애플리케이션에서 가장 흔한 문제(중복코드, 프록시 클래스 작성의 번거로움, 객체들 간 관계 복잡도 증가 ...)에 대한 해결책을 지원하는 것이 목적입니다.

* Classic Spring Proxy 기반&#x20;
* AOP Pure-POJO Aspect&#x20;
* @AspectJ Annotation 기반&#x20;
* Aspect AspectJ Aspect에 bean 주입

### 3-1 프록시 패턴

&#x20;대상 원본 객체를 대리하여 대신 처리하게 함으로써 로직의 흐름을 제어하는 행동 패턴으로 객체 지향 프로그래밍에서 클라이언트가 대상 객체를 직접 쓰는게 아니라 중간에 프록시(대리인)을 거쳐서 쓰는 코드 패턴이라고 보면 된다. 따라서 대상 객체(Subject)의 메소드를 직접 실행하는 것이 아닌, 대상 객체에 접근하기 전에 프록시(Proxy) 객체의 메서드를 접근한 후 추가적인 로직을 처리한뒤 접근하게 됩니다.

사용 이유 : 대상 클래스가 민감한 정보를 가지고 있거나 인스턴스화 하기에 무겁거나 추가 기능을 가미하고 싶은데, 원본 객체를 수정할수 없는 상황일 때를 사용하며 다음과 같은 부수 효과가 있다.

1. **보안(Security)** : 프록시는 클라이언트가 작업을 수행할 수 있는 권한이 있는지 확인하고 검사 결과가 긍정적인 경우에만 요청을 대상으로 전달한다.&#x20;
2. **캐싱(Caching)** : 프록시가 내부 캐시를 유지하여 데이터가 캐시에 아직 존재하지 않는 경우에만 대상에서 작업이 실행되도록 한다.&#x20;
3. **데이터 유효성 검사(Data validation)** : 프록시가 입력을 대상으로 전달하기 전에 유효성을 검사한다.&#x20;
4. **지연 초기화(Lazy initialization)** : 대상의 생성 비용이 비싸다면 프록시는 그것을 필요로 할때까지 연기할 수 있다.&#x20;
5. **로깅(Logging)** : 프록시는 메소드 호출과 상대 매개 변수를 인터셉트하고 이를 기록한다. 원격 객체(Remote objects) : 프록시는 원격 위치에 있는 객체를 가져와서 로컬처럼 보이게 할 수 있다.

<figure><img src="../../.gitbook/assets/image (18).png" alt="" width="563"><figcaption></figcaption></figure>

1. Subject : Proxy와 RealSubject를 하나로 묶는 인터페이스 &#x20;
   * 대상 객체와 프록시 역할을 동일하게 하는 추상 메소드 operation() 를 정의한다.&#x20;
   * 인터페이스가 있기 때문에 클라이언트는 Proxy 역할과 RealSubject 역할의 차이를 의식할 필요가 없다.
2. RealSubject : 원본 대상 객체&#x20;
3. Proxy : 대상 객체(RealSubject)를 중계할 대리자 역할
   * 프록시는 대상 객체를 합성(composition)한다.&#x20;
   * 프록시는 대상 객체와 같은 이름의 메서드를 호출하며, 별도의 로직을 수행 할수 있다 (인터페이스 구현 메소드)
   * &#x20;프록시는 흐름제어만 할 뿐 결과값을 조작하거나 변경시키면 안 된다.
4.  Client : Subject 인터페이스를 이용하여 프록시 객체를 생성해 이용.

    클라이언트는 프록시를 중간에 두고 프록시를 통해서 RealSubject와 데이터를 주고 받는다.

```java
interface Subject {
    void action();
}

class RealSubject implements Subject {
    public void action() {
        System.out.println("원본 객체 액션 !!");
    }

class Proxy implements Subject {
    private RealSubject subject; // 대상 객체를 composition

    Proxy(RealSubject subject) {
        this.subject = subject;
    }

    public void action() {
        subject.action(); // 위임
        /* do something */
        System.out.println("프록시 객체 액션 !!");
    }
}
```

### 3-2.  스프링의 Proxy

스프링의 Proxy는 리플렉션과 바이트코드 조작을 이용해 실제 타겟의 기능을 대신 수행하면서, 기능을 확장하거나 추가할 수도 있는(OCP원칙) 다이나믹 프록시 객체를 의미로 \[런타임에 프록시 인스턴스가 동적으로 변경되는] 다이나믹 프록시 기법으로 구현되어져있다.

Spring Aspect는 Target 객체를 감싸는 프록시 형태로 구현되며, 이 프록시는 먼저 호출을 가로챈 후 추가적인 Aspect 로직을 수행 하고 나서야 Target Method를 호출 한다.

<figure><img src="../../.gitbook/assets/image (177).png" alt=""><figcaption></figcaption></figure>

#### 3-2-1. Pointcut 표현식

* execution : 메소드 실행 Join Point와 일치 시키는데 사용&#x20;
* within : 특정 타입에 속하는 Join Point 정의&#x20;
* bean : bean 이름으로 pointcut

```java
package co.kr.abacus

public interface SvcByEntrService {
     public void saveService
}

```

<figure><img src="../../.gitbook/assets/image (178).png" alt=""><figcaption></figcaption></figure>

1. 메소드 실행 시작
2. 메소드 명세
3. 리턴 타입 지정
   * \* : 모든 리턴 타입 허용
   * void : 리턴 타입이 void인 메소드
   * !void : 리턴 타입이 void가 아닌 메소드    &#x20;
4. 메소드가 속하는 타입
   * 패키지 + 클래스
   * 패키지 지정
     * co.kr.abacus : 해당 패키지만
     * co.kr.abacus.. : 지정된 패키지로 시작하는 모든 패키지
   * 클래스
     * FullName (SvcByEntrService ) : 헤당 클래스만
     * \*Service : 이름이 Service로 끝나는 클래스
     * Service+: 클래스 이름 뒤에 '+'가 붙으면 해당 클래스로부터 파생된 모든 자식 클래스 선택, 인터페이스 이름 뒤에 '+'가 붙으면 해당 인터페이스를 구현한 모든 클래스
5. 메소드
   * \*: 모든 메소드 선택&#x20;
   * save\* : save로 시작 하는 모든 메소드
6. 인자
   * (..) : 모든 매개변수
   * &#x20;(\*) : 반드시 1개의 매개변수를 가지는 메소드만 선택
   * (Fullpackage) : 매개변수로 작성된 Class가 가지고 있는 메소드
   * (!Fullpackage) : 매개변수로 작성된 Class를 가지지 않는 메소드
   *   (Integer, ..) : 한 개 이상의 매개변수를 가지되, 첫 번째 매개변수

       &#x20;    의 타입이 Integer인 메소드만
   *   (Integer, \*) : 반드시 두 개의 매개변수를 가지되, 첫 번째 매개변수

       &#x20;    의 타입이 Integer인 메소드만  -
7. 범위 제한
   * Execution ( \* co.kr.abacus.SvcByEntrService.saveService(…))    && within(co.kr.abacus.\* )
   * 조합 및 연산자&#x20;
     * &#x20;&& : and 연산자&#x20;
     * || : or 연산자&#x20;
     * ! : 부정

## 4. Sprig AOP – JoinPoint Interface

Advice 메소드를 의미 있게 구현하려면 클라이언트가 호출한 비즈니스 메소드의 정보가 필요하다. 예를 들면 예외가 터졌는데, 예외발생한메소드의 이름 등을 기록할 필요가 있을 수 있다. 이럴때 JoinPoint 인터페이스가 제공하는 유용한 API들이 있다.

* Signature getSignature() : 클라이언트가 호출한 메소드의 시그니처(리턴타입, 이름, 매개변수) 정보가 저장된 Signature 객체 리턴
* Object getTarget() : 클라이언트가 호출한 비즈니스 메소드를 포함하는 비즈니스 객체 리턴
* Object\[] getArgs() : 클라이언트가 메소드를 호출할 때 넘겨준 인자 목록을 Object 배열 로 리턴
* String getName() : 클라이언트가 호출한 메소드 이름 리턴
* String toLongString() : 클라이언트가 호출한 메소드의 리턴타입, 이름, 매개변수(시그니처)를 패키지 경로까지 포함 하여 리턴
* String toShortString() : 클라이언트가 호출한 메소드 시그니처를 축약한 문자열로 리턴

```java
   @Around("entrBySvc()")
   public void watchEntrBySvc(ProceedingJoinPoint  pjp) {
      logger.info("start - " + pjp.getSignature().getDeclaringTypeName() + " / " + pjp.getSignature().getName());
      try {
         pjp.proceed();
      } catch (Throwable e) {
         e.printStackTrace();
      }
      logger.info("finished - " + pjp.getSignature().getDeclaringTypeName() + " / " + pjp.getSignature().getName());
   }
}
```

## 5. 예제

### 5-1. pom.xml

```java
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-aop</artifactId>
</dependency>

```

### 5-2. Application Start

```java
@EnableAspectJAutoProxy // 이존성 주입 
@SpringBootApplication
public class ExpAopApplication {
   public static void main(String[] args) {
      SpringApplication.run(ExpAopApplication.class, args);
   }
}

@Component
public class ExpAopRunner implements ApplicationRunner {

   @Autowired
   private EntrBySvcService entrBySvcService;
	
    @Autowired
    private EntrService entrService;
	
	
    @Override
    public void run(ApplicationArguments args)          throws Exception {
       System.out.println("*** Spring Autowired *************");
       entrService.entrServcie();
       entrBySvcService.entrBySvc();
    }
}

@Configuration
public class EntrBySvcConfig {	
    @Bean
    public EntrServiceAspect entrBySvcBeanAspect() {
       return new EntrServiceAspect(); // 의존성 주입 
    }
}

@Aspect
public class EntrServiceAspect {
   Logger logger = LoggerFactory.getLogger(EntrServiceAspect.class);
….
}


@Aspect
@Component
public class EntrBySvcAspect {
   Logger logger = LoggerFactory.getLogger(EntrBySvcAspect.class);	
   // Pointcut 정의 
   @Pointcut("execution(* co.kr.abacus.spring.aop.entrsvc.service.EntrBySvcService.entrBySvc(..))")
   public void entrBySvc() {};
	
   //@Before("execution(* co.kr.abacus.spring.aop.entrsvc.service.EntrBySvcService.entrBySvc(..))")
   @Before("entrBySvc()")
   public void beforeService() { 
      logger.info("*** 실행 이전 "); 
   }
   
   @AfterReturning("entrBySvc()")
   public void afterReturningService() { 
      logger.info("*** 실행 성공  "); 
   }	
   
   @AfterThrowing("entrBySvc()")
   public void AfterThrowingService() { 
      logger.info("*** 실행 실패  "); 
   }
   
   @Around("entrBySvc()")
   public void watchEntrBySvc(ProceedingJoinPoint  pjp) {
      logger.info("start - " + pjp.getSignature().getDeclaringTypeName() + " / " + pjp.getSignature().getName());
      try {
         pjp.proceed();
      } catch (Throwable e) {
         e.printStackTrace();
      }
      logger.info("finished - " + pjp.getSignature().getDeclaringTypeName() + " / " + pjp.getSignature().getName());
   }
}
```

<figure><img src="../../.gitbook/assets/image (179).png" alt=""><figcaption></figcaption></figure>

참고 : [https://jiwondev.tistory.com/152](https://jiwondev.tistory.com/152)

