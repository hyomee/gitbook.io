# 자원공유(ExecutionContext)

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

Job ExecutionContext, Step ExecutionContext의 범위를 알아보기 위해 Tasklet의 execute() 메서드에 사용자 정의 값을 설정한다.

<figure><img src="../.gitbook/assets/image (258).png" alt=""><figcaption></figcaption></figure>

{% code lineNumbers="true" %}
```java
@Component
public class DataSharingFirstTasklet implements Tasklet {
    @Override
    public RepeatStatus execute(StepContribution contribution,
                                ChunkContext chunkContext) throws Exception {

        // ExecutionContext 설정 
        ExecutionContext stepExecutionContext = getSetpExecutionContext(chunkContext);
        ExecutionContext jobExecutionContext = getJobExecutionContext(chunkContext);

        // Step ExecutionContext 설정 
        stepExecutionContext.put("FIRST_STEP_EXECUTION_CONTEXT", "DataSharingFirstTasklet Step Value1");
        
        // Job ExecutionContext 설정 
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
{% endcode %}

* 8 Line: Step ExecutionContext를 얻어오기 위해 ChunkContext에 있는 StepExecutionContext의 ExecutionContext를 가지고 온다&#x20;
* 9 Line: Job ExecutionContext를 얻어오기 위해 ChunkContext에 있는 StepExecutionContext의 JobExecution에서 ExecutionContext를 가지고 온다&#x20;
* 12 Line: Step ExecutionContext에 "FIRST\_STEP\_EXECUTION\_CONTEXT"에 값을 설정 하다.
* 15 Line: Job ExecutionContext에 "FIRST\_JOB\_EXECUTION\_CONTEXT"에 값을 설정 하다.

DataSharingFirstTasklet에서 Job ExecutionContext, Step ExecutionContext에 설정한 값을 출력하기 위해 DataSharingSecondTasklet를 작성한다.

{% code lineNumbers="true" %}
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
        
        // Step ExecutionContext 출력
        log.debug("FIRST_STEP_EXECUTION_CONTEXT :: " + FIRST_STEP_EXECUTION_CONTEXT);
        
        // Job ExecutionContext 출력
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
{% endcode %}

* 8 \~ 9 Line: Job ExecutionContext, Step ExecutionContext 객체를 선언한다.
* 11 \~ 18 Line: Step ExecutionContext( "FIRST\_STEP\_EXECUTION\_CONTEXT"), Job ExecutionContext( "FIRST\_JOB\_EXECUTION\_CONTEXT") 값을 출력한다.

## 2.  Chunk예제

Chunk 예제로 ItemReader, ItemProcessor, ItemWriter에서 Step ExecutionContext의 범위를 보기 위해서 아래와 같이 작성한다.

<figure><img src="../.gitbook/assets/image (257).png" alt=""><figcaption></figcaption></figure>

{% code lineNumbers="true" %}
```java
@Component
@Slf4j
public class StepItemReader implements ItemReader<TbDeployVO>, StepExecutionListener {

    private OpenCsvFileUtils openCsvFileUtils;


    @Override
    public void beforeStep(StepExecution stepExecution) {
        String file = stepExecution.getJobExecution().getJobParameters().getString("file");
        openCsvFileUtils = new OpenCsvFileUtils(file);
        
        // Step ExecutionContext 설정 
        stepExecution.getExecutionContext().put("READER_STEP", "StepItemReader beforeStep");
    }
    
    @Override
    public TbDeployVO read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
       ......
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        openCsvFileUtils.closeReader();
        // Step ExecutionContext 출력
        log.debug("StepItemReader :: READER_STEP :: " + stepExecution.getExecutionContext().get("READER_STEP"));
        return ExitStatus.COMPLETED;
    }
}
```
{% endcode %}

* 14, 26 Line:  Step ExecutionContext애 갑 설정 및 출력

{% code lineNumbers="true" %}
```java
@Component
@Slf4j
public class StepItemWriter implements ItemWriter<TbDeployWriteVO>, StepExecutionListener {

    private OpenCsvFileUtils openCsvFileUtils;
    @Override
    public void beforeStep(StepExecution stepExecution) {
        String file = stepExecution.getJobExecution().getJobParameters().getString("outfile");
        openCsvFileUtils = new OpenCsvFileUtils(file);
        
         // Step ExecutionContext 설정 
        stepExecution.getExecutionContext().put("WRITER_STEP", "StepItemWriter beforeStep");
    }

    @Override
    public void write(Chunk<? extends TbDeployWriteVO> tbDeployWriteVOs) throws Exception {
     .....

    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        openCsvFileUtils.closeWriter();
        
        // Step ExecutionContext 출력        
        log.debug("StepItemWriter :: READER_STEP :: " + stepExecution.getExecutionContext().get("READER_STEP"));
        log.debug("StepItemWriter :: PROCESSOR_STEP :: " + stepExecution.getExecutionContext().get("PROCESSOR_STEP"));
        log.debug("StepItemWriter :: WRITER_STEP :: " + stepExecution.getExecutionContext().get("WRITER_STEP"));

        // Job ExecutionContext 출력
        log.debug("StepItemWriter :: FIRST_JOB_EXECUTION_CONTEXT :: " + stepExecution.getJobExecution().getExecutionContext().get("FIRST_JOB_EXECUTION_CONTEXT"));
        return ExitStatus.COMPLETED;
    }

}
```
{% endcode %}

* 14, 26\~31 Line:  Step ExecutionContext애 갑 설정 및 출력

## 3.  Job Config

Tasklet, Chunk 를 실행하기 위해 Job을 설정 한다.

```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class DataSharingConfig {

    private final StepItemReader stepItemReader;
    private final StepItemProcessor stepItemProcessor;
    private final StepItemWriter stepItemWriter;

    // Job 설정 
    @Bean
    public Job dataSharingJob(JobRepository jobRepository,
                              Step firstStep,
                              Step secondStep,
                              Step case01Step) {
        return new JobBuilder("dataSharingJob", jobRepository)
                .incrementer(new RunIdIncrementer())
                .start(firstStep)
                .next(secondStep)
                .next(case01Step)
                .build();
    }

    // Step 설정 : DataSharingFirstTasklet 
    @Bean
    public Step firstStep(JobRepository jobRepository,
                          PlatformTransactionManager transactionManager,
                          DataSharingFirstTasklet dataSharingFirstTasklet) {
        return new StepBuilder("firstStep", jobRepository)
                .tasklet(dataSharingFirstTasklet, transactionManager)
                .build();
    }


    // Step 설정 : dataSharingSecondTasklet
    @Bean
    public Step secondStep(JobRepository jobRepository,
                           PlatformTransactionManager transactionManager,
                           DataSharingSecondTasklet dataSharingSecondTasklet) {
        return new StepBuilder("secondStep", jobRepository)
                .tasklet(dataSharingSecondTasklet, transactionManager)
                .build();
    }

    // Step 설정 : Chunk 설정정
    @Bean
    public Step case01Step(JobRepository jobRepository,
                           PlatformTransactionManager transactionManager
                           ) {
        return  new StepBuilder("CASE01_STEP", jobRepository)
                .<TbDeployVO, TbDeployWriteVO>chunk(2, transactionManager)
                .reader(stepItemReader)
                .processor(stepItemProcessor)
                .writer(stepItemWriter)
                .startLimit(1)
                .build();
    }
}
```

## 4.  결과

<figure><img src="../.gitbook/assets/image (218).png" alt=""><figcaption><p>Tasklet </p></figcaption></figure>

* Job ExecutionContext 출력: firstStep에서 설정하고 secondStep에서 출력 됨&#x20;
* Step ExecutionContext 출력 되지 않음:  firstStep에서 설정하고 secondStep에서 출력 되지 않음 \
  Step은 자신의 Step에서만 유효하다.

<figure><img src="../.gitbook/assets/image (219).png" alt=""><figcaption><p>Chunk</p></figcaption></figure>

* Job ExecutionContext 출력: firstStep에서 설정하고 case01Step에서 출력 됨&#x20;
* Step ExecutionContext 출력 :  case01Step의  StepItemReader , StepItemProcessor, StepItemWriter 에서 설정한 Step는 모두 출력됨

