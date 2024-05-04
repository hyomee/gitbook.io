# Step Listener

Step에서 발생하는 에러(예외)를Listener를 등록하여 예외에 대한 처리를 하기 위해서 StepExecutionListener에 대한 구현체를 작성해야 한다.

* **StepExecutionListener**: 배치 스텝(Step)이 실행되기 전과 후에 어떤 작업을 수행할지 정의하는 것으로 로그 기록, 실행 시간 측정, 상태 정보 조회 등에 활용된다.
*   **StepExecution:** 배치 스텝(Step)의 실행 상태를 나타내는 객체로  배치 스텝이 실행되는 동안의 정보를 포함하며, 스텝의 성공 여부, 시작 시간, 종료 시간, 상태 등을 가지고 있다.\


    <table><thead><tr><th width="204">속성</th><th>설명</th></tr></thead><tbody><tr><td><strong>Step</strong></td><td>StepExecution이 속한 배치 스텝(Step)을 나타내며. 하나의 Step은 동일한 Job 내에서 여러 StepExecution을 가질 수 있다.</td></tr><tr><td><strong>JobExecution</strong></td><td>StepExecution이 속한 JobExecution을 나타낸다.</td></tr><tr><td><strong>StepName</strong></td><td>Step의 이름</td></tr><tr><td><strong>ExitStatus</strong></td><td>배치 작업이 완료된 후의 상태를 나타내며  성공적으로 완료되었는지, 실패했는지 등을 확인할 수 있다.</td></tr><tr><td><strong>ExecutionContext</strong></td><td>StepExecution이 실행되는 동안 사용자 정의 데이터를 저장하는 데 사용되며 StepExecution간에 데이터를 공유할 때 유용하다.</td></tr></tbody></table>
* Step 실행에 따른 상태를 이용하여 오류(예외) 처리를 하기 위해 Step Listener를 등록하고 Step Execution 객체의 Status속성, ExitStatus속성을 사용하여 오류에 대한 후 처리 작업을 한다.

<figure><img src="../../../.gitbook/assets/image.png" alt="" width="563"><figcaption></figcaption></figure>

## 1. **Step Listener**

* sS@Component

```java
@Slf4j
public class StepUserListenter {

    @BeforeStep
    public void beforeStep(final StepExecution stepExecution) {
        log.info("Step execution started");
    }

    @AfterStep
    public void afterStep(final StepExecution stepExecution) {
        log.info("Step execution finished");
        BatchStatus batchStatus = stepExecution.getStatus();

        if (BatchStatus.FAILED.equals(batchStatus)) {
            // 여기에 에처 처리 
            log.error(String.format("에러코드 : %s , 에러메세지 : %s ",
                    stepExecution.getExitStatus().getExitCode(),
                    stepExecution.getExitStatus().getExitDescription() ));
        }


    }
}
```

<details>

<summary>Step Listener 구현시 StepExecutionListener의 구현체로 작성 기 코드</summary>

```java
public class StepUserListener implements StepExecutionListener {

    @Override
    public void beforeStep(StepExecution stepExecution) {
        // 여기에 작성
    }
    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        // 여기에 작성
        return ExitStatus.COMPLETED;
    }
}
```

</details>

```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class JobErrorConfig {

      private final StepUserListenter stepUserListenter;
    

    @Bean
    public Step chunkErrorStep( JobRepository jobRepository,
                       PlatformTransactionManager transactionManager) {
        return new StepBuilder("Chunk_Step_Error", jobRepository )
                .<String, String>chunk(2,transactionManager )
                .reader(new ListItemReader<>(Arrays.asList("item1", "item2", "item3", "item4", "item5", "item6")))
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
                .listener(stepUserListenter)
                .build();
    }

    @Bean
    public Job errorJob(JobRepository jobRepository,
                        Step chunkErrorStep ) {
        return new JobBuilder("Job-Errorr", jobRepository)
                .start(chunkErrorStep)
                .listener(jobUserListener)
                .build();
    }
}
```
