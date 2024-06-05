# 배치 파라메터 전달

스프링 배치 실행시 받은 파라메터를 Tasklet으로 전달하는 방법은 여러 방법이 있지만 여기에서는 Map으로 변환 후 JobParameter로 설정하는 방법에 대해서 소개합니다.

## 1. 요구사항

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

1. 배치 실행시 받은 파라메터를 JobParameter로 설정하여 배치 프로그램에서 사용할 수 있어여 한다.
2. 베치 실행시 현재 일자가 자동 설정 되어야 한다.

## 2. 생각하기

자바로 작성된 모든 코드는 main 메서드는 String 배열을 통해 데이터를 받습니다. 이것은 스프링으로 작성한 코드에도 동일하게 적용 되므로 다음과 같은 기능을 작성합니다.&#x20;

1. 스프링 구동시 Arguments를 받아서 JobLancher를 실행하기 위해 다양한 방법이 있지만 여기서는 ApplicationRunner 인터페이스의 구현 코드를 작성한다. \
   \-[ <mark style="color:purple;">3-1. AcubeBatchCommandLineRunner.java)</mark> ](springbatchargs.md#id-3-1.-applicationrunner)
2. Main 에서 받은 Arguments는 ApplicationRunner 인터페이스 구현체의 ApplicationArguments 로 전달 되므로 ApplicationArguments 를 Map으로 변환 하는 메서드 생성 \ <mark style="color:purple;">-</mark> [<mark style="color:purple;">3-2. AcubeBatchCommandLineRunner.argumentsToMap()</mark>](springbatchargs.md#id-3-2.-map)
3. Map 데이터를 JobParameter 객체로 변환\
   \- [<mark style="color:purple;">3-3.  AcubeBatchCommandLineRunner.mapToJobParameters()</mark>](springbatchargs.md#id-3-3.-map-to-jobparametersbuilder)
4. JobLauncher를 실행하는 코드 생성\
   \- [<mark style="color:purple;">3-4. AcubeBatchCommandLineRunner.runJob()</mark>](springbatchargs.md#id-3-4.-job)
5. Job 코드 완성&#x20;

## 3. 코드

* Spring Batch: spring-boot-starter-batch:3.2.4
* Lombok: lombok:1.18.30
* MariaDB: mariadb-java-client:1.18.30

### 3-1.    ApplicationRunner 인터페이스 구현체

#### 3-1-1. HelloWorldApplication.java 파일에서 JobLauncher 실행 코드를 다음과 같이 제거 합니다.

{% code title="HelloWorldApplication.java" lineNumbers="true" %}
```java
@SpringBootApplication
public class HelloWorldApplication {

    public static void main(String[] args) {
      SpringApplication.run(HelloWorldApplication.class, args);
   }
}
```
{% endcode %}

#### 3-1-2. ApplicationRunner 를 상속 받아 run 메서드를 다음과 같이 작성한다.

{% code lineNumbers="true" %}
```java
@Order(0)
@Component
@Slf4j
public class AcubeBatchCommandLineRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        if (args.getSourceArgs().length == 0) {
            log.debug("Arguments 는 없습니다.");
            return;
        }
        for (String arg: args.getSourceArgs()) {
            log.debug(String.format("arg : %s", arg ));
        }
    }
}
```
{% endcode %}

* 1 Line:  CommandLineRunner 객체가 많은 경우 실행 우선 순위 지정&#x20;
* 2 Line:  스프링 빈으로 선언&#x20;
* 7 Line: Arguments 이 비어 있으면 로그 출력 후 리턴, 스프링 배치에서 기본으로 설정된 속성으로 진행 됨
* 11\~13 Line: Arguments 츨력

#### 3-1-3. 실행&#x20;

arguments:  --Job=myjob --key=1

<figure><img src="../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

### 3-2. Map로 변환 메서드 작성

다음 코드는 ApplicationRunner 인터페이스를  상속받아 구현한 AcubeBatchCommandLineRunner 객체에서 사용하는 메서드로 ApplicationArguments 를 받아서 Map으로 변경하는 코드로 다음과 같은 기능이 포함됩니다.

* &#x20;key=value 구조로 받은 데이터를 split 메서드를 사용해서 String 배열로 변환
* String 배열에서 "--" 문자포함 하고 있으면 제외
* &#x20;key=value 구조를 Map으로 변환&#x20;

{% code lineNumbers="true" %}
```java
private Map<String, Object> argumentsToMap(ApplicationArguments args) {
    Map<String, Object> argMap = new HashMap<>();
    for(String str: args.getSourceArgs()) {
        String[] keyValue = str.split("=");

        if (keyValue.length == 2) {
            String key =  keyValue[0];
            if (key.contains("--")) {
                // "--" 제외
                key = keyValue[0].substring(2);
            }

            String value = keyValue[1];
            argMap.put(key, value);
        }
    }
    return argMap;
}
```
{% endcode %}

run 메서드를 다음과 같이 변경 후 실행 합니다.

{% code lineNumbers="true" %}
```java
public void run(ApplicationArguments args) throws Exception {
    if (args.getSourceArgs().length == 0) {
        log.debug("Arguments 는 없습니다.");
        return;
    }
    Map<String, Object> argsMap = argumentsToMap( args);
    argsMap.forEach( (pa, cnt)-> log.debug(pa + " = " + (String) argsMap.get(pa))) ;
}
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 3-3. Map To JobParametersBuilder

Map 를 Argument로 받아서 JobParametersBuilder 객체로 변환 하는 코드로 요구사항 "베치 실행시 현재 일자가 자동 설정 되어야 한다."를 중족하기 위해서 시스템의 현재 일자를 얻어서 timestamp에 추가하는 코드 입니다.

{% code lineNumbers="true" %}
```java
private JobParametersBuilder mapToJobParameters(Map<String, String> argMap) {
    JobParametersBuilder jobParameters = new JobParametersBuilder();
    jobParameters.addLong("timestamp", System.currentTimeMillis());
    argMap.forEach((key, value)-> {
        jobParameters.addString(key,  value);
    });
    return jobParameters;
}
```
{% endcode %}

### 3-4. Job 실행 코드 작성

runJob 메서드는 스프링 배치의 Job를 실행 하기 위해 JobLauncher 메서드를 사용하여 JobExecution 객체를 생성하는 코드를 작성합니다.

* JobLauncher bean 을 ApplicationContext에서 가지고 오기 위해 ApplicationContext를 주입한다.
* getBean 메서드를 사용하여 JobLauncher, 실행하고자 하는 Job 객체를 생성한다.
* JobLauncher를 사용하여 Job을 실행 한다.

<pre class="language-java" data-line-numbers><code class="lang-java">@Order(0)
@Component
@RequiredArgsConstructor
@Slf4j
public class AcubeBatchCommandLineRunner implements ApplicationRunner {

     private final ApplicationContext applicationContext;

    public void run(ApplicationArguments args) throws Exception { ... }
    
<strong>    private Map&#x3C;String, String> argumentsToMap(ApplicationArguments args) { ... }
</strong> 
    private JobParametersBuilder mapToJobParameters(Map&#x3C;String, String> argMap) { ... }
        
    private String runJob(String jobName, 
                          JobParametersBuilder jobParametersBuilder) {
        JobExecution jobExecution;
        try {
            JobLauncher jobLauncher = applicationContext.getBean(JobLauncher.class);
            Job helloJob = applicationContext.getBean(jobName, Job.class);
            jobExecution = jobLauncher.run(helloJob, jobParametersBuilder.toJobParameters());

        } catch (Exception e) {
            new RuntimeException(e.getMessage()) ;
        }
        return  jobName + "이 수행 되었습니다." ;
    }
}
</code></pre>

* 7 Line: ApplicationContext 주입  (lombok의  @RequiredArgsConstructor 사용 )
* 19 Line: 스프링에서 관리하는 빈에서 JobLauncher 를 얻어와 객체 생성&#x20;
* 20 Line:  스프링에서 관리하는 빈에서 실행하고자 하는 Job 생성\
  이때 jobName은 실행을 원하는 Bean으로 @Bean으로 선언이 되어 있어야 한다.
* 21 Line: JobLauncher.run을 이용하여 job 실행&#x20;

### 3-4. Job 실행

{% code lineNumbers="true" %}
```java
@Override
public void run(ApplicationArguments args) throws Exception {
    if (args.getSourceArgs().length == 0) {
        log.debug("Arguments 는 없습니다.");
        return;
    }
    Map<String, String> argsMap = argumentsToMap(args);
    JobParametersBuilder mapToJobParameters =  mapToJobParameters(argsMap);
    runJob(argsMap.get("job"), mapToJobParameters);
}
```
{% endcode %}

* 7 Line:  ApplicationArguments 객체를 Map으로 변환
* 8 Line: Map을 JobParametersBuilder로 변환&#x20;
* 9 Line: JobLauncher.run호출 하기 위한 메서드 호출&#x20;

다음과 같이 로그를 확인 할 수 있습니다.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

myTasklet, myStep01, myStep02, myJob이 bean으로 등록 된 것을 확인 할 수 있으며 step에 정의된 Tasklet의 실행 로그를 확인 할 수 있습니다.

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 4.  전체 소스 코드



<details>

<summary>pom.xml</summary>

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
    <groupId>kr.co.abacus</groupId>
    <artifactId>base_springboot_bacth</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>base_springboot_bacth</name>
    <description>base_springboot_bacth</description>
    <properties>
        <java.version>21</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-batch</artifactId>
        </dependency>

<!--        <dependency>-->
<!--            <groupId>org.springframework.boot</groupId>-->
<!--            <artifactId>spring-boot-starter-data-jpa</artifactId>-->
<!--        </dependency>-->

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>3.0.3</version>
        </dependency>


        <dependency>
            <groupId>org.mariadb.jdbc</groupId>
            <artifactId>mariadb-java-client</artifactId>
            <version>3.3.3</version>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.30</version>
            <scope>provided</scope>
            <!--          <optional>true</optional>-->
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
            </plugin>
        </plugins>
    </build>

</project>
```

</details>

<details>

<summary>application.yml</summary>

```yaml
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
    <groupId>kr.co.abacus</groupId>
    <artifactId>base_springboot_bacth</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>base_springboot_bacth</name>
    <description>base_springboot_bacth</description>
    <properties>
        <java.version>21</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-batch</artifactId>
        </dependency>

<!--        <dependency>-->
<!--            <groupId>org.springframework.boot</groupId>-->
<!--            <artifactId>spring-boot-starter-data-jpa</artifactId>-->
<!--        </dependency>-->

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>3.0.3</version>
        </dependency>


        <dependency>
            <groupId>org.mariadb.jdbc</groupId>
            <artifactId>mariadb-java-client</artifactId>
            <version>3.3.3</version>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.30</version>
            <scope>provided</scope>
            <!--          <optional>true</optional>-->
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
            </plugin>
        </plugins>
    </build>

</project>
```

</details>

<details>

<summary>HelloWorldApplication.java</summary>

```java
@SpringBootApplication
public class HelloWorldApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(HelloWorldApplication.class, args);
    }

}
```

</details>

<details>

<summary>AcubeBatchCommandLineRunner.java</summary>

```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.context.ApplicationContext;
import org.springframework.core.annotation.Order;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import java.util.HashMap;
import java.util.Map;

@Order(0)
@Component
@RequiredArgsConstructor
@Slf4j
public class AcubeBatchCommandLineRunner implements ApplicationRunner {


    private final ApplicationContext applicationContext;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        if (args.getSourceArgs().length == 0) {
            log.debug("Arguments 는 없습니다.");
            return;
        }
        Map<String, String> argsMap = argumentsToMap( args);
        JobParametersBuilder mapToJobParameters =  mapToJobParameters(argsMap);
        runJob(argsMap.get("job"), mapToJobParameters);
    }


    private Map<String, String> argumentsToMap(ApplicationArguments args) {
        Map<String, String> argMap = new HashMap<>();
        for(String str: args.getSourceArgs()) {
            String[] keyValue = str.split("=");
            if (keyValue.length == 2) {
                String key =  keyValue[0];
                if (key.contains("--")) {
                    // Remove the leading "--"
                    key = keyValue[0].substring(2);
                }

                String value = keyValue[1];
                argMap.put(key, value);
            }
        }
        return argMap;
    }


    private JobParametersBuilder mapToJobParameters(Map<String, String> argMap) {
        JobParametersBuilder jobParameters = new JobParametersBuilder();
        jobParameters.addLong("timestamp", System.currentTimeMillis());
        argMap.forEach((key, value)-> {
            jobParameters.addString(key,  value);
        });
        return jobParameters;
    }

    private String runJob(String jobName,
                          JobParametersBuilder jobParametersBuilder) {
        JobExecution jobExecution;
        try {
            JobLauncher jobLauncher = applicationContext.getBean(JobLauncher.class);
            Job helloJob = applicationContext.getBean(jobName, Job.class);
            jobExecution = jobLauncher.run(helloJob, jobParametersBuilder.toJobParameters());

        } catch (Exception e) {
            new RuntimeException(e.getMessage()) ;
        }
        return  jobName + "이 수행 되었습니다." ;
    }

}
```

</details>

<details>

<summary>HelloWordlConfig.java</summary>

```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
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
public class HelloWorldConfig {

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

</details>

<details>

<summary>MyUserTasklet.java</summary>

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
        chunkContext.getStepContext().getJobParameters().forEach((key, value)->
                log.debug(key + "=" + value));
        return RepeatStatus.FINISHED;
    }
}
```

</details>
