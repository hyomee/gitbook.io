# Skip/Retry



## 1. Skip

* Skip은 특정 예외가 발생했을 때 해당 데이터 처리를 건너뛰는 기능이다
* `ItemReader`, `ItemProcessor`, `ItemWriter`에 적용 가능하다
* `faultTolerant()`를 호출하여 skip 기능을 활성화한다.
* `skipLimit()`과 `skip()` 메서드를 사용하여 건너뛸 예외와 최대 건너뛸 횟수를 설정한다.

{% code lineNumbers="true" %}
```java
@Bean
public Step chunkErrorStep( JobRepository jobRepository,
                   PlatformTransactionManager transactionManager) {
    return new StepBuilder("CHUNK_STEP_SKIP_TEST_015", jobRepository )
            .<String, String>chunk(2,transactionManager )
            .reader( new ListItemReader<>(Arrays.asList("item1",
                    "item2",
                    "item3",
                    "item4",
                    "item5",
                    "item6")))
            .processor(new ItemProcessor<String, String>() {
                @Override
                public String process(String item) throws Exception {
                    log.debug("CHUNK_STEP_00 :: process ...." + item);
                    return "변환 process : " + item;
                }
            })
            .writer(new ItemWriter<String>() {
                @Override
                public void write(Chunk<? extends String> items) throws Exception {
                    items.forEach(item -> {
                        log.debug("CHUNK_STEP_00 :: Writer  .... || "+ item);
                        if ("변환 process : item2".equals(item)) {
                            log.debug("CHUNK_STEP_00 :: RuntimeException  .... || "+ item);
                            throw new RuntimeException("CHUNK_STEP_00 :: ERROR :: item ::" + item);
                        }
                        if ("변환 process : item3".equals(item)) {
                            log.debug("CHUNK_STEP_00 :: RuntimeException  .... || "+ item);
                            throw new RuntimeException("CHUNK_STEP_00 :: ERROR :: item ::" + item);
                        }
                    });
                }
            })
            .listener(stepUserListenter)
            .faultTolerant()
            .skipLimit(2)
            .skip(RuntimeException.class)
            .build();
}
```
{% endcode %}

* 24 \~ 31 Line: 강제 오류 발생 코드
* 36 Line: `faultTolerant()`를 호출하여 skip 기능을 활성화
* 37 Line: 에러 Skip 2회 설정&#x20;
* 38 Line: Skip 에러 설정
*   결과: 6개 item을  Chunk 2로 6개를 읽는 동안 2회 에러가 발생하여 모두 스킵되어 작업은 성공적으로 종료 한다. 몰론 에러 2개는 처리 되지는 않는다.\


    <figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><br></p></figcaption></figure>

    <figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 1-1. Skip 을 적제 설정 하면&#x20;

동일 소스에서 에러 SKIP를 적제 설정하면 Job은 실패로 된다.

```java
.faultTolerant()
.skipLimit(1)
.skip(RuntimeException.class)
```

```log

Caused by: java.lang.RuntimeException: 
     CHUNK_STEP_00 :: ERROR :: item ::변환 process : item3 
     at kr.co.abacus.batch.error.JobErrorConfig$1.lambda$write$0(JobErrorConfig.java:76) at java.base/java.lang.Iterable.forEach(Iterable.java:75) at kr.co.abacus.batch.error.JobErrorConfig$1.write(JobErrorConfig.java:68) at org.springframework.batch.core.step.item.SimpleChunkProcessor.writeItems(SimpleChunkProcessor.java:203) at 

     org.springframework.batch.core.step.item.FaultTolerantChunkProcessor.scan(FaultTolerantChunkProcessor.java:574) ... 51 common frames omitted 

2024-05-06 20:17:59,791 INFO [kr.co.abacus.batch.error.StepUserListenter] 
     STEP FINISHED !! 2024-05-06 20:17:59,791 ERROR [kr.co.abacus.batch.error.StepUserListenter] 에러코드 : FAILED , 
     에러메세지 : org.springframework.batch.core.step.skip.SkipLimitExceededException: 
     Skip limit of '1' exceeded at org.springframework.batch.core.step.skip.LimitCheckingItemSkipPolicy.shouldSkip(LimitCheckingItemSkipPolicy.java:125) at ....

2024-05-06 20:17:59,868 INFO [org.springframework.batch.core.step.AbstractStep] 
     Step: [CHUNK_STEP_SKIP_TEST_016] executed in 659ms 2024-05-06 20:18:00,048 INFO [kr.co.abacus.batch.error.JobUserListener] 
     JOB ERROR: FAILED 

2024-05-06 20:18:00,160 INFO [org.springframework.batch.core.launch.support.SimpleJobLauncher] Job: [SimpleJob: [name=JOB_ERR_SKIP_TEST_016]] completed with the following parameters: [{'job':'{value=errorJob, type=class java.lang.String, identifying=true}','file':'{value=D:/ABACUS_PRJ/abacus/acube-svc-batch/file/in/TB_DEPLOY.csv, type=class java.lang.String, identifying=true}','outfile':'{value=D:/ABACUS_PRJ/abacus/acube-svc-batch/file/out/TB_DEPLOY_BEAN.csv, type=class java.lang.String, identifying=true}'}] and the following status: [FAILED] in 1s966ms 

2024-05-06 20:18:00,169 INFO [org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean] Closing JPA EntityManagerFactory for persistence unit 'mysql' 
2024-05-06 20:18:00,172 INFO [com.zaxxer.hikari.HikariDataSource] pool-acfw-jpa - Shutdown initiated... 2024-05-06 20:18:00,331 INFO [com.zaxxer.hikari.HikariDataSource] pool-acfw-jpa - Shutdown completed. 
2024-05-06 20:18:00,336 INFO [kr.co.abacus.batch.AcubeBatchCommandLineRunner] BatchMagDTO(isRun=true, msg=errorJob이 수행 되었습니다., summary=StepExecution: id=307, version=3, name=TASKLET_STEP_016, status=COMPLETED, exitStatus=COMPLETED, readCount=0, filterCount=0, writeCount=0 readSkipCount=0, writeSkipCount=0, processSkipCount=0, commitCount=1, rollbackCount=0 StepExecution: id=308, version=4, name=CHUNK_STEP_SKIP_TEST_016, status=FAILED, exitStatus=FAILED, readCount=4, filterCount=0, writeCount=1 readSkipCount=0, writeSkipCount=1, processSkipCount=0, commitCount=2, rollbackCount=4 )
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

동일 소스에 에러 스킵을 3으로 올리면 Job과 Step이 모두 정상 처리 된다.\
Tasklet가실행이 되지 않는 이유는 이전 실행에서 정상 실행 되어서 이다.

```
.faultTolerant()
.skipLimit(3)
.skip(RuntimeException.class)
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 1-2. skipPolicy

특정 에러가 발생 하였을 때 Skip를 하기 위해 에러 정책을 설정 하여 skip하는 경우 사용 한다.

```java
.faultTolerant() 
.skipPolicy(new CustomSkipPolicy())
```

{% code title="CustomSkipPolicy.java" lineNumbers="true" %}
```java
@Slf4j
public class CustomSkipPolicy implements SkipPolicy {
    @Override
    public boolean shouldSkip(Throwable throwable, long skipCount) throws SkipLimitExceededException {
        if (throwable instanceof CustomItem3Exception)  {
            log.debug("CustomItem3Exception :: Skipping " + skipCount  );
            return true;
        }

        if (throwable instanceof CustomItemException ex  ) {
            log.debug("CustomItemException :: Skipping " + skipCount  );
            return ex.getItem().equals("item2");
        }

        return false;
    }
}
```
{% endcode %}

* 사용자 정의 Exception

```java
public class CustomItem3Exception  extends RuntimeException {

    private String item;

    public CustomItem3Exception(String item){
        this.item = item;
    }

    public String getItem() {
        return item;
    }
}


public class CustomItemException  extends RuntimeException {
 
}

```

<figure><img src="../../.gitbook/assets/image (235).png" alt=""><figcaption></figcaption></figure>

## 2. Retry

* **Retry 기능**은 배치 작업이 실패했을 때 지정된 횟수만큼 재시도하는 기능이다.
* `ItemProcessor`와 `ItemWriter`에 적용 가능합니다.
* 예외가 발생하면 지정된 횟수만큼 재시도하고, 재시도 대상 예외를 지정할 수 있습니다.
* Retry 기능을 구성하려면 `RetryTemplate`을 사용하거나, Spring Batch에서 제공하는 `retry()`메서드를 활용할 수 있다.

```java
.faultTolerant()
.retryLimit(1) //retry 횟수, retry 사용시 필수 설정, 해당 Retry 이후 Exception시 Fail 처리
.retry(SQLException.class) // SQLException에 대해선 Retry 수행
.noRetry(NullPointerException.class) // NullPointerException에 no Retry
//.retryPolicy(new CustomRetryPolicy().retryPolicy()) // 사용자가 커스텀하며 Retry Policy 설정 가능
```



## 3. noRollback

* `StepBuilder`의 `noRollback()` 메서드를 호출하여 롤백을 일으키지 않을 예외를 지정할 수 있다.
* 예를 들어, 아래 코드에서 `ValidationException`이 발생하면 롤백을 일으키지 않도록 설정했습니다:

```java
.faultTolerant()
.noRollback(ValidationException.class)
.noRollback(NullPointerException.class) // NullPointerException 발생  rollback이 되지 않게 설정
.build();
```
