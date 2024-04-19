# Spring Framework 격리 레벨

Transactional 애노테이션에서 isolation(격리수준)은 명시적으로 어떤 격리 수준도 설정하지 않고, 데이터베이스가 기본적으로 사용하는 격리 수준을 사용한다는 의미한다.

```java
public @interface Transactional {
    Isolation isolation() default Isolation.DEFAULT;
}
```

[참고 ](https://stackoverflow.com/questions/8490852/spring-transactional-isolation-propagation)[: java - Spring @Transactional - isolation, propagation - Stack Overflow](https://stackoverflow.com/questions/8490852/spring-transactional-isolation-propagation)

* ISOLATION\_READ\_UNCOMMITTED: 더티 리드를 허용.
* ISOLATION\_READ\_COMMITTED: 더티 리드를 허용하지 않음.&#x20;
* ISOLATION\_REPEATABLE\_READ: 동일한 트랜잭션 내에서 두 번 이상 동일한 행을 읽으면 항상 동일한 결과를 반환&#x20;
* ISOLATION\_SERIALIZABLE: 모든 트랜잭션을 순차적으로 실행.

## 1. @Transactional Propagation Levels

Spring에서 @Transactional 어노테이션의 전파 레벨(Propagation Level)은 트랜잭션의 경계를 어떻게 설정할지 결정하는 옵션으로 기본적으로 Propagation.REQUIRED가 설정되어 있으며, 이는 부모 트랜잭션이 존재하면 현재 메소드를 부모 트랜잭션에 포함시키고, 존재하지 않으면 새로운 트랜잭션을 시작하는 것을 의미한다.

**Spring 사용방법 : @Transactional(propagation = Propagation.NEVER)**

### 1-1. 트랜잭션 격리 수준(Transaction Isolation Level)

#### 1-1-1.  REQUIRED: Propagation.REQUIRED&#x20;

부모 트랜잭션이 존재하면 현재 메소드를 부모 트랜잭션에 포함시키고, 존재하지 않으면 새로운 트랜잭션을 시작 - 자식/부모에서 rollback이 발생된다면 자식과 부모 모두 rollback&#x20;

#### 1-1-2.  SUPPORTS: Propagation.SUPPORTS&#x20;

부모 트랜잭션이 존재하면 포함되고, 없으면 트랜잭션 없이 실행&#x20;

#### 1-1-3. MANDATORY:&#x20;

REQUIRED와 비슷하게 이미 시작된 트랜잭션이 있으면 참여한다. 하지만, 트랜잭션이 시작된 것이 없으면 예외를 발생, 혼자서는 독립적으로 트랜잭션을 진행하면 안되는 경우에 사용&#x20;

#### 1-1-4. REQUIRES\_NEW: Propagation.REQUIRES\_NEW&#x20;

항상 새로운 트랜잭션을 시작 - 자식쪽에 예외가 발생할 경우 자식쪽은 트랜잭션이 롤백, 이 때 부모쪽은 트랜잭션이 전파가 되지 않지만 예외는 전파, - 부모쪽에서 그 예외를 자식으로부터 받았기 때문에 부모쪽에도 예외가 전파되어 롤백

#### 1-1-5. NOT\_SUPPORTED: Propagation.NOT\_SUPPORTED&#x20;

트랜잭션을 사용하지 않음&#x20;

#### 1-1-6. NEVER: Propagation.NEVER&#x20;

트랜잭션을 사용하지 않으며, 부모 트랜잭션이 존재하면 예외를 발생&#x20;

#### 1-1-7. NESTED: Propagation.NESTED&#x20;

부모 트랜잭션이 존재하면 중첩된 트랜잭션을 시작

## 2. @Transactional 그외 Option

### 2-1. 트랜잭션 롤백(Transaction Rollback) 규칙&#x20;

roolbackFor 및 noRollbackFor annotation Parameter를 사용하여 트랜잭션 rollback을 할지 안 할지에 대한 설정을 한다.

* 롤백 지정 : @Transactional(rollbackFor = { SQLException.class }) \
  \- SQLException이 발생하는 경우 트랜잭션을 롤백하도록 지정한다.
* 롤백 예외 : @Transactional(noRollbackFor = { NullPointerException.class }) \
  \- NullPointerException으로 인해 트랜잭션이 롤백되지 않도록 지정한다.
* 주의사항 : try-catch 로 에러를 핸드링하여도 트랜잭션 관리자는 트랜잭션을 롤백 한다. \
  \- noRollbackFor 속성을 사용하여 롤백되지 않도록 하여야 한다. \
  \- @Transactional(noRollbackFor = { CustomException.class })&#x20;

### 2-2. 시간제한(Transaction Timeout)

* @Transactional(timeout = 10) - 트랜잭션이 주어진 시간 내에 완료  한다.\
  \- 그렇지 않으면 트랜잭션을 롤백시키며 트랜잭션 예외(트랜잭션 시간 만료 오류)가 발생&#x20;

### 2-3. readOnly Flag&#x20;

* @Transactional(readOnly = true) - 트랜젝션을 읽기 전용으로 설정한다.
* <mark style="color:purple;">JPA에서의 ReadOnly Flag</mark>:&#x20;
  * 스프링 프레임워크가 세션 플러시 모드를 MANUAL로 설정&#x20;
  * 강제로 플러시를 호출하지 않는 한 플러시가 일어나지 않음 \
    \-> 트랜잭션이 커밋되면서 실수로 엔티티가 등록, 수정, 삭제되는 일을 방지 할 수 있음&#x20;
  * <mark style="color:orange;">트랜잭션 내에서 데이터를 수정해야 하는 경우 읽기 전용 트랜잭션을 사용해서는 안 됩니다</mark>

## 3. @Transactional 함정

Spring 프록시는 Class Level 프록시로 동일 Class에서 중첩된 propagation에 대해서는 적용되지 않는다.

<figure><img src="../.gitbook/assets/image (203).png" alt="" width="563"><figcaption></figcaption></figure>

```java
@Service
public class UserService {
    @Autowired 
    private ProductService productService;
    
    @Transactional
    public void createUser() {
        userEntity.save();
        productService.save();
    }
    
    @Transactional(Propagation.REQUIRES_NEW)
    public void createAddress() {
        userAddressEntity.save();
    }   
}

@Service
public class ProductService {
    @Transactional
    public void save() {
        // ...
    }
}
```

<figure><img src="../.gitbook/assets/image (202).png" alt="" width="563"><figcaption></figcaption></figure>

## 4. 트랜잭션 동작 방식

### 4-1. 일반 JDBC 트랜잭션 동작 방식

<figure><img src="../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

1. 데이터베이스 연결 \
   \- url, id, pwd 로 데이터 베이스에 연결 \
   \-  엔터프라이즈 응용 프로그램은 환경 구성을 통해 연결하고 데이터 소스를 가져옴&#x20;
2. 트랜잭션 시작 \
   \- setAutoCommit(true) 는 모든 단일 SQL 문이 자동으로 자체 트랜잭션에 래핑 \
   \- setAutoCommit(false) 는 모든 단일 SQL 문이 자동으로 자체 트랜잭션에 래핑 하지 않음&#x20;
3. 트랜잭션 커밋 - 수행된 쿼리 모든 쿼리에 대해 커밋&#x20;
4. 예외가 있는 경우 변경 사항을 롤백

### 4-2. Spring 동작 방식

```java
@Service
public class UserService {
    @Autowired
    private TransactionTemplate template;
    
    public Long registerUser(User user) {
        Long id = template.execute(status ->  {
             // // SQL 구문 실행...
             return id;
        });
    }
}
```

1. JDBC 동작 방식의 데이터 베이스 연결, 커밋, 롤백은 Spring에서 담당 \
   \- 트랜잭션 콜백을 사용 \
   \- Spring이 이러한 예외를 런타임 예외로 변환하므로 SQLExceptions를 잡을 필요가 없음. \
   \- TransactionTemplate은 내부적으로 데이터 소스를 사용하는 TransactionManager를 사용 \
   \- Spring 컨텍스트 구성에서 DataSource, SqlSessionFactory 등 지정
2. SQL 쿼리 만 실행

참고 : [16. Transaction Management (spring.io)](https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/transaction.html)

**@Transactional 어노테이션 사용 : 선언적 트랜잭션 관리(Declarative Transaction Management): 적용으로 변경**&#x20;

```java
@Service
public class UserService {
    @Autowired
    private TransactionTemplate template;
    
    @Transactional
    public Long registerUser(User user) {
        Long id = template.execute(status ->  {
             // // SQL 구문 실행...
             return id;
        });
    }
}
```

1. Spring Configuration에 @EnableTransactionManagement 사용 \
   \- Spring Boot 는 자동 수행 됨 \
   \- Spring Configuration에서 트랜잭션 관리자를 지정&#x20;
2. @Transactional 주석으로 주석을 달았던 모든 빈의 공개 메소드는 데이터베이스 트랜잭션 내에서 실행

```java
@Configuration
@EnableTransactionManagement
public class MySpringConfig {
    @Bean
    public PlatformTransactionManager txManager() {
        return yourTxManager;
    }
}
```

참고 : [Using @Transactional: Spring Framework](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)

### 4-3. Spring 동작 방식 - CGlib & JDK Proxy

프록시로 작업됨

<figure><img src="../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

1. 데이터베이스 연결/트랜잭션 열기 및 닫기 \
   \- 트랜잭션 상태(열기, 커밋, 닫기)를 처리하는 프록시 자체가 아니라 트랜잭션 관리자에게 트랜잭션은 위임함
2. UserService에 위임&#x20;
3. UserRestController는 Proxy를 알지 못 한다.

<figure><img src="../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>

**논리적 트랜잭션으로 변경**&#x20;

<figure><img src="../.gitbook/assets/image (210).png" alt="" width="563"><figcaption></figcaption></figure>
