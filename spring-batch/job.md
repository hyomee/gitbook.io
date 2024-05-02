# Job 실행

스프링 배치에서 Job을 실행은 Job Runner에서 시작을 하는데 다음과 같은 방법이 있다.

* CommandLineJobRunner:&#x20;
* JobRegistryBackgroundJobRunner
* JobLauncherCommandLineRunner

## 1. CommandLineJobRunner

스프링 배치(Job)을 명령행에서 실행하기 위한 도구로 터미널 또는 명령 프롬프트에서 스프링 배치 Job를 간단하게 실행한다.

* 용법: CommandLineJobRunner jobPath jobIdentifier (jobParameters)
* 예:  java -cp "target/dependency-jars/\*:target/your-project.jar" org.springframework.batch.core.launch.support.CommandLineJobRunner \
  spring/batch/jobs/job-read-files.xml \
  readJob \
  file.name=testing.cvs
  * your-project.jar: 스프링 배치 애플리케이션의 JAR 파일 경로
  * jobPath: Job 설정 파일의 경로
  * jobIdentifier: 실행할 Job의 식별자
  * jobParameters: file.name=testing.cvs

## 2. JobRegistryBackgroundJobRunner

스프링 배치(Job)을 명령행에서 실행하기 위한 도구로 터미널 또는 명령 프롬프트에서 스프링 배치 Job을 간단하게 실행할 수 있으며 주로 주로 외부 트리거와 함께 사용한다. ( 예: JobLauncher를 위한 JMX MBean 래퍼나 Quartz 트리거와 같은 외부 트리거와 연동하여 등록된 Job을 실행 )

```java
@Deprecated(since="5.0", forRemoval=true) 
public class JobRegistryBackgroundJobRunner extends Object
```

* java -cp your-application.jar org.springframework.batch.core.launch.support.JobRegistryBackgroundJobRunner jobPath jobIdentifier (jobParameters)
  * your-project.jar: 스프링 배치 애플리케이션의 JAR 파일 경로
  * jobPath: Job 설정 파일의 경로
  * jobIdentifier: 실행할 Job의 식별자
  * jobParameters: file.name=testing.cvs

## 3. JobLauncherCommandLineRunner

Spring Boot에서 제공되며, 배치 작업을 실행하는 데 유용하며기본적으로 주변 컨텍스트에 있는 모든 작업을 실행하며, 작업 이름을 제공하여 특정 작업을 실행할 수도 있다.

```java
@Deprecated
public class JobLauncherCommandLineRunner
extends JobLauncherApplicationRunner
```

```
spring.batch.job.enabled = true # 자동 생성
spring.batch.job.names = testJob, test2Job # , 구분으로 지정된 Job 실행, 만약 존재하지 않으면 모든 Job을 실행
```

## 4. 잡(Job) 구성 요소

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 4-1. JobLauncher

* Job을 실행하는 기능으로 Job.execute 을 호출한다.
* 잡의 재실행 가능 여부 검증, 잡의 실행 방법, 파라미터 유효성 검증 등을 수행한다.

### 4-2. Job Repository

* 배치 수행과 관련된 수치 데이터와 잡의 상태를 유지 및 관리한다.
* 배치 작업에 대한 메타데이터를 저장(실행된 스텝, 현재 상태, 읽은 아이템 및 처리된 아이템 수 등이 모두 JobRepository 에 저장)

### 4-3. Job, Step

#### 4-2-1. Job

* 처음부터 끝까지 실행할 수 있는 완전한 일괄 처리 프로세스
* 작업은 XML 또는 Java 기반 구성을 사용하여 정의
* 1개 이상의 step 구성

#### 4-2-2. Step

* 일괄처리 작업 내의 작업 단위 ( 잡을 구성하는 독립된 작업의 단위 )
* Tasklet, Chunk 기반으로 2가지가 있다.
  * Chunk-Oriented Step: 데이터를 청크로 처리
    * ItemReader, ItemProcessor, ItemWriter 라는 3개의 주요 부분으로 구성이 되며 ItemProcessor는 필수 사항이 아니다.
    * ItemReader: 파일 또는 데이터베이스와 같은 원본에서 데이터를 검색
      * ItemReader : JdbcCursorItemReader, FlatFileItemReader
    * ItemProcessor: 입력 데이터를 변환하거나 처리
      * ItemProcessor : ItemProcessorAdapter, CompositeItemProcessor
    * ItemWriter : 처리된 데이터를 파일 또는 데이터베이스와 같은 대상에 저장
      * ItemWriter: MapJobRepository, JobRepositoryFactoryBean
  * Tasklet Step  단일 작업을 실행
    * Step이 중지될 떄 까지 execute 메서드가 실행이 되며 트랜잭션 단위가 된다.

### 4-4. Job/Step 실행 완료

실행 완료시, JobRepository 내에 있는 JobExecution 또는 StepExecution 이 최종 상태로 업데이트 된다.

#### 4-3-1. JobInstance

* 스프링 배치 잡의 논리적인 실행 단위로 Job을 각 다른 파라미터로 실행될 때마다 새로운 JobInstance 가 생성되며 여러개의 JobExecution 을 가질 수 있다.

#### 4-3-2. JobExecution

* 스프링 배치 Job의 실제 실행을 의미하며 Job이 실행될 때마다 매번 새로운 JobExecution 을 얻는다.

#### 4-3-3. StepExecution

* 스프링 배치 스텝(Step) 의 실제 실행을 의미한다

#### 4-3-4. JobExplorer&#x20;

* 기본적으로 작업 인스턴스, 실행 및 관련 단계를 검사할 수 있는 쿼리 API.&#x20;

#### 4-3-5. JobOperator&#x20;

* 작업 시작, 중지 및 쿼리와 같은 작업을 허용하는 더 높은 수준의 API.
