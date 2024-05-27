# Controlling Step Flow

스프링 배치에서   Job은 Step로 구성이 되는데 Step의 처리 순서를 결정할 수 있다.

다음 코드는 Step 코드이다.

* startStep: 시작 Step
* failedStep: 시작 Step가 "FAILED"인 경우 실행되는 Step
* completedStep: 시작 Step가 "COMPLETED"인 경우 실행되는 Step
* finishStep: 마직막으로 실행 되는 Step

```java
@Bean
public Step startStep(JobRepository jobRepository,
                      PlatformTransactionManager transactionManager ) {
    return new StepBuilder("START_STEP_" + CNT, jobRepository)
            .tasklet((contribution, chunkContext) -> {
                log.info("START STEP!");
                contribution.setExitStatus(EXIT_STATUS);
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
```

## 1.  **Sequential Flow**

Step를 순차적으로 싫행한다.

<figure><img src="../.gitbook/assets/image (9).png" alt="" width="563"><figcaption></figcaption></figure>

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

*   결과:  \


    <figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 2.  **Conditional Flow**

선행 Step의 결과에 따라서 다른 Step을 진행 하는 것으로 선행 Step의 결과는 **Step의 ExitStatus를 참조** 한다,

<figure><img src="../.gitbook/assets/image (237).png" alt="" width="563"><figcaption></figcaption></figure>

패턴에는 두 개의 특수 문자만 허용된다.

* "\*": 는 0개 이상의 문자와 일치 \[ "c\*t"는 "cat" 및 "count"와 일치 ]
* "?": 정확히 한 문자와 일치. \["c?t"는 "cat"과 일치하지만 "count"와 일치하지 않는다]

### 2-1. "FAILED" 인 경우

<pre class="language-java"><code class="lang-java"><strong>private final ExitStatus EXIT_STATUS =  ExitStatus.FAILED;
</strong>
@Bean
public Job JobFlowJob(JobRepository jobRepository,
                  Step startStep,
                  Step failedStep,
                  Step completedStep,
                  Step finishStep){

        // Sequential Flow
        return new JobBuilder("JOB_" + CNT, jobRepository)
                .start(startStep)
                .on("*").to(completedStep)
                .from(startStep).on("FAILED").to(failedStep)
                .end()
                .build();
}
</code></pre>

*   결과:  시작 Step 결과가  FAILED 이고 "FAILED"일 떄 실행되는 Step은은 COMPLETED 이다. Job은 정상 수행이 되어 COMPLETED가 된다\


    <figure><img src="../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

    <figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 2-2. "COMPLETED" 인 경우

```java
private final ExitStatus EXIT_STATUS =  ExitStatus.COMPLETED;

@Bean
public Job JobFlowJob(JobRepository jobRepository,
                  Step startStep,
                  Step failedStep,
                  Step completedStep,
                  Step finishStep){

        // Sequential Flow
        return new JobBuilder("JOB_" + CNT, jobRepository)
                .start(startStep)
                .on("*").to(completedStep)
                .from(startStep).on("FAILED").to(failedStep)
                .end()
                .build();
}
```

*   결과:  시작 Step 결과가  COMPLETED이고 "COMPLETED"일 떄 실행되는 Step은 COMPLETED 이다. Job은 정상 수행이 되어 COMPLETED가 된다  \


    <figure><img src="../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

    <figure><img src="../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

<details>

<summary>다음 코드의 예상 결과는 .......</summary>

```java
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
                    .to(failedStep)            // failedStep 실행
                    .on("*")            // failedStep 결과 상관없이
                    .to(finishStep)            // finishStep 실행
                    .on("*")            // finishStep 결과 상관없이
                    .end()                     // Flow 종료.
                .from(startStep)               // ExitStatus : COMPLETED
                    .on("COMPLETED")    // COMPLETED일 경우
                    .to(completedStep)         // completedStep 실행
                    .on("*")            // completedStep 결과 상관없이
                    .to(finishStep)            // finishStep 실행
                    .on("*")            // finishStep 결과 상관없이
                    .end()                     // Flow를 종료.
                .from(startStep)               // ExitStatus : NOT FAILED AND NOT COMPLETED
                    .on("*")            // 모든 경우
                    .to(finishStep)            // finishStep 실행
                    .on("*")            // finishStep 결과 상관없이
                    .end()                     // Flow를 종료.
                .end()
                .build();
    }
```

</details>

## 3. **Configuring for Stop**

<figure><img src="../.gitbook/assets/image (256).png" alt="" width="563"><figcaption></figcaption></figure>

### 3-1. end ( Job: COMPLETED)

```java
return new JobBuilder("JOB_" + CNT, jobRepository)
                .start(startStep)
                .on("FAILED").end()
                .from(startStep)
                   .on("COMPLETED")
                   .to(completedStep)
                   .on("*")
                   .to(finishStep)
                .from(startStep)
                   .on("*")
                   .to(finishStep)
                .end()
                .build();
```

```
private final ExitStatus EXIT_STATUS =  ExitStatus.FAILED;
```

* ExitStatus.FAILED:   결과 : 중지

<figure><img src="../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

*   ExitStatus.COMPLETED: 결과\


    <figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>


*   ExitStatus.UNKNOWN: 결과\


    <figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

### 3-2.  fail ( Job: FAILED)&#x20;

```java
return new JobBuilder("JOB_" + CNT, jobRepository)
        .start(startStep)
        .next(nextStep)
        .on("FAILED").fail()
        .from(nextStep).on("COMPLETED").to(completedStep).on("*").to(finishStep)
        .from(nextStep).on("*").to(finishStep)
        .end()
        .build();
```

nextStep에 따라서 실행 Step가 결정 되는데 **fail() 함수 적용시 Job은 실패로 됨**

<figure><img src="../.gitbook/assets/image (238).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (239).png" alt=""><figcaption></figcaption></figure>

### 3-3.  stopAndRestart  (Job: STOPPED -> COMPLETED)

조건이 만족 하면 해당 Step는 COMPLETED 되고 Job은 STOPPED 상태로 종료되어 다시 실행 하면 nextStep만 실행한다.

```java
return new JobBuilder("JOB_" + CNT, jobRepository)
        .start(startStep)
            .on("COMPLETED")
            .stopAndRestart(nextStep)
        .end()
        .build();
```

<figure><img src="../.gitbook/assets/image (240).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (241).png" alt=""><figcaption></figcaption></figure>

* 다시 시작: nextStep 만 실행 됨&#x20;

<figure><img src="../.gitbook/assets/image (242).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (243).png" alt=""><figcaption></figcaption></figure>

## 4. **Programmatic Flow Decisions**

다음 단계에 대한 실행을 동적으로 작성하기 위해 Decider를 작성해서 제어한다.

다음 코드는 사용자 정의 Decider 코드 이다.

```java
@Component
@Slf4j
public class UserDecider implements JobExecutionDecider {
    @Override
    public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {

        String status;
        if (true) { // 조건식
            status = "USER1";
        }
        else {
            status = "USER2";
        }
        log.debug("decide : staus :  " + status);
        return new FlowExecutionStatus(status);
    }
}
```

```java
@Bean
public Job JobFlowJob(JobRepository jobRepository,
    Step startStep,
    Step failedStep,
    Step completedStep,
    UserDecider userDecider){


    // Programmatic  Flow
    return new JobBuilder("JOB_" + CNT, jobRepository)
    .start(startStep)
    .next(userDecider).on("USER1").to(failedStep)
    .from(userDecider).on("USER2").to(completedStep)
    .end()
    .build();
}
```

* 결과: userDecider에서 "USER1"을 리턴 하여 failedStep 가 실행된다.

<figure><img src="../.gitbook/assets/image (245).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (246).png" alt=""><figcaption></figcaption></figure>

## 5. **Split Flows**

Step를 group로 묶어서 실행하는 방법으로 비동기 실행을 할 수 있다.

* flow1: startStep, nextStep
* flow2: next3Step
* step: finishStep
* split: SimpleAsyncTaskExecutor를 사용해서 flow2를 비동기로 실행

<figure><img src="../.gitbook/assets/image (255).png" alt="" width="563"><figcaption></figcaption></figure>

```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class JobFlowConfig {

    private final String CNT = "SPLIT_003";
    private final ExitStatus EXIT_STATUS =  ExitStatus.COMPLETED;

    @Bean
    public Job JobFlowJob(JobRepository jobRepository,
                          Flow flow1,
                          Flow flow2,
                          Step finishStep){
 

        return new JobBuilder("JOB_FLOW_" + CNT, jobRepository)
                .start(flow1)
                .split(new SimpleAsyncTaskExecutor())
                .add(flow2)
                .next(finishStep)
                .end()
                .build();
    }


    @Bean
    public Flow flow1(Step startStep,
                      Step nextStep) {
        return new FlowBuilder<SimpleFlow>("FLOW_01_"+ CNT)
                .start(startStep)
                .next(nextStep)
                .build();
    }

    @Bean
    public Flow flow2(Step next3Step) {
        return new FlowBuilder<SimpleFlow>("FLOW_02_"+ CNT)
                .start(next3Step)
                .build();
    }
    @Bean
    public Step startStep(JobRepository jobRepository,
                          PlatformTransactionManager transactionManager ) {
        return new StepBuilder("START_STEP_" + CNT, jobRepository)
                .tasklet((contribution, chunkContext) -> {
                    log.info("START STEP!");
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .build();
    }

    @Bean
    public Step nextStep(JobRepository jobRepository,
                          PlatformTransactionManager transactionManager ) {
        return new StepBuilder("NEXT_STEP_" + CNT, jobRepository)
                .tasklet((contribution, chunkContext) -> {
                    log.info("NEXT STEP!");
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .build();
    }

    @Bean
    public Step next3Step(JobRepository jobRepository,
                         PlatformTransactionManager transactionManager ) {
        return new StepBuilder("NEXT3_STEP_" + CNT, jobRepository)
                .tasklet((contribution, chunkContext) -> {
                    log.info("NEXT3 STEP!");
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

* 결과: flow1, flow2 가 비동기로 실행되어 startStep. next3Step, nextStep 로 같이 실행이 되고 마직막으로 finishStep가 실행 된다.

<figure><img src="../.gitbook/assets/image (248).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (249).png" alt=""><figcaption></figcaption></figure>

## 6. Job에서 Job 실행

Job에서 다른 Job를 살행 할 수 있다.

<figure><img src="../.gitbook/assets/image (254).png" alt=""><figcaption></figcaption></figure>

```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class JobFlowConfig {

    private final String CNT = "EXP_JOB_005";

    @Bean
    public Job JobFlowJob(JobRepository jobRepository,
                          Step  jobStepJobStep1){       

        return new JobBuilder("JOB_FLOW_" + CNT, jobRepository)
                .start(jobStepJobStep1)
                .build();
     
    }

    @Bean
    public Step jobStepJobStep1(JobRepository jobRepository,
                                JobLauncher jobLauncher,
                                JobParametersExtractor jobParametersExtractor,
                                Job job) {
        return new StepBuilder("JOB_STEP_jobStepJobStep1_" + CNT, jobRepository)
                .job(job) 
                .build();
    }

    @Bean
    public Job job(JobRepository jobRepository,
                   Step startStep) {
        return new JobBuilder("JOB_FLOW_RUN_JOB_" + CNT, jobRepository)
                .start(startStep)
                .build();
    }
    
    @Bean
    public Step startStep(JobRepository jobRepository,
                          PlatformTransactionManager transactionManager ) {
        return new StepBuilder("START_STEP_" + CNT, jobRepository)
                .tasklet((contribution, chunkContext) -> {
                    log.info("START STEP!");
                    Map<String, Object> param = chunkContext.getStepContext().getJobParameters();
                    Iterator var2 = chunkContext.getStepContext().getJobParameters().entrySet().iterator();

                    while(var2.hasNext()) {
                        Map.Entry entry = (Map.Entry) var2.next();
                        log.debug("Key: " + (String)entry.getKey() + ", Value:" + entry.getValue());

                    }
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .build();
    }

}
```

* 결과:&#x20;

<figure><img src="../.gitbook/assets/image (252).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (253).png" alt=""><figcaption></figcaption></figure>
