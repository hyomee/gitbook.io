# Hello World

## &#x20;Hello World 콘솔 출력

초간단 Hello World를 출력 하는 예제를 통해서 스프링 배치 5를 이해하고자 합니다.

## 2. 요구사항

* Java  Java 17+
* Spring Boot: spring-boot-starter-parent: v3.2.4
* Spring Batch: spring-boot-starter-batch
* DB: 마리아 DB&#x20;

## 3. 요구사항

Step 2개를 만들고 각각 다음과 같이 출력을 하세요.

* Step 1: "안녕하세요. 난 Step 1번 입니다.  화살표 함수로 작성된 코드 입니다."
* Step 2:"안녕하세요. 난 Step 2번으로. Tasklet를 상속 받아 execute를 구현 했습니다."

## 4. 프로젝트&#x20;

### 4-1. 프로젝트 생성  - 의존성 구성&#x20;

[https://start.spring.io/](https://start.spring.io/) 에서 프로젝트를 생성하고 다운로드 받는다. ( Maven )

<figure><img src="../../.gitbook/assets/image (195).png" alt=""><figcaption><p>프로젝트 구성</p></figcaption></figure>



<details>

<summary>pom.xml</summary>

{% code lineNumbers="true" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.2.4</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>21</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-batch</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.mariadb.jdbc</groupId>
			<artifactId>mariadb-java-client</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.batch</groupId>
			<artifactId>spring-batch-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<excludes>
						<exclude>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
						</exclude>
					</excludes>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>
```
{% endcode %}

</details>

### 4-2. 환경 구성&#x20;

#### 4-2-1. 테이블 생성&#x20;

스프링 배치가 구동 되기 위해서는 기본적으로 Job관련 테이블 4개, Step 관련 테이블 2개이 구성이 되어 있어여 합니다. JPA를 사용해서 테이블을 생성 할 수 있지만 추전하는 것이 아니어서 먼저 DB에 생성 해야 합니다. 생성 스크립트는 다음과 같습니다.

참고 : [Appendix B. Meta-Data Schema](https://docs.spring.io/spring-batch/docs/3.0.x/reference/html/metaDataSchema.html)

<details>

<summary>테이블 생성 스크립트</summary>

```sql
// Identity
CREATE SEQUENCE BATCH_STEP_EXECUTION_SEQ;
CREATE SEQUENCE BATCH_JOB_EXECUTION_SEQ;
CREATE SEQUENCE BATCH_JOB_SEQ;

// MySQL Sample
CREATE TABLE BATCH_STEP_EXECUTION_SEQ (ID BIGINT NOT NULL) type=InnoDB;
INSERT INTO BATCH_STEP_EXECUTION_SEQ values(0);
CREATE TABLE BATCH_JOB_EXECUTION_SEQ (ID BIGINT NOT NULL) type=InnoDB;
INSERT INTO BATCH_JOB_EXECUTION_SEQ values(0);
CREATE TABLE BATCH_JOB_SEQ (ID BIGINT NOT NULL) type=InnoDB;
INSERT INTO BATCH_JOB_SEQ values(0);

// JOB 관련 
CREATE TABLE BATCH_JOB_INSTANCE  (
  JOB_INSTANCE_ID BIGINT  PRIMARY KEY ,
  VERSION BIGINT,
  JOB_NAME VARCHAR(100) NOT NULL ,
  JOB_KEY VARCHAR(2500)
);

// JOB 파라메터 
CREATE TABLE BATCH_JOB_EXECUTION_PARAMS  (
	JOB_EXECUTION_ID BIGINT NOT NULL ,
	TYPE_CD VARCHAR(6) NOT NULL ,
	KEY_NAME VARCHAR(100) NOT NULL ,
	STRING_VAL VARCHAR(250) ,
	DATE_VAL DATETIME DEFAULT NULL ,
	LONG_VAL BIGINT ,
	DOUBLE_VAL DOUBLE PRECISION ,
	IDENTIFYING CHAR(1) NOT NULL ,
	constraint JOB_EXEC_PARAMS_FK foreign key (JOB_EXECUTION_ID)
	references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
);

// JOB실행 
CREATE TABLE BATCH_JOB_EXECUTION  (
  JOB_EXECUTION_ID BIGINT  PRIMARY KEY ,
  VERSION BIGINT,
  JOB_INSTANCE_ID BIGINT NOT NULL,
  CREATE_TIME TIMESTAMP NOT NULL,
  START_TIME TIMESTAMP DEFAULT NULL,
  END_TIME TIMESTAMP DEFAULT NULL,
  STATUS VARCHAR(10),
  EXIT_CODE VARCHAR(20),
  EXIT_MESSAGE VARCHAR(2500),
  LAST_UPDATED TIMESTAMP,
  JOB_CONFIGURATION_LOCATION VARCHAR(2500) NULL,
  constraint JOB_INSTANCE_EXECUTION_FK foreign key (JOB_INSTANCE_ID)
  references BATCH_JOB_INSTANCE(JOB_INSTANCE_ID)
) ;

// STEP 실행
CREATE TABLE BATCH_STEP_EXECUTION  (
  STEP_EXECUTION_ID BIGINT  PRIMARY KEY ,
  VERSION BIGINT NOT NULL,
  STEP_NAME VARCHAR(100) NOT NULL,
  JOB_EXECUTION_ID BIGINT NOT NULL,
  START_TIME TIMESTAMP NOT NULL ,
  END_TIME TIMESTAMP DEFAULT NULL,
  STATUS VARCHAR(10),
  COMMIT_COUNT BIGINT ,
  READ_COUNT BIGINT ,
  FILTER_COUNT BIGINT ,
  WRITE_COUNT BIGINT ,
  READ_SKIP_COUNT BIGINT ,
  WRITE_SKIP_COUNT BIGINT ,
  PROCESS_SKIP_COUNT BIGINT ,
  ROLLBACK_COUNT BIGINT ,
  EXIT_CODE VARCHAR(20) ,
  EXIT_MESSAGE VARCHAR(2500) ,
  LAST_UPDATED TIMESTAMP,
  constraint JOB_EXECUTION_STEP_FK foreign key (JOB_EXECUTION_ID)
  references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
) ;

// BATCH_JOB_EXECUTION_CONTEXT
CREATE TABLE BATCH_JOB_EXECUTION_CONTEXT  (
  JOB_EXECUTION_ID BIGINT PRIMARY KEY,
  SHORT_CONTEXT VARCHAR(2500) NOT NULL,
  SERIALIZED_CONTEXT CLOB,
  constraint JOB_EXEC_CTX_FK foreign key (JOB_EXECUTION_ID)
  references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
) ;

//BATCH_STEP_EXECUTION_CONTEXT
CREATE TABLE BATCH_STEP_EXECUTION_CONTEXT  (
  STEP_EXECUTION_ID BIGINT PRIMARY KEY,
  SHORT_CONTEXT VARCHAR(2500) NOT NULL,
  SERIALIZED_CONTEXT CLOB,
  constraint STEP_EXEC_CTX_FK foreign key (STEP_EXECUTION_ID)
  references BATCH_STEP_EXECUTION(STEP_EXECUTION_ID)
) ;
```

</details>

#### 4-2-2. 구성 요소 설정

application.properties (or application.yml )을 다음과 같이 작성 합니다.

{% code title="application.yml" overflow="wrap" lineNumbers="true" %}
```yaml
spring:
  application:
    name: base_springboot_bacth
  batch:
    jdbc:
      initialize-schema: never  
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://1.217.139.30:14302/hong
    username: hong
    password: hong1234
  jpa:
    show-sql: false
    hibernate:
      dialect: org.hibernate.dialect.MariaDB102Dialect
logging:
  level:
    root: info
    sql: info
    kr.co.abacus: debug
```
{% endcode %}

1. 1\~3: spring.application.name: 스프링 프로젝트 이름
2. 4\~6: spring.batch: 설정 관련 스프링 배치&#x20;
   * jdbc.initialize-schema: 스프링 배치 테이블 생성을 위한 옵션&#x20;
     * ALWAYS: 항상 생성&#x20;
     * EMBEDDED: 내장 데이터베이스에만 생성&#x20;
     * NEVER: 생성하지 않음
3. 7\~11: JDBC 설정&#x20;
4. 12\~16: JPA 설정&#x20;
5. 17\~28: 로그 레벨 설정&#x20;

### 4-3.  코드 작성&#x20;

스프링 배치는 배치 작업 시 Tasklet/chunk 방식으로 처리 할 수 있는데 "helloworld"예제는 Tasklet 방식으로 코드를 작성 합니다.

* Tasklet은 **execute**라는 하나의 메서드를 가지며, 이 메서드는 TaskletStep에서 반복적으로 호출됩니다.
* Tasklet은 **RepeatStatus.FINISHED**를 반환하거나 예외를 throw하여 실패를 알리기까지 호출됩니다.&#x20;
* 각 Tasklet 호출은 트랜잭션으로 래핑됩니다.

#### 4-3-1.  Tasklet 코드 작성

요구사항을 만족 하기 위해서 Step 2애서 실행 할 Tasklet 코드로 Tasklet를 상속 받아 execute를 구현한 구현체 입니다.

{% code title="MyUserTasklet.java" lineNumbers="true" %}
```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.stereotype.Component;


@Component
@Slf4j
 public class MyUserTasklet implements Tasklet {
    @Override
    public RepeatStatus execute(StepContribution contribution,
                                ChunkContext chunkContext) throws Exception {
        log.debug("####-> 안녕하세요. 난 Step 2번으로. T" +
                "asklet를 상속 받아 execute를 구현 했습니다.");
        return RepeatStatus.FINISHED;
    }
}
```
{% endcode %}



* 11\~19: Tasklet를 상속 받아 MyUserTasklet 클래스 구현&#x20;
* 13: execute 메서드 구현&#x20;
* 15\~16: 로그 출력 "MyUserTasklet Hello, World!"&#x20;

#### 4-3-2.  Job/Step 코드 작성&#x20;

{% code title="HelloWorldConfig.java" lineNumbers="true" %}
```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.job.builder.JobBuilder;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.PlatformTransactionManager;

@Configuration
@RequiredArgsConstructor
@Slf4j
public class HelloWorldConfig{

    private final  MyUserTasklet myUserTasklet;

    @Bean
    public Tasklet myTasklet() {
        log.debug("####-> myTasklet!");
        return (contribution, chunkContext) -> {
            log.debug("####->  안녕하세요. 난 Step 1번 입니다." +
                      "화살표 함수로 작성된 코드 입니다");
            return RepeatStatus.FINISHED;
        };
    }

    @Bean
    public Step myStep01(JobRepository jobRepository,
                         Tasklet myTasklet,
                         PlatformTransactionManager transactionManager) {
        log.debug("####->  myStep01!");
        return new StepBuilder("myStep01", jobRepository)
                .tasklet( myTasklet , transactionManager)
                .build();
    }

    @Bean
    public Step myStep02(JobRepository jobRepository,
                         PlatformTransactionManager transactionManager) {
        log.debug("####->  myStep02!");
        return new StepBuilder("myStep02", jobRepository)
                .tasklet( myUserTasklet , transactionManager)
                .build();
    }

    @Bean
    public Job myJob(JobRepository jobRepository,
                         Step myStep01,
                         Step myStep02) {
        log.debug("####->  myJob!");
        return new JobBuilder("myJob", jobRepository)
                .start(myStep01)
                .next(myStep02)
                .build();
    }
}
```
{% endcode %}

* 15 Line: @Configuration 클래스 레밸의 어노테이션으로 하나 이상의 @Bean을 선언하고 있어야 하며 스프링 컨테이너에 Bean 정의 및 요청을 생성하는데 사용. 즉 Bean 주입&#x20;
* 20 Line: 다른 클래스를 사용하기 위해 선언한 맴버로 lombok의 @RequiredArgsConstructor을 사용하였으면 요구사항 Step 2에서 실행 할 객체를 주입한 것입니다.
* 22\~23 Line: 요구사항 Step 1에서 사용 하는 Tasklet으로 화살표함수로 작성한 코드 입니다.
* 33\~49 Line: 스프링 배치에서 Step를 등록 하는 것으로 Spring Batch 5에서는 StepBuilderFactory 는Deprecated되어  사용 할 수 없으며 StepBuilder를 사용해서 Step에서 실행 할 Tasklet를 선언 합니다.
  * myStep01: 화살표 함수로 작성된 myTasklet 실행&#x20;
  * myStep02: Tasklet을 상속받아 구현한  myUserTasklet 실행&#x20;
* 33\~49 Line: 스프림 배치의 Job을 등록 한 것으로  Spring Batch 5에서는 JobBuilderFactory는Deprecated되고 JobBuilder를 사용하여 Job에서 실행하는 Step를 선언 합니다.
  * start: 최초 실행할 Step.
  * next: 다음 실행할 Step.



{% hint style="info" %}
스프링 배치에서랜트잭션을 분리 하거나 데이터 소스를 다른 것을 사용 하기 위해서 Job/Step에    transactionManager,  jobRepository를 지정 할 수 있습니다.
{% endhint %}

#### &#x20;4-3-3.  JobLauncher 작성

스프링 배치의 설정에서 JobLauncher 를자동 실행 할 수 있자만 파라메터 조작, 스케줄 등을 고려해서 직접 실향하는 코드로 작성합니다.

{% code title="HelloWorldBacthApplication.java" lineNumbers="true" %}
```java
@SpringBootApplication
public class HelloWorldApplication {

    public static void main(String[] args) {

        ConfigurableApplicationContext context = 
              SpringApplication.run(HelloWorldBacthApplication .class, args);

        JobLauncher jobLauncher = context.getBean(JobLauncher.class);
        Job helloJob = context.getBean("myJob", Job.class);

        try {
            jobLauncher.run(helloJob, new JobParametersBuilder()
                    .addLong("timestamp", System.currentTimeMillis())
                    .toJobParameters());
        } catch (Exception e) {
            e.printStackTrace();
        }

        context.close();
    }

}
```
{% endcode %}

* 6 Line:  JobLauncher를 실행 하기 위해서 스프링 배치가 구동 된 후 ApplicationContext을 얻습니다.
* 9 Line: getBean 메서드를 사용해서 JobLauncher.class를 선언 합니다.
* 10 Line: 실행할 job을 선언 합니다.
* 13\~20 Line: jobLauncher에 job를 등록 하고 실향 합니다.
  * JobParameter로 현재 시간을 등록 한 이유는 배치가 실행 될 떄 마가 job을 실행 하기 위해서 입니다,
  * 만약, 파라메터의 변경이 없다면 스프림 배치는 기 실행 건으로 실행이 되지 않습니다.

#### &#x20;4-3-4.  결과

* 스프링"배치  시작 전에 HelloWorldConfig가 로딩 되면서 job, step, tasklet이 등록 되는 과정을 로그를 통해서 확인 할 있습니다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 스프링 배치가 실행 되고 job에 정의한 step01, step02가 순서적으로 실행 되면서 tasklet를 실행 한 로그를 확인 할 수 있습니다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 5. 개선 과제

HelloWorldApplication.java 파일에서 job명이 하드코딩 되어 있습니다. &#x20;

```
Job helloJob = context.getBean("myJob", Job.class)
```

동적으로 job을 실행 하기 위해서는 어떻게 해야 할 까요 ?

