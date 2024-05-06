# 스템 흐름 제어

Step의 결과에 따라 조건으로 다른 Step 실행&#x20;



{% code lineNumbers="true" %}
```java
@Configuration
@Slf4j
public class JobFlowConfig {


    private final String CNT = "FLOW_002";
    @Bean
    public Job JobFlowJob(JobRepository jobRepository,
                          Step startStep,
                          Step failedStep,
                          Step completedStep,
                          Step finishStep){
        // Flow에서 on은 RepeatStatus가 아닌 ExitStatus을 참조 한다.
        return new JobBuilder("JOB_" + CNT, jobRepository)
                .start(startStep)
                    .on("FAILED")       // ExitStatus: FAILED
                    .to(failedStep)     // failedStep 실행
                    .on("*")            // failedStep 결과 상관없이
                    .to(finishStep)     // finishStep 실행
                    .on("*")            // finishStep 결과 상관없이
                    .end()              // Flow 종료.
                .from(startStep)        // ExitStatus : COMPLETED
                    .on("COMPLETED")    // COMPLETED일 경우
                    .to(completedStep)  // completedStep 실행
                    .on("*")            // completedStep 결과 상관없이
                    .to(finishStep)     // finishStep 실행
                    .on("*")            // finishStep 결과 상관없이
                    .end()              // Flow를 종료.
                .from(startStep)        // ExitStatus : NOT FAILED AND NOT COMPLETED
                    .on("*")            // 모든 경우
                    .to(finishStep)     // finishStep 실행
                    .on("*")            // finishStep 결과 상관없이
                    .end()              // Flow를 종료.
                .end()
                .build();
    }

    @Bean
    public Step startStep(JobRepository jobRepository,
                          PlatformTransactionManager transactionManager ) {
        return new StepBuilder("START_STEP_" + CNT, jobRepository)
                .tasklet((contribution, chunkContext) -> {
                    log.info("START STEP!");
                    contribution.setExitStatus(ExitStatus.COMPLETED);
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .build();
    }

    @Bean
    public Step failedStep(JobRepository jobRepository,
                             PlatformTransactionManager transactionManager){
        return new StepBuilder("FAILED_STEP_" + CNT, jobRepository)
                .tasklet((contribution, chunkContext) -> {
                    log.info("FAILED STEP!");
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .build();
    }

    @Bean
    public Step completedStep(JobRepository jobRepository,
                            PlatformTransactionManager transactionManager){
        return new StepBuilder("COMPLETED_STEP_" + CNT, jobRepository)
                .tasklet((contribution, chunkContext) -> {
                    log.info("COMPLETED STEP !");
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .build();
    }


    @Bean
    public Step finishStep(JobRepository jobRepository,
                          PlatformTransactionManager transactionManager){
        return new StepBuilder("FINISHED_STEP" + CNT, jobRepository)
                .tasklet((contribution, chunkContext) -> {
                    log.info("FINISHED STEP!");
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .build();
    }

}

           
```
{% endcode %}
