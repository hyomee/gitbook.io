# Job Listener 오류처리

###

## 1. Job Listener

&#x20;Job 실헹에 대한 상태를 확인하기 위해 다음과 같은 코드를 작성 한다.

{% code lineNumbers="true" %}
```java
@Component
@Slf4j
public class TaskletJobUserListener {
    @AfterJob
    public void afterJob(JobExecution jobExecution) {

        if (jobExecution.getStatus() == BatchStatus.COMPLETED) {
            log.info("JOB FINISHED !!");
        } else if (BatchStatus.FAILED.equals(jobExecution.getStatus())  ) {
            // 여기에서 다른 작업 처리를 한다.
            log.info("JOB ERROR: " + jobExecution.getExitStatus().getExitDescription());
        }
    }
}

// JobExecutionListener 상속을 통한 구현 
@Component
@Slf4j
public class TaskletJobUserListener implements JobExecutionListener {
    public void afterJob(JobExecution jobExecution) {
        if (jobExecution.getStatus() == BatchStatus.COMPLETED) {
            log.info("JOB FINISHED !!");
        } else if (BatchStatus.FAILED.equals(jobExecution.getStatus())  ) {
            log.info("JOB ERROR: " + jobExecution.getExitStatus().getExitDescription());
        }
    }
}
```
{% endcode %}

* 8 - 11 Line: Job 실행 상태에 따라서 후 처리를 할 수 있다.

## 2. Job 설정

&#x20;아래 소스는 Job 설정 소스 입니다.

{% code lineNumbers="true" %}
```java
@Component
@RequiredArgsConstructor
@Slf4j
public class TasklerErrorConfig {

    private final TaskletJobUserListener taskletJobUserListener;

    @Bean
    public Step taskletErrorStep(JobRepository jobRepository,
                                 PlatformTransactionManager transactionManager ) {
        return new StepBuilder("Tasklet_Error_Step", jobRepository)
                .tasklet((contribution, chunkContext)-> {
                    log.debug("Tasklet_Error_Step .... ");
                    if (true) {
                        throw new RuntimeException("Tasklet_Error_Step .... Error....");
                    }
                    return RepeatStatus.FINISHED;
                 }, transactionManager)
                .build();

    }

    @Bean
    public Job tasklerErrorJob(JobRepository jobRepository,
                               Step taskletErrorStep ) {
        return new JobBuilder("Tasklet-Error", jobRepository)
                .start(taskletErrorStep)
                .listener(taskletJobUserListener)
                .build();
    }
}

```
{% endcode %}

* 24 \~ 30 Line: tasklerErrorJob() 메서드는 JobBuilder를 통해서 job을 등록한 것으로 listener로 TaskletJobUserListener를 등록 하여 Job의 실행 상태를 관리할 수 있다.

## 3. 결과 :&#x20;

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

