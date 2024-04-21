# FlatFileItem

대량 데이터 일괄 처리는 데이터 수집, 처리, 결과를 생성하는데 효율적인 방법으로 스프링 배치는 로깅, 트랜잭션 관리, 작업 다시 시작(작업이 완료되지 않은 경우), 작업 건너뛰기, 작업 처리 통계 및 리소스 관리하는 기능를 제공하는데 이 예제는 File To File을 FlatFileItemReader, FlatFileItemWriter을 사용 합니다.

참고 : [Flat Files](https://docs.spring.io/spring-batch/reference/readers-and-writers/flat-files.html)&#x20;

플랫 파일 리소스의 종류로는 PathResource,  FileSystemResource, InputStreamResource, UrlResource, ByteArrayResource 등등이 될 수 있으며 ClassPathResource를 사용하는 경우 클래스 경로에 파일을 넣어야 하며 FileSystemResource를 사용하는 경우 절대 경로도 설정할 수 있습니다

<figure><img src="../../../.gitbook/assets/image (197).png" alt=""><figcaption><p>Item Processor를 사용한 청크 지향 처리</p></figcaption></figure>

## 1. 배치 Config&#x20;

스프링 배치를 사용하기 위해서 Configuration 파일을 생성 합니다.

{% code title="BeanFlatFileConfig.java" lineNumbers="true" %}
```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class BeanFlatFileConfig {
    private final ApplicationArguments args;
}
```
{% endcode %}

* 5 Line: 배치 실행시 파라메터를 가지고 오기 위해서 주입&#x20;
  * 예:    --job=beanFlatFileJob \
    &#x20;        \--file=D:/ABACUS\_PRJ/abacus/acube-svc-batch/file/in/TB\_DEPLOY.csv \
    &#x20;        \--outfile=D:/ABACUS\_PRJ/abacus/acube-svc-batch/file/out/TB\_DEPLOY\_BEAN.csv

## 2. ItemReader 생성 ( FlatFileItemReader )

CSV 파일을 포함하여 플랫 파일에서 데이터를 읽는 데 사용되며 플랫 파일이란 CSV 파일이나 고정 길이 파일과 같은 형식 처럼 이차원적으로 구성된 파일을 의미 합니다.

FlatFileItemReader은  Resource와  LineMapper를 정의 하여 구성되며 속성과 기능은 다음과 같습니다.

* **Resource**: 읽어올 파일의 리소스를 지정합니다. 파일 시스템, 클래스 경로, URL 등 다양한 리소스를 사용
* **LineMapper**: String 라인을 도메인 객체로 변환합니다. 각 라인을 어떻게 매핑할지 설정.
* **linesToSkip**: 파일 상단에서 무시할 라인 수를 지정.
* **recordSeparatorPolicy**: 라인 종료 위치를 결정하고, 따옴표 안에서 라인 종료를 처리하는 데 사용.
* **encoding**: 사용할 텍스트 인코딩을 지정(기본값은 UTF-8).

파일을 읽어오기 위해서 ItemReader로 FlatFileItemReader를 @Bean으로 선언하는 코드로 Step 선언시 reader에 사용됩니다.

* _DelimitedLineTokenizer:_ 열 이름을 지정
* _BeanWrapperFieldSetMapper:_ 각 줄을 _Person_ 개체에 매핑

<details>

<summary>BeanFlatFileItemReader.java</summary>

{% code lineNumbers="true" %}
```java
/**
 * FlatFileItemReader을 사용해서 파일을 읽어 온다.
 */
@Slf4j
public class BeanFlatFileItemReader {

    /**
     * FlatFileItemReader 객체를 생성 하고 파일을 읽어서 객체에 저장 하여 반환하는 기능  
     * - ApplicationArguments로 받은 데이터 중 "file"에 정의한 파일을 읽는다.
     * - lineMapper : 라인 Mapper를 통해서 라인 단위로 읽어 tokenizer후 객체에 저장 
     * @param args
     * @return
     */
    public FlatFileItemReader<TbDeployVO> reader(ApplicationArguments args)   {
        log.debug("#### item -> BeanFlatFileItemReader FlatFileItemReader ");
        // ApplicationArgumentsr객체에서 file값 읽어 오기
        String file = BatchUtils.getArgumentByKey(args, "file");

        // FlatFileItemReader 객체 생성
        FlatFileItemReader<TbDeployVO> flatFileItemReader = 
            new FlatFileItemReader<>();
        flatFileItemReader.setName("BeanFlatFileItemReader");

        // PathResource 사용해서 File Resource 지정
        // - new ClassPathResource(path): ClassPath 에서 읽어옴
        // - new FileSystemResource(path): File System 에서 읽어옴
        flatFileItemReader.setResource(ResourceUtils.getResource(file));

        // lineMapper 생성 : Line 단위로 읽어 오기 위해서 정의
        flatFileItemReader.setLineMapper(lineMapper());
        // flatFileItemReader.open(new ExecutionContext());
        return flatFileItemReader;
    }


    /**
     * Line 단위로 읽어 오기 위한 기능으로
     * - LineTokenizer : 라인 단위로 읽어올 때 어떻게 분리 하고 HashSet에 순서 대로 Field 명 정의
     * - FieldSetMapper : 라인 단위로 Tokenizer한 HashSet을 객체로 변환
     * @return
     */
    private DefaultLineMapper<TbDeployVO> lineMapper() {
        return new DefaultLineMapper<>() {
            {
                setLineTokenizer(tokenizer());
                setFieldSetMapper(fieldSetMapper());
            }
        };
    }

    /**
     * 라인 단위로 읽어올 때 어떻게 분리 하고 HashSet에 순서 대로 Field 명 정의
     * @return
     */
    private DelimitedLineTokenizer tokenizer() {
        return new DelimitedLineTokenizer() {
            {
                setDelimiter(",");
                setNames("empUuid",
                        "createDateTime",
                        "createUserId",
                        "updateDateTime",
                        "updateUserId",
                        "deployDiv",
                        "deployEndDate",
                        "deployInDate",
                        "deployRsvDate",
                        "empName",
                        "empNo",
                        "projId");
            }
        };
    }

    /**
     * 라인 단위로 Tokenizer한 HashSet을 객체로 변환
     * @return
     */
    private BeanWrapperFieldSetMapper<TbDeployVO> fieldSetMapper() {
        return new BeanWrapperFieldSetMapper<>() {
            {
                setTargetType(TbDeployVO.class);
            }
        };
    }
}
```
{% endcode %}

</details>

## 3. ItemProcessor 생성

FlatFileItemReader에서 읽을 값을 출력하기 위해서 변환하는 기능으로 이곳에 비지니스 규칙을 정의 할 수 있습니다.

<details>

<summary>BeanFlatFileItemProcessor.java</summary>

{% code lineNumbers="true" %}
```java
/**
 * ItemReader 에서 받은 값을 ItemWriter로 전달 하는 기능으로 
 * 이곳에 비지니스 규칙을 정의 할 수 있다
 */
@Slf4j
public class BeanFlatFileItemProcessor 
        implements ItemProcessor<TbDeployVO, TbDeployWriteVO>,
                StepExecutionListener {

    @Override
    public void beforeStep(StepExecution stepExecution) {
        log.debug("#### item -> StepFlatFileItemProcessor beforeStep.");
    }

    /**
     * ItemReader 에서 받은 값을 ItemWriter로 전달 하는 기능
     * @param tbDeployVO
     * @return
     */
    @Override
    public TbDeployWriteVO process(TbDeployVO tbDeployVO)   {
        log.debug("#### item -> StepFlatFileItemProcessor process " 
                + tbDeployVO.toString());
        
        if (tbDeployVO == null) return null;
        
        // MapStruct를 이용해서 변환 
        return Case01Mapper.INSTANCE.tbDeployVOToTbDeployWriteVO(tbDeployVO);
    }


    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        log.debug("#### item -> StepFlatFileItemProcessor afterStep ");
        return ExitStatus.COMPLETED;
    }
}

```
{% endcode %}

</details>

## 4. ItemWriter 생성

* _BeanWrapperFieldExtractor_ :  모델 객체의 필드 이름(예: _이 경우_ TbDeployWriteVO)을 가져와서 인스턴스에서 값을 추출하여 최종적으로 _DelimitedLineAggregator_에 전달.
* _DelimitedLineAggregator:_ 모든 필드가 구분 기호로 구분되는 플랫 파일 형식으로 필드 데이터를 정렬.
* 마지막으로 구분된 데이터가 _WriteableResource_에 기록.

<details>

<summary>BeanFlatFileItemWriter.java</summary>

{% code lineNumbers="true" %}
```java
/**
 * ItemProcessor에서 받은 객체(TbDeployWriteVO)의 정보를 FlatFileItemWriter객체를
 * 통해서 파일로 쓰기 기능 
 */
@Slf4j
public class BeanFlatFileItemWriter {

    public FlatFileItemWriter<TbDeployWriteVO> writer(ApplicationArguments args) {
        log.debug("#### item -> StepFlatFileItemWriter write ");
        String file = BatchUtils.getArgumentByKey(args, "outfile");
        return new FlatFileItemWriter<>() {
            {
                setName("BeanFlatFileItemWriter");
                // 쓰기 Resource : new PathResource(path)
                // - new ClassPathResource(path): ClassPath 파일로 쓰기
                // - new FileSystemResource(path): File System 파일로 쓰기
                setResource(ResourceUtils.getWriteResource(file));
                
                // 파일이 있으면 삭제 후 신규 쓰기
                setShouldDeleteIfExists(true);
                
                // 라인 단위 쓰기 할 떼 속성 지정 및 객체를 String[]로 변환 
                setLineAggregator(delimitedLineAggregator());
            }
        };
    }

    /**
     * DelimitedLineAggregator : 라인 단위 쓰기 할 떼 속성 지정 및 객체를 String[]로 변환 
     * - setDelimiter: 구분자 지정 
     * - setFieldExtractor: 객체를 String[]로 변환 
     * @return
     */
    private DelimitedLineAggregator  delimitedLineAggregator() {
        return new DelimitedLineAggregator() {
            {
                setDelimiter(",");
                setFieldExtractor(beanWrapperFieldExtractor());
            }
        };
    }

    /**
     * 객체를 String[]로 변환 
     * @return
     */
    private  BeanWrapperFieldExtractor beanWrapperFieldExtractor() {
        return new BeanWrapperFieldExtractor() {
            {
                setNames(new String[]{"empUuid",
                    "deployDiv",
                    "deployEndDate",
                    "deployInDate",
                    "deployRsvDate",
                    "empName",
                    "empNo",
                    "projId"});
            }
        };
    }

}
```
{% endcode %}

</details>

## 5. Step & Job 생성

배치 Config에서 작성한 코드를 완성 합니다.

* beanFlatFileJob : Job 등록
  * JobCompletionNotificationListener: JobExecutionListener의 구현체로 Job의 상태를 관찰 하기 위한 Listener
* beanFlatFileStep: Step 등록
  * chunk: 2개씩 읽어서 처리
  * Listener: Step, Reader, Process, Writer을 관찰 하기 위한 Listener
* beanFlatFileItemReader:
  * FlatFileItemReader로 파일을 읽기 위한 객체 빈 선언
* beanFlatFileItemProcessor:
  * ItemProcessor로 읽은 내용을 가공하기 위한 객체 빈 선언&#x20;
* beanFlatFileItemWriter:
  * FlatFileItemWrite로 파일에 쓰기 위한 객체 빈 선언&#x20;

{% code lineNumbers="true" %}
```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class BeanFlatFileConfig {

    private final ApplicationArguments args;

    @Bean
    public Job beanFlatFileJob(JobRepository jobRepository,
                               Step beanFlatFileStep ) {
        return new JobBuilder("BEAN_FLAT_FILE_JOB", jobRepository)
                .flow(beanFlatFileStep)
                .end()
                .listener(new JobCompletionNotificationListener(args))
                .build();
    }

    @Bean
    public Step beanFlatFileStep(JobRepository jobRepository,
                           PlatformTransactionManager transactionManager
                           ) {
        log.debug("####->  case01Step!");
        return new StepBuilder("BEAN_FLAT_FILE_STEP", jobRepository)
                .listener(new StepComplateNotiListener())
                .<TbDeployVO, TbDeployWriteVO>chunk(2, transactionManager)
                .reader(beanFlatFileItemReader())
                .listener(new BeanFlatFileItemReaderListener())
                .processor(beanFlatFileItemProcessor())
                .listener(new BeanFlatFileItemProcessorListener())
                .writer(beanFlatFileItemWriter())
                .listener(new BeanFlatFileItemWriterListener())
                .build();
    }

    @Bean
    FlatFileItemReader<TbDeployVO> beanFlatFileItemReader()   {
        return new BeanFlatFileItemReader().reader(args);
    }

    @Bean
    ItemProcessor<TbDeployVO, TbDeployWriteVO> beanFlatFileItemProcessor() {
        return new BeanFlatFileItemProcessor();
    }
    @Bean
    public FlatFileItemWriter<TbDeployWriteVO> beanFlatFileItemWriter() {
        return new BeanFlatFileItemWriter().writer(args);
    }

}
```
{% endcode %}

## 6. 빈 주입&#x20;

&#x20;Reader, Processor, Writer을 모두 @Bean으로 등록 하였기 때문에 다음과 같은 코드로 변경해도 됩니다.

{% code lineNumbers="true" %}
```java
@Bean
public Step beanFlatFileStep(JobRepository jobRepository,
                             PlatformTransactionManager transactionManager,
                             ItemReader<TbDeployVO> beanFlatFileItemReader,
                             ItemProcessor<TbDeployVO,TbDeployWriteVO> beanFlatFileItemProcessor,
                             ItemWriter<TbDeployWriteVO> beanFlatFileItemWriter

) {
    log.debug("####->  case01Step!");
    return new StepBuilder("BEAN_FLAT_FILE_STEP", jobRepository)
            .listener(new StepComplateNotiListener())
            .<TbDeployVO, TbDeployWriteVO>chunk(2, transactionManager)
            .reader(beanFlatFileItemReader)
            .listener(new BeanFlatFileItemReaderListener())
            .processor(beanFlatFileItemProcessor)
            .listener(new BeanFlatFileItemProcessorListener())
            .writer(beanFlatFileItemWriter)
            .listener(new BeanFlatFileItemWriterListener())
            .build();
}
```
{% endcode %}

## 7. Listener

다음은 리스너 코드 입니다.

{% tabs %}
{% tab title="BeanFlatFileItemReaderListener " %}
```java
@Slf4j
public class BeanFlatFileItemReaderListener implements ItemReadListener<TbDeployVO> {

    @Override
    public void beforeRead() {        
        log.info("신규 TbDeployVO 읽기");
    }

    @Override
    public void afterRead(TbDeployVO input) {
        
        log.info("신규 TbDeployVO 읽은 정보 : " + input);
    }

    @Override
    public void onReadError(Exception e) {        
        log.error("오류  TbDeployVO  : " + e);
    }

}
```
{% endtab %}

{% tab title="BeanFlatFileItemProcessorListener" %}
```java
@Slf4j
public class BeanFlatFileItemProcessorListener
        implements ItemProcessListener<TbDeployVO, TbDeployWriteVO> {

    @Override
    public void beforeProcess(TbDeployVO input) {
        log.info("TbDeployVO 처리 전 " + input);
    }

    @Override
    public void afterProcess(TbDeployVO input, TbDeployWriteVO result) {
        log.info("TbDeployWriteVO 처리 후  : " + result);
    }

    @Override
    public void onProcessError(TbDeployVO input, Exception e) {
        log.error("오류 TbDeployVO  : " + input);
        log.error("오류 메세지 : " + e);
    }
}
```
{% endtab %}

{% tab title="BeanFlatFileItemWriterListener" %}
```java
@Slf4j
public class BeanFlatFileItemWriterListener implements ItemWriteListener<TbDeployWriteVO> {

    @Override
    public void beforeWrite(Chunk<? extends TbDeployWriteVO> items) {
        log.info("쓰기 전 TbDeployWriteVO 목록 : " + items);
    }

    @Override
    public void afterWrite(Chunk<? extends TbDeployWriteVO> items) {
        log.info("쓰기 완료 TbDeployWriteVO 목록 : " + items);
        ;
    }

    @Override
    public void onWriteError(Exception e, Chunk<? extends TbDeployWriteVO> items) {
        log.error("쓰기 오류 : 목록 :" + items);
        log.error("쓰기 오류 : 에러 : " + e);
    }

}
```
{% endtab %}

{% tab title="JobCompletionNotificationListener " %}
```java
@Slf4j
public class JobCompletionNotificationListener implements JobExecutionListener {

    private ApplicationArguments args;

    public JobCompletionNotificationListener(ApplicationArguments args) {
        this.args = args;
    }

    @SneakyThrows
    @Override
    public void afterJob(JobExecution jobExecution) {
        if (jobExecution.getStatus() == BatchStatus.COMPLETED) {
            log.info("JOB FINISHED !!");
    }
}
```
{% endtab %}

{% tab title="StepComplateNotiListener" %}
```java
@Slf4j
public class StepComplateNotiListener implements StepExecutionListener {

    @Override
    public void beforeStep(StepExecution stepExecution) {
        log.debug("#### item -> StepFlatFileItemProcessor beforeStep." + stepExecution.getStepName());
    }
    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        log.debug("#### item -> StepFlatFileItemProcessor afterStep " + stepExecution.getStepName());
        return ExitStatus.COMPLETED;
    }
}
```
{% endtab %}
{% endtabs %}



## 8. 결과&#x20;

<figure><img src="../../../.gitbook/assets/image (198).png" alt=""><figcaption></figcaption></figure>
