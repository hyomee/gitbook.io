# Job Listener

**Job Listener**는 배치 작업(Job)이 실행되기 전과 후에 어떤 작업을 수행하기 위해 사용하는 것으로 배치 작업의 상태를 확인을 통해 할 수 있는데 이 기능을 이용하여 Job에 대한 오류를 확인하여 처리 할 수 있다.

* &#x20;**JobExecution 객체**:    배치 작업(Job)의 실행 상태를 나타내는 객체입니다. 이 객체는 특정 배치 작업이 실행되는 동안의 정보를 포함하며, 작업의 성공 여부, 시작 시간, 종료 시간, 상태 등을 추적하며 다음과 같은 속성을 가지고 있다.

<table><thead><tr><th width="204">속성</th><th>설명</th></tr></thead><tbody><tr><td><strong>JobInstance</strong></td><td>JobExecution이 속한 Job의 인스턴스를 나타내며 하나의 JobInstance는 동일한 Job을 실행하는 여러 JobExecution을 가질 수 있다.</td></tr><tr><td><strong>JobParameters</strong></td><td>Job 실행에 필요한 매개변수를 나타낸다.</td></tr><tr><td><strong>JobStatus</strong></td><td>JobExecution의 상태를 나타내며 COMPLETED, STARTING, STARTED, STOPPING, STOPPED, FAILED, ABANDONED, UNKNOWN가 있다.</td></tr><tr><td><strong>ExitStatus</strong></td><td>배치 작업이 완료된 후의 상태를 나타내며  성공적으로 완료되었는지, 실패했는지 등을 확인할 수 있다.</td></tr><tr><td><strong>ExecutionContext</strong></td><td>JobExecution이 실행되는 동안 사용자 정의 데이터를 저장하는 데 사용되며 JobExecution 간에 데이터를 공유할 때 유용하다.</td></tr></tbody></table>

* Job 실행에 따른 상태를 이용하여 오류(예외) 처리를 하기 위해 Job Listener를 등록하고 JobExecution 객체의 JobStatus속성을 사용하여 오류에 대한 후 처리 작업을 한다.

<figure><img src="../../.gitbook/assets/image (221).png" alt="" width="563"><figcaption></figcaption></figure>

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

## 2. Tasklet Step

Job 생성시 Tasklet Step을 시작하고 Listener을 등록 한다.

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

* 14 \~ 16 Libe:  강제로 에러 발생&#x20;
* 24 \~ 30 Line: tasklerErrorJob() 메서드는 JobBuilder를 통해서 job을 등록한 것으로 listener로 TaskletJobUserListener를 등록 하여 Job의 실행 상태를 관리할 수 있다.
*   **결과** : \


    <figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

## 3. Chunk Step

Job 생성시 Chunk Step을 시작하고 Listener을 등록 한다.

{% code lineNumbers="true" %}
```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class JobErrorConfig {

    private final JobUserListener jobUserListener;
    

    @Bean
    public Step chunkErrorStep( JobRepository jobRepository,
                       PlatformTransactionManager transactionManager) {
        return new StepBuilder("Chunk_Step_Error", jobRepository )
                .<String, String>chunk(2,transactionManager )
                .reader(new ListItemReader<>(Arrays.asList("item1", 
                    "item2", 
                    "item3", 
                    "item4", 
                    "item5", 
                    "item6")))
                .processor(new ItemProcessor<String, String>() {
                    @Override
                    public String process(String item) throws Exception {
                        return "변환 process : " + item;
                    }
                })
                .writer(new ItemWriter<String>() {
                    @Override
                    public void write(Chunk<? extends String> items) throws Exception {
                        if (true) {
                            throw new RuntimeException("Chunk_Step_Error .... Error....");
                        }
                        items.forEach(item -> log.info(item));
                    }
                })
                .build();
    }

    @Bean
    public Job errorJob(JobRepository jobRepository,
                        Step chunkErrorStep ) {
        return new JobBuilder("Job-Error", jobRepository)
                .start(chunkErrorStep)
                .listener(jobUserListener)
                .build();
    }
}
```
{% endcode %}

* 29 \~ 31 Libe:  강제로 에러 발생
* 41 \~ 44 Line: errorJob() 메서드는 JobBuilder를 통해서 job을 등록한 것으로 listener로 TaskletJobUserListener를 등록 하여 Job의 실행 상태를 관리할 수 있다.
*   결과:



    <figure><img src="../../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>
