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


    <figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

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


    <figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

    <figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

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


    <figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

    <figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

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

### 2-3. 실패 시 중지

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

*   ExitStatus.FAILED: 결과\


    <figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>
*   ExitStatus.COMPLETED: 결과\


    <figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>


*   ExitStatus.UNKNOWN: 결과\


    <figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

## 3. **Configuring for Stop**



## 4. **Programmatic Flow Decisions**

## 5. **Split Flows**

## 6. **Externalizing Flow Definitions and Dependencies Between Jobs**



