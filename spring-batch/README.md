# Spring Batch

많은 데이터를 일괄처리할 수 있도록 도와주는 경량화된 프레임워크로 Spring 기반 일괄처리를 효율적으로 처리 하기 위한 기능을 제공합니다.

## 1. 주요 특징

* **Spring Boot 통합** : Spring Boot Batch는 Spring Boot 프레임워크와 통합되어 있어 설정이 간단하다.
* **간결한 코드** : 간결한 코드로 배치 작업 구현&#x20;
* **내장된 기능** : 실패한 작업을 다시 실행하거나 작업 상태를 모니터링 하는 기능과 같은 다양한 내장 기능 제공&#x20;
* **풍부한 지원** : Spring Boot 커뮤니티와 지원을 통해 문제 해결과 지속적인 업데이트

## 2. 주요 특징

* **로깅/추적 (Logging/Tracing)**: 배치 작업의 실행 상태를 기록하고 추적.&#x20;
* **트랜잭션 관리 (Transaction Management)**: 안전한 데이터 처리를 위해 트랜잭션을 관리.&#x20;
* **작업 처리 통계 (Job Processing Statistics)**: 작업의 성공, 실패, 건너뛰기 등의 통계 정보를 수집.&#x20;
* **작업 재시작 (Job Restartability)**: 작업이 중단되었을 때 이전 상태에서 다시 시작할 수 있도록 지원.&#x20;
* **건너뛰기 (Skip)**: 특정 조건에서 오류가 발생해도 작업을 건너뛸 수 있음.&#x20;
* **리소스 관리 (Resource Management)**: 데이터베이스 연결, 파일 I/O 등 리소스를 효율적으로 관리&#x20;
* **청크기반 처리 (Chunk-based processing)**: 대규모 데이터 세트를 관리 가능한 청크로 분할하여 처리&#x20;
* **확장성 및 병렬 처리**: 수평으로 확장할 수 있는 기능을 갖추고 있어 여러 시스템에서 병렬 처리를 지원

## 3. 적용사례

* Data Migration.&#x20;
* 주기적 동기화 작업.&#x20;
* 보고서 생성&#x20;
* 데이터 정리.

## 4. 주요 구성 요소

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>



## 5. Spring Batch Meta-Data Schema

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 5-1 Job 관련 테이블

* BATCH\_JOB\_INSTANCE&#x20;
  * Job이 실행될 때 Job Instance 정보가 저장되며 job\_name과 job\_key를 키로 하여 하나의 데이터가 저장&#x20;
  * 동일한 job\_name과 job\_key로 중복 저장될 수 없음
* &#x20;BATCH\_JOB\_EXECUTION
  * &#x20;Job의 실행정보가 저장되며 Job 생성, 시작, 종료 시간, 실행 상태, 메시지 등을 관리&#x20;
* BATCH\_JOB\_EXECUTION\_PARAMS&#x20;
  * Job과 함께 실행되는 Job Parameter 정보를 저장&#x20;
* BATCH\_JOB\_EXECUTION\_CONTEXT
  * &#x20;Job의 실행동안 여러가지 상태정보, 공유 데이터를 직렬화 (Json 형식) 해서 저장 Step 간 서로 공유 가능함

### 5-2. Step 관련 테이블

* BATCH\_STEP\_EXECUTION&#x20;
  * Step의 실행 정보가 저장되며 생성, 시작, 종료 시간, 실행 상태, 메시지 등을 관리&#x20;
* BATCH\_STEP\_EXECUTION\_CONTEXT&#x20;
  * Step의 실행동안 여러가지 상태 정보, 공유 데이터를 직렬화(Json 형식) 해서 저장 Step 별로 저장되며 Step 간 서로 공유할 수 없음

## 6. 의존성

### 6-1. pom.xml

```yaml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>

<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
    <version>3.3.3</version>
    <scope>runtime</scope>
</dependency>

<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 6-2. jpa

```yaml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

```yaml
spring:
    application:
        name: base_springboot_bacth
    datasource:
        driver-class-name: org.mariadb.jdbc.Driver
        url: jdbc:mariadb://x.x.x.x:14302/hong
        username: hong
        password: hong1234
    jpa: open-in-view: false
        show-sql: true
        hibernate:
            dialect: org.hibernate.dialect.MariaDB102Dialect
```

### 6-3. mybatis

```yaml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
```

```yaml
spring:
  application:
    name: base_springboot_bacth
  batch:
    jdbc:
      initialize-schema: never # always # never
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://x.x.x.x:14302/hong
    username: hong
    password: hong1234\
mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: : kr.co.abacus.batch.**.dto
```

### 6-4. Jpa + mybatis

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency><dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
```

```yaml
spring:
  application:
    name: base_springboot_bacth
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://x.x.x.x:14302/hong
    username: hong
    password: hong1234
  jpa:
    open-in-view: false
    show-sql: true
    hibernate:
      dialect: org.hibernate.dialect.MariaDB102Dialect
mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: : kr.co.abacus.batch.**.dto
```
