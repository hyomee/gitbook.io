# 재시작설정

ErrorJob은 에러 테스트를 위한 Job으로 TaskletErrorStep, ChunkErrorStep 두개의 Step를 가지고 있다.

## 1. 정상 소스 및 결과&#x20;

<details>

<summary>정상 소스</summary>

```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class JobErrorConfig {

    private final JobUserListener jobUserListener;
    private final StepUserListenter stepUserListenter;

    @Bean
    public Step taskletErrorStep(JobRepository jobRepository,
                                 PlatformTransactionManager transactionManager ) {
        return new StepBuilder("TASKLET_STEP_00", jobRepository)
                .tasklet((contribution, chunkContext)-> {
                    log.debug("TASKLET_STEP_00 :: 실행....");
                    return RepeatStatus.FINISHED;
                 }, transactionManager)
                .build();

    }

    @Bean
    public Step chunkErrorStep( JobRepository jobRepository,
                       PlatformTransactionManager transactionManager) {
        return new StepBuilder("CHUNK_STEP_00", jobRepository )
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
                        log.debug("CHUNK_STEP_00 :: 실행....");
                        return "변환 process : " + item;
                    }
                })
                .writer(new ItemWriter<String>() {
                    @Override
                    public void write(Chunk<? extends String> items) throws Exception {
                        log.debug("CHUNK_STEP_00 :: Writer 실행 ....");
                        items.forEach(item -> log.info(item));
                    }
                })
                .listener(stepUserListenter)
                .build();
    }

    @Bean
    public Job errorJob(JobRepository jobRepository,
                        Step taskletErrorStep,
                        Step chunkErrorStep ) {
        return new JobBuilder("JOB_ERR_TEST", jobRepository)
                .start(taskletErrorStep)
                .next(chunkErrorStep)
                .listener(jobUserListener)
                .build();
    }
}
```

</details>

* **정상 수행 결과**

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>수행 쿼리</summary>

<pre class="language-sql"><code class="lang-sql"><strong>SELECT JOB.JOB_INSTANCE_ID,
</strong>       JOB.JOB_NAME,
       EXE.JOB_EXECUTION_ID AS JOB_ID,
       EXE.STATUS,
       EXE.EXIT_CODE,
       SETP.STEP_EXECUTION_ID AS STEP_ID,
       SETP.STEP_NAME,
       SETP.STATUS,
       SETP.EXIT_CODE
  FROM BATCH_JOB_INSTANCE JOB
  INNER JOIN BATCH_JOB_EXECUTION EXE
        ON JOB.JOB_INSTANCE_ID = EXE.JOB_INSTANCE_ID
  INNER JOIN BATCH_STEP_EXECUTION SETP
        ON EXE.JOB_EXECUTION_ID = SETP.JOB_EXECUTION_ID
</code></pre>

</details>

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 2.  오류   수정 소스및 결과

&#x20;Job으로 에러는 다음과 같이 강제 오류가 발생 한다,

* TaskletErrorStep: 강제 오류 발생&#x20;
* ChunkErrorStep: ItemWriter에서 강제 오류 발생

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>Job Configuration: Job 등록 ( 오류 수정 코드 :  강제 오류 발생 )</summary>

```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class JobErrorConfig {

    private final JobUserListener jobUserListener;
    private final StepUserListenter stepUserListenter;

    @Bean
    public Step taskletErrorStep(JobRepository jobRepository,
                                 PlatformTransactionManager transactionManager ) {
        return new StepBuilder("TASKLET_STEP_00", jobRepository)
                .tasklet((contribution, chunkContext)-> {
                    log.debug("TASKLET_STEP_00 :: 실행....");
                    if (true) {
                        throw new RuntimeException("Tasklet_Error_Step .... Error....");
                    }
                    return RepeatStatus.FINISHED;
                 }, transactionManager)
                .build();

    }

    @Bean
    public Step chunkErrorStep( JobRepository jobRepository,
                       PlatformTransactionManager transactionManager) {
        return new StepBuilder("CHUNK_STEP_00", jobRepository )
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
                        log.debug("CHUNK_STEP_00 :: 실행....");
                        return "변환 process : " + item;
                    }
                })
                .writer(new ItemWriter<String>() {
                    @Override
                    public void write(Chunk<? extends String> items) throws Exception {
                        log.debug("CHUNK_STEP_00 :: Writer 실행 ....");
                        items.forEach(item -> {
                            if ("item3".equals(item)) {
                                throw new RuntimeException("CHUNK_STEP_00 :: ERROR :: item ::" + item);
                            }
                            log.info(item);
                        });
                    }
                })
                .listener(stepUserListenter)
                .build();
    }

    @Bean
    public Job errorJob(JobRepository jobRepository,
                        Step taskletErrorStep,
                        Step chunkErrorStep ) {
        return new JobBuilder("JOB_ERR_TEST", jobRepository)
                .start(taskletErrorStep)
                .next(chunkErrorStep)
                .listener(jobUserListener)
                .build();
    }
}

```

</details>

* 아래 코드는 강제 오류를 발생하기 위해 수정된 부분의 코드 일부이다.

```java
.tasklet((contribution, chunkContext)-> {
    log.debug("TASKLET_STEP_00 :: 실행....");
    if (true) {
        throw new RuntimeException("Tasklet_Error_Step .... Error....");
    }
    return RepeatStatus.FINISHED;
 }, transactionManager)
.build();

 

.writer(new ItemWriter<String>() {
    @Override
    public void write(Chunk<? extends String> items) throws Exception {
        log.debug("CHUNK_STEP_00 :: Writer 실행 ....");
        items.forEach(item -> {
            if ("item3".equals(item)) {
                throw new RuntimeException("CHUNK_STEP_00 :: ERROR :: item ::" + item);
            }
            log.info(item);
        });
    }
})
```

*   실행 결과: TaskletErrorJob에서 오류가 발생 하여 Job이 종료된다.\


    <figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

    <figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 3.  Job - preventRestart

기본적으로 스프링 배치는  같은 Job 파라미터를 가진 작업이 성공 하였으면 재시작 하여도 실행이 되지 않는다.  이 기능을 제어하기 위해 제공된 속성으로 기본값은 선언 하지 않았을 때 이고 true이다. false로 설정 하기 위해서는 `preventRestart를 설정하면 된다.`

### 3-1. 기본값 (preventRestart:true)

정상 종료이면 다시 시작을 해도 실행이 되지 않지만 오류 실행시 다시 시작하면 실행이 된다.

```java
@Bean
public Job errorJob(JobRepository jobRepository,
                    Step taskletErrorStep,
                    Step chunkErrorStep ) {
    return new JobBuilder("JOB_ERR_RESTART_TRUE", jobRepository)
            .start(taskletErrorStep)
            .next(chunkErrorStep)
            .listener(jobUserListener)
            .build();
}
```

*   **실행 시 실패**\
    (

    <figure><img src="../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

    <figure><img src="../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>
*   **재 실행 - 다시 시작 - 실패로 실행됨**\


    <figure><img src="../../.gitbook/assets/image (227).png" alt=""><figcaption></figcaption></figure>



    <figure><img src="../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

### 3-2.   preventRestart 지정(preventRestart:false)

preventRestart를 선언하면 false값으로 되고 오류가 나도 다시 시작시 실행이 되지 않는다

```java
@Bean
public Job errorJob(JobRepository jobRepository,
                    Step taskletErrorStep,
                    Step chunkErrorStep ) {
    return new JobBuilder("JOB_ERR_RESTART_FALSE", jobRepository)
            .start(taskletErrorStep)
            .next(chunkErrorStep)
            .listener(jobUserListener)
            .preventRestart()
            .build();
}
```

*   **실행 시 실패**\


    <figure><img src="../../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

    <figure><img src="../../.gitbook/assets/image (230).png" alt=""><figcaption></figcaption></figure>
*   **다시   작 시 실헹되지 않음**\


    <figure><img src="../../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>

## 4.  Step

### 4-1. **Start Limit**&#x20;

기본값은 `Integer.MAX_VALUE`로, 무제한으로 실행되는 것으로 특정 스텝이 한 번만 실행되도록 설정하거나, 특정 리소스를 한 번만 처리해야 하는 경우에 사용할 때 사용된다.

```java
public Step taskletErrorStep(JobRepository jobRepository,
                             PlatformTransactionManager transactionManager ) {
    return new StepBuilder("TASKLET_STEP_004", jobRepository)
            .tasklet((contribution, chunkContext)-> {
                log.debug("TASKLET_STEP_00 :: 실행....");
                if (true) {
                    throw new RuntimeException("Tasklet_Error_Step .... Error....");
                }
                return RepeatStatus.FINISHED;
             }, transactionManager)
            .startLimit(2)
            .build();

}
```

<figure><img src="../../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>

2번 실패 후 다시 수행 하면 다음과 같은 오류가 난다.

<figure><img src="../../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

### 4-2. **allow-start-if-complete**&#x20;

기본적으로 재시작된 작업에서는 COMPLETED 상태인 스텝은 건너뛰게 되어있는데 `true`로 변경하면 항상 실행되도록 할 수 있다.

```java
public Step taskletErrorStep(JobRepository jobRepository,
                                 PlatformTransactionManager transactionManager ) {
        return new StepBuilder("TASKLET_STEP_005", jobRepository)
                .tasklet((contribution, chunkContext)-> {
                    log.debug("TASKLET_STEP_00 :: 실행....");
                    return RepeatStatus.FINISHED;
                 }, transactionManager)
                .startLimit(2)
                .allowStartIfComplete(true)
                .build();

    }
```

<figure><img src="../../.gitbook/assets/image (234).png" alt=""><figcaption></figcaption></figure>

성공을 하여도 다시 실행하는 기능으로 2번 실행이 되고 startLimit(2)에 의해 다시 실행하면 실행이 되지 않는다.

