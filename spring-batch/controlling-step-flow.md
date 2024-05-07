# Controlling Step Flow

스프링 배치에서   Job은 Step로 구성이 되는데 Step의 처리 순서를 결정할 수 있다.

## 1.  **Sequential Flow**

Step를 순차적으로 싫행한다.

<figure><img src="../.gitbook/assets/image.png" alt="" width="563"><figcaption></figcaption></figure>

{% code lineNumbers="true" %}
```java
@Bean
public Job JobFlowJob(JobRepository jobRepository,
                      Step startStep,
                      Step failedStep,
                      Step completedStep,
                      Step finishStep){

    return new JobBuilder("JOB_" + CNT, jobRepository)
            .start(startStep)
            .next(completedStep)
            .next(finishStep)
            .build();
}
```
{% endcode %}

* 결과:&#x20;

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## 2.  **Conditional Flow**

선행 Step의 결과에 따라서 다른 Step을 진행 하는 것으로 선행 Step의 결과는 **Step의 ExitStatus를 참조** 한다,

<figure><img src="../.gitbook/assets/image (237).png" alt="" width="563"><figcaption></figcaption></figure>

## 3. **Configuring for Stop**

## 4. **Programmatic Flow Decisions**

## 5. **Split Flows**

## 6. **Externalizing Flow Definitions and Dependencies Between Jobs**







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
