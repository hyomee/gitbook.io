# ExecutionContext

배치 작업을 실행하는 동안 필요한 데이터를 지속 가능한 상태로 저장할 수 있도록 Key/Value 데이터 컨테이너이다. 즉 **상태 정보를 저장**하는 데 사용된다.

* **Job ExecutionContext**:
  * **Job** 수준에서 사용됩니다.
  * Job 전체에 걸쳐 유지되며, 각 Step이 종료될 때 업데이트됩니다.
  * Job 상태 정보를 저장하고, Job 내에서 공유되는 데이터를 보관합니다.
  * Job ExecutionContext에 데이터를 저장하면 해당 Job의 모든 Step에서 접근 가능합니다
* **Step ExecutionContext**:
  * **Step** 수준에서 사용됩니다.
  * 각 Step에서만 유지되며, 각 청크가 커밋될 때 업데이트됩니다.
  * Step 상태 정보를 저장하고, Step 내에서 공유되는 데이터를 보관합니다.
  * 각 Step의 실행 도중에만 접근 가능하며, 다른 Step에서는 참조할 수 없습니다.

즉,  Job ExecutionContext에 데이터를 저장하면 해당 Job의 모든 Step에서 접근 가능하며, Step 간의 강한 결합을 피할 수 있다.



## 1.  Tasklet  예제

```java
@Component
public class DataSharingFirstTasklet implements Tasklet {
    @Override
    public RepeatStatus execute(StepContribution contribution,
                                ChunkContext chunkContext) throws Exception {

        ExecutionContext stepExecutionContext = getSetpExecutionContext(chunkContext);
        ExecutionContext jobExecutionContext = getJobExecutionContext(chunkContext);

        stepExecutionContext.put("FIRST_STEP_EXECUTION_CONTEXT", "DataSharingFirstTasklet Step Value1");
        jobExecutionContext.put("FIRST_JOB_EXECUTION_CONTEXT", "DataSharingFirstTasklet Job Value1");

        return RepeatStatus.FINISHED;
    }


    private ExecutionContext getSetpExecutionContext(ChunkContext chunkContext) {
        return chunkContext.getStepContext().getStepExecution().getExecutionContext();
    }

    private ExecutionContext getJobExecutionContext(ChunkContext chunkContext) {
        return chunkContext.getStepContext().getStepExecution().getJobExecution().getExecutionContext();
    }
}
```

```java
@Component
@Slf4j
public class DataSharingSecondTasklet implements Tasklet {
    @Override
    public RepeatStatus execute(StepContribution contribution,
                                ChunkContext chunkContext) throws Exception {

        ExecutionContext stepExecutionContext = getSetpExecutionContext(chunkContext);
        ExecutionContext jobExecutionContext = getJobExecutionContext(chunkContext);

        String FIRST_STEP_EXECUTION_CONTEXT = (String) stepExecutionContext.get("FIRST_STEP_EXECUTION_CONTEXT");
        String FIRST_JOB_EXECUTION_CONTEXT = (String) jobExecutionContext.get("FIRST_JOB_EXECUTION_CONTEXT");

        log.debug("FIRST_STEP_EXECUTION_CONTEXT :: " + FIRST_STEP_EXECUTION_CONTEXT);
        log.debug("FIRST_JOB_EXECUTION_CONTEXT :: " + FIRST_JOB_EXECUTION_CONTEXT);
        return RepeatStatus.FINISHED;
    }


    private ExecutionContext getSetpExecutionContext(ChunkContext chunkContext) {
        return chunkContext.getStepContext().getStepExecution().getExecutionContext();
    }

    private ExecutionContext getJobExecutionContext(ChunkContext chunkContext) {
        return chunkContext.getStepContext().getStepExecution().getJobExecution().getExecutionContext();
    }
}
```

## 2.  Chunk예제

```java
@Component
@Slf4j
public class StepItemReader implements ItemReader<TbDeployVO>, StepExecutionListener {

    private OpenCsvFileUtils openCsvFileUtils;


    @Override
    public void beforeStep(StepExecution stepExecution) {
        String file = stepExecution.getJobExecution().getJobParameters().getString("file");
        openCsvFileUtils = new OpenCsvFileUtils(file);
        stepExecution.getExecutionContext().put("READER_STEP", "StepItemReader beforeStep");
    }
    
    @Override
    public TbDeployVO read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
       ......
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        openCsvFileUtils.closeReader();
        log.debug("StepItemReader :: READER_STEP :: " + stepExecution.getExecutionContext().get("READER_STEP"));
        return ExitStatus.COMPLETED;
    }
}
```

```java
@Component
@Slf4j
public class StepItemWriter implements ItemWriter<TbDeployWriteVO>, StepExecutionListener {

    private OpenCsvFileUtils openCsvFileUtils;
    @Override
    public void beforeStep(StepExecution stepExecution) {
        String file = stepExecution.getJobExecution().getJobParameters().getString("outfile");
        openCsvFileUtils = new OpenCsvFileUtils(file);
        stepExecution.getExecutionContext().put("WRITER_STEP", "StepItemWriter beforeStep");
    }

    @Override
    public void write(Chunk<? extends TbDeployWriteVO> tbDeployWriteVOs) throws Exception {
     .....

    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        openCsvFileUtils.closeWriter();
        log.debug("StepItemWriter :: READER_STEP :: " + stepExecution.getExecutionContext().get("READER_STEP"));
        log.debug("StepItemWriter :: PROCESSOR_STEP :: " + stepExecution.getExecutionContext().get("PROCESSOR_STEP"));
        log.debug("StepItemWriter :: WRITER_STEP :: " + stepExecution.getExecutionContext().get("WRITER_STEP"));


        log.debug("StepItemWriter :: FIRST_JOB_EXECUTION_CONTEXT :: " + stepExecution.getJobExecution().getExecutionContext().get("FIRST_JOB_EXECUTION_CONTEXT"));
        return ExitStatus.COMPLETED;
    }

}
```

## 3.  Job Config

```
@Configuration
@Slf4j
public class DataSharingConfig {

    @Bean
    public Job dataSharingJob(JobRepository jobRepository,
                              Step firstStep,
                              Step secondStep,
                              Step case01Step) {
        log.debug("####->  dataSharingJob!");
        return new JobBuilder("dataSharingJob", jobRepository)
                .incrementer(new RunIdIncrementer())
                .start(firstStep)
                .next(secondStep)
                .next(case01Step)
                .build();
    }

    @Bean
    public Step firstStep(JobRepository jobRepository,
                          PlatformTransactionManager transactionManager,
                          DataSharingFirstTasklet dataSharingFirstTasklet) {
        return new StepBuilder("firstStep", jobRepository)
                .tasklet(dataSharingFirstTasklet, transactionManager)
                .build();
    }



    @Bean
    public Step secondStep(JobRepository jobRepository,
                           PlatformTransactionManager transactionManager,
                           DataSharingSecondTasklet dataSharingSecondTasklet) {
        return new StepBuilder("secondStep", jobRepository)
                .tasklet(dataSharingSecondTasklet, transactionManager)
                .build();
    }


}
```

## 4.  결과

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>



