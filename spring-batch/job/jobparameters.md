# JobParameters

## 1. Tasklet에서 얻어오기

Tasklet은 execute() 메서드를 사용하여 실행하는데 여기에는 2개의 파라메터가 있다.

* StepContribution: 아직 커밋되지 않은 현재 트랜잭션에 대한 정보
* ChunkContext: 실행 시점의 잡 상태 제공

파라메터는 얻기 위해서는 ChunkContext 객체에  있는getStepContext() 메서드의  getJobParameters()에서 얻을 수 있다.

{% code lineNumbers="true" %}
```java
@Component
@Slf4j
 public class MyUserTasklet  implements Tasklet  {

    @Override
    public RepeatStatus execute(StepContribution contribution, 
                        ChunkContext chunkContext) throws Exception {
        log.debug("####-> MyUserTasklet Hello, World!");

        Map<String, Object> param = BatchUtils.getParam(chunkContext);
        log.debug("###-> MyUserTasklet param :" + param.toString());
        return RepeatStatus.FINISHED;
    }
}
```
{% endcode %}

ChunkContext 객체에서 파라메터를 얻어오는 예제 이다.

{% code lineNumbers="true" %}
```java
public class BatchUtils {
    public static Map<String, Object> getParam(ChunkContext chunkContext) {
        Map<String, Object> rnMap = new Gson().fromJson((String) chunkContext.getStepContext().getJobParameters().get("param"), Map.class);
        rnMap.put("timestamp", chunkContext.getStepContext().getJobParameters().get("timestamp").toString());

        rnMap.put("jobName", chunkContext.getStepContext().getJobName());
        rnMap.put("jobStep", chunkContext.getStepContext().getStepName());
        return rnMap;
    }
}
```
{% endcode %}

## 2 Listener 에서 얻어오기

### 2-1. StepExecutionListener

StepExecutionListener는beforeStep(), afterStep() 메소드의를 제공하며 각각 1개의 StepExecution객체를  파라메터로 가지고 있는데 이 객체의 JobExecution().getJobParameters() 메서드를 통해서 JobParameter 값을 얻을 수 있다.

{% code lineNumbers="true" %}
```java
private OpenCsvFileUtils openCsvFileUtils;

    @Override
    public void beforeStep(StepExecution stepExecution) {
        String file = stepExecution.getJobExecution().getJobParameters().getString("file");
        openCsvFileUtils = new OpenCsvFileUtils(file);
        log.debug("#### item -> StepItemReader beforeStep");
    }
    ...
}
```
{% endcode %}

### 2-2. JobExecutionListener

JobExecutionListener는 2개의 메소드를 beforeJob(), afterJob()메서드의 파라메터인 JobExecution 객체의 getJobParameters()메서드를 통해서 JobParameter 값을 얻을 수 있다.

