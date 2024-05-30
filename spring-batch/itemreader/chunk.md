# Chunk

스프링 배치에서 Step을 처리 하는 방법 중 하나로 큰 데이터를 쪼개서 처리 하는 트랜잭션 단위로 Chunk단위로 읽어서 처리 하는 것으로 오류가 발생시 Chunk로 지정한 수 만큼 롤백 처리 됩니다.

## 1.  요구사항

csv 파일을 읽어서 다른 파일 csv로 데이터를 저장하는 기능으로 다음 기능을 만족해야 합니다.

1. 읽을 파일과 쓰기 파일은 외부에서 받아서 차리해야 한다.
2. classpath에 지정한 파일이 없는 경우 지정한 파일을 읽는다.
3. chunk의 크기는 외부에서 받아서 동적으로 처리해아 한다.
4. 배치가 수행 되고 나면 촐 커밋 수, 오류 수등 집계를 제공해야 한다.

## 2.  생각하기

스프링 배치에서 Chunk는 아래 그림처럼 처리 됩니다.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Step 지정시 Chunk의 수을 지정 하면 ItemReader이 지정한 수 만큼 읽어서 Processor에 전달 합니다.
* Processor에서는 Chunk에 지정한 수 만큼 처리 하고 Writer로 전달 합니다.
* 즉. 전체가 3이고 Chunk를 2로 지정 하면 reader가 2개 읽어 Processor로 전달하고 Processor는 2개를 처리 하고 writer로 Array 형태 (Chunk)로 전달 하여 처리 하고 다시 Reader를 읽어 Step을 처리 합니다.&#x20;
* 요구사항을 만족하기 위한 기능 처리&#x20;
  1. [classpath에 지정한 파일이 없는 경우 지정한 파일을 읽는다](chunk.md#id-3-2.-in-out).
     * opencsv 의존성을 pom.xml에 선언한다.
     * 파일을 읽을 때 ClassLoader 객체르 사용해서 파일을 검사하여 있으면 해당 위치에서 파일을 읽고 그렇지 않으면 지정한 위치에서 읽게 한다.
  2. [읽을 파일과 쓰기 파일은 외부에서 받아서 차리해야 한다.](chunk.md#id-3-3.-file-read-write)
  3. [chunk의 크기는 외부에서 받아서 동적으로 처리해아 한다](chunk.md#id-3-3.-chunksize).
  4. [배치가 수행 되고 나면 총 커밋 수, 오류 수등 집계를 제공해야 한다.](chunk.md#id-3-4)

## 3.  코드작성

* Spring Batch: spring-boot-starter-batch:3.2.4
* Lombok: lombok:1.18.30
* MariaDB: mariadb-java-client:1.18.30
* mapstruct:  1.5.5.Final
* opencsv: 5.9

### 3-1. 의존성 추가

<details>

<summary>pom.xml</summary>

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>3.2.3</version>
	<relativePath/> <!-- lookup parent from repository -->
</parent>

<properties>
	<java.version>21</java.version>
	<org.projectlombok.version>1.18.30</org.projectlombok.version>
	<org.mapstruct.version>1.5.5.Final</org.mapstruct.version>
</properties>

<dependencies>	
	<dependency>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-batch</artifactId>
	</dependency>
	        
	<dependency>
		<groupId>org.mapstruct</groupId>
		<artifactId>mapstruct</artifactId>
		<version>${org.mapstruct.version}</version>
	</dependency>
	
	<dependency>
		<groupId>org.mapstruct</groupId>
		<artifactId>mapstruct-processor</artifactId>
		<version>${org.mapstruct.version}</version>
		<scope>provided</scope>
	</dependency>
	
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>${org.projectlombok.version}</version>
		<scope>provided</scope>
	<!--	<optional>true</optional>-->
	</dependency>
	
	<dependency>
	    <groupId>com.opencsv</groupId>
	    <artifactId>opencsv</artifactId>
	    <version>5.9</version>
	</dependency>
</dependencies>
```

</details>

### 3-2. in/out 파일 외부 지정&#x20;

배치 파라메터 전달 소스와 동일 하며 코드는 작성 방법은 다음을 참조 하면 됩니다. ([ApplicationRunner 인터페이스 구현체](https://hyomee.gitbook.io/develop/spring-batch/sample/springbatchargs#id-3-1.-applicationrunner))&#x20;

### 3-3. File Read/Write

ItemReader/ItemWriter에서 사용할 수 있게 opencsv를 사용한  유틸리티  클래스를 만듭니다. 다음은  이 클래스에 작성할 기능들 입니다.

* OpenCsvFileUtils(): 생성자로 파일이름을 받아서 파일이름 맴버 변수에 저장&#x20;
* readLine(): 파일을 오픈 하고 파일을 읽는 기능&#x20;
*   initReader(): 파일 객체를 생성하는 메서드로 요구사항 2를 만족하기 위해 ClassLoader 객체를 사용해서 해당 위치에 파일이 없으면 지정한 파일을  File 객체로  생성 합니다.\


    {% code lineNumbers="true" %}
    ```java
    private void initReader() throws Exception {
        ClassLoader classLoader = this
                .getClass()
                .getClassLoader();
        if (file == null) {
            if (classLoader.getResource(fileName) != null) {
                fileName = classLoader
                        .getResource(fileName)
                        .getFile();
            }
            file = new File(fileName);
        }
        if (fileReader == null) fileReader = new FileReader(file);
        if (CSVReader == null) CSVReader = new CSVReader(fileReader);
    }
    ```
    {% endcode %}



    * 2\~4 Line: ClassLoader 객체  생성 \
      java.lang 패키지에 있느 객체로 자바 가상 머신 (JVM)에서 **동적으로 클래스를 로드**하는 역할을 하는 클래스로 여기에서는스프링에서 resource 폴더 아래에 있는 파일에 접근하기 위해서 사용합니다.
    * 5\~12 Line: file 객체가 생성 하는 블럭\
      \- 파일 객체가 생성되어 있지 않으면 new File를 사용해서 파일 객체를 생성 하는 기능 \
      \- 지정한 파일은 ClassPath에 있으면 해당 파일의 물리적 위치를 읽어 온다. (6\~10 Line)\
      &#x20;   \- resource 폴더 파일 : file/in/TB\_DEPLOY.csv\
      &#x20;   \- 물리적 위치 파일 : D:/Code/Spring/abacus/acube-svc-batch/file/in/TB\_DEPLOY.csv
    * 13 Line: 파일 읽기 객체 생성으로 문자셋을 지정 할 수 있다,
    * 14 Line: OpenCsv에서 제공하는 기능으로 CSVReader 객체 생성\
      \- CSVReader:  OpenCSV 라이브러리를 사용하여 Java에서 CSV 파일을 읽기 위한 클래스\

* writeLine(): 파일을 오픈하고 String\[]을 인자로 받아서 CSVWriter 객체를 생성하고 writeNext 메서드를 사용하여 파일에 쓰는 기능
*   initWriter(): File 객체를 생성하고 CSVWriter 객체를 반환 합니다.\


    {% code lineNumbers="true" %}
    ```java
    private void initWriter() throws Exception {
        if (file == null) {
            file = new File(fileName);
            file.createNewFile();
        }
        if (fileWriter == null) fileWriter = new FileWriter(file, true);
        if (CSVWriter == null) CSVWriter = new CSVWriter(fileWriter);
    }
    ```
    {% endcode %}


* closeWriter():  오픈 된 쓰기 파일을 닫는 기능
* closeReader(): 오픈 된 읽기 파일을 닫는 기능

<details>

<summary>OpenCsvFileUtils.java 전체 소스</summary>

{% code lineNumbers="true" %}
```java
import com.opencsv.CSVReader;
import com.opencsv.CSVWriter;
import lombok.extern.slf4j.Slf4j;


import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

@Slf4j
public class OpenCsvFileUtils {

    private String fileName;
    private CSVReader CSVReader;
    private CSVWriter CSVWriter;
    private FileReader fileReader;
    private FileWriter fileWriter;
    private File file;

    public OpenCsvFileUtils(String fileName) {
        this.fileName = fileName;
    }

    public List<String> readLine() {
        try {
            if (CSVReader == null) initReader();
            String[] line = CSVReader.readNext();
            if (line == null) return null;
            return Arrays.stream(line)
                    .collect(Collectors.toList());
        } catch (Exception e) {
            log.error("파일 읽는 중 오류 발생 파일이름 : " + this.fileName);
            return null;
        }
    }

    private void initReader() throws Exception {
        ClassLoader classLoader = this
                .getClass()
                .getClassLoader();
        if (file == null) {
            if (classLoader.getResource(fileName) != null) {
                fileName = classLoader
                        .getResource(fileName)
                        .getFile();
            }
            file = new File(fileName);
        }
        if (fileReader == null) fileReader = new FileReader(file);
        if (CSVReader == null) CSVReader = new CSVReader(fileReader);
    }

    public void writeLine(StringBuffer sb) {
        try {
            if (CSVWriter == null) initWriter();
            String[] lineStr = sb.chars()
                    .mapToObj(c -> String.valueOf((char) c))
                    .toArray(String[]::new);
            CSVWriter.writeNext(lineStr);
        } catch (Exception e) {
            log.error("파일 쓰기중 오류 발생 파일이름 : " + this.fileName);
        }
    }


    public void writeLine(String[] lineStr) {
        try {
            if (CSVWriter == null) initWriter();
            CSVWriter.writeNext(lineStr);
        } catch (Exception e) {
            log.error("파일 쓰기중 오류 발생 파일이름 : " + this.fileName);
        }
    }

    public void writeLine(List<String[]> lineStrs) {
        try {
            if (CSVWriter == null) initWriter();
            for (String[] line: lineStrs) {
                CSVWriter.writeNext(line);
            }
        } catch (Exception e) {
            log.error("E파일 쓰기중 오류 발생 파일이름 : " + this.fileName);
        }
    }
   

    private void initWriter() throws Exception {
        if (file == null) {
            file = new File(fileName);
            file.createNewFile();
        }
        if (fileWriter == null) fileWriter = new FileWriter(file, true);
        if (CSVWriter == null) CSVWriter = new CSVWriter(fileWriter);
    }

    public void closeWriter() {
        try {
            CSVWriter.close();
            fileWriter.close();
        } catch (IOException e) {
            log.error("쓰기 파일 닫기 중 오류.");
        }
    }

    public void closeReader() {
        try {
            CSVReader.close();
            fileReader.close();
        } catch (IOException e) {
            log.error("읽기 파일 닫기 중 오류");
        }
    }
}
```
{% endcode %}

</details>

### 3-3. ChunkSize 외부화

스프링에서 스프링 외부에서 있는 값을 참조하는 방법은 스프링 설정 파일에 선언하고 필요시 잀어 들이는 방법이 있습니다. 이 예제에서는 어플리케이션 구동 시 아큐먼트로 받아서 설정하는 방법과 같이 사용하는 것으로 요구사항을 만족시키고자 합니다.

#### 3-3-1.  설정  파일&#x20;

스프링 설정 파일에 "job.chunksize"로 설정 하고 @Value("${job.chunksize}") 을 사용하여 변수에 값을 주입 하여.사용 합니다.( 선언되 어 있지 않으면 int 형 변수로 0로 초기화 )

{% code title="application.yml" lineNumbers="true" %}
```yaml
job:  
  type: direct # direct 즉시 실헹
  chunksize: 3
```
{% endcode %}

{% code title="JobConfig,java" lineNumbers="true" %}
```java
@Value("${job.chunksize}")
public int chunksize;
```
{% endcode %}

#### 3-3-2.  ApplicationArguments&#x20;

스프링은 Main 메서드에서 받은 값을 ApplicationArguments객체에 저장 하고 있으므로 여기에서 값을 얻어 사용합니다. 다음은 ApplicationArguments객체의 값을 얻기 위해 기능 입니다.

* 아큐먼트로 "--job=case01Job --chunk=1 " 형식으로 받으므로 key-value 구조인 Map으로 변환 하는 가기능 구현&#x20;
* Map에서 chunk 사이즈를 얻어서 Step를 설정하는 기능 구현&#x20;

3-3-2-1. ApplicationUtils.java

ApplicationArgumentsr객체를 받아서 값을 얻어오는 유틸리티 클래스로 ApplicationArguments객체를 Map으로 변환, 값 얻어 오기 기능을 가지고 있습니다.

{% code title="ApplicationUtils.java" lineNumbers="true" %}
```java
public class ApplicationUtils {

    public static Map<String, String> argumentsToMap(ApplicationArguments args) {
        Map<String, String> argMap = new HashMap<>();
        for(String str: args.getSourceArgs()) {
            String[] keyValue = str.split("=");

            if (keyValue.length == 2) {
                String key =  keyValue[0];
                if (key.contains("--")) {
                    key = keyValue[0].substring(2); 
                }

                String value = keyValue[1];
                argMap.put(key, value);
            }
        }
        return argMap;
    }

    public static String getApplicationArgument(ApplicationArguments args, 
                                                String key, 
                                                String defaultValue) {
        Map<String, String> argumentMap = ApplicationUtils.argumentsToMap( args);
        return StringUtils.defaultIfBlank(argumentMap.get(key), defaultValue) ;
    }

    public static int getApplicationArgument(ApplicationArguments args, 
                                             String key, 
                                             int defaultValue) {
        Map<String, String> argumentMap = ApplicationUtils.argumentsToMap( args);
        return Integer.parseInt(StringUtils.defaultIfBlank(argumentMap.get(key), 
                String.valueOf(defaultValue))) ;
    }
}
```
{% endcode %}

* 4 \~ 20 Line: ApplicationArguments 객체를 Map으로 변환하여 반환합니다.\
  \- 예) --job=case01Job --chunk=1 이면 "="를 기준으로 분할하고 "--"를 제외한 나머지로 Map 구성&#x20;
* 22 \~ 35 Line:  ApplicationArguments , 찾고자 하는 키 값(key), 기본값(defaultValue)를 파라메터로 받아서 Map으로 변환 후 찾고자 하는 키 값의 값을 반환 하는 코드로 String, Int로 반환 하는 두 개의 메소드

### 3-4. 배치 집계

JobLauncher.run()  실행 결과로 JobExecution을 반환하며 JobExecution객체는 Job에 대한 실행 정보를 담고 있어 이 객체에서 제공하는 정보로 집계 정보를 확인 할 수 있습니다.

{% code lineNumbers="true" %}
```java
StringBuffer summay = new StringBuffer();

Collection<StepExecution> stepExecutions = jobExecution.getStepExecutions() ;
Iterator steps = stepExecutions.iterator();
while (steps.hasNext()) {
    StepExecution stepExecution = (StepExecution) steps.next();
    summay.append(stepExecution.getSummary() + "\n");
}
```
{% endcode %}

* 3 Line: JobExecution의 Step 정보를 얻습니다. 하나의  Job은 여러개의 Step를 가지고 있아서 Collection 타입 입니다.
* 4 \~ 6 Line: Step 실행 정보를 순회 하면서 Step의 집계 정보를 summay에 저장 합니다.

### **3-5. Job, Step 빈 생성**

Step 생성시 chunk를 사용하고 있으며, 파일을 읽는 부분(reader  선언), 파일을 읽어서 처리하는 부분(processor), 최종 결과 처리 부분(writer)로 구성 되어 있습니다.

```java
new StepBuilder("CASE01_STEP", jobRepository)
        .<TbDeployVO, TbDeployWriteVO>chunk( csize, transactionManager)
        .reader(stepItemReader)
        .processor(stepItemProcessor)
        .writer(stepItemWriter)
        .build();
```

Job생성시 Step의 실행은 start로 시작 할 수 있고 flow로 시작 할 수 있는데 여기서는 flow를 사용하였습니다.

전체 코드는 디음과 같습니다.

{% code title="JobConfig.java" lineNumbers="true" %}
```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class JobConfig {

    private final ApplicationArguments applicationArguments;

    private final StepItemReader stepItemReader;
    private final StepItemProcessor stepItemProcessor;
    private final StepItemWriter stepItemWriter;

    @Value("${job.chunksize}")
    public int chunksize;


    @Bean
    public Step case01Step(JobRepository jobRepository,
                           PlatformTransactionManager transactionManager ) {
        log.debug("####->  case01Step!");
        this.chunksize = ApplicationUtils.getApplicationArgument(applicationArguments,
                                                            BatchConstant.ARGUMEMTS_CHUNK_SIZE,
                                                            this.chunksize);

        return makeStep(jobRepository,
                        transactionManager,
                        this.chunksize);
    }

    private Step makeStep(JobRepository jobRepository,
                          PlatformTransactionManager transactionManager,
                          int csize) {
        return new StepBuilder("CASE01_STEP", jobRepository)
                .<TbDeployVO, TbDeployWriteVO>chunk( csize, transactionManager)
                .reader(stepItemReader)
                .processor(stepItemProcessor)
                .writer(stepItemWriter)
                .startLimit(1)
                .build();
    }

    @Bean
    public Job case01Job(JobRepository jobRepository, Step case01Step ) {
        log.debug("####->  case01Job!");
        return new JobBuilder("CASE01_JOB", jobRepository)
                .flow(case01Step)
                .end()
                .build();
    }
}
```
{% endcode %}

### **3-6. itemReader**&#x20;

ItemReader는 반환 객체만 가지고 있는 인터페이스로 상속 받아 구현을 해야 합니다.

파일을 읽어 오기 위해 opencsv를 이용한 유틸클래스를 사용하고 있으며 Step 처리 후 오픈한 파일을 닫고 Step를 종료하기 위해 StepExecutionListener를 상속 받아 beforeStep, afterStep에 코드를 설명을 코드의 주석을 보면 됩니다.

{% code lineNumbers="true" %}
```java
@Component
@Slf4j
public class StepItemReader implements ItemReader<TbDeployVO>, StepExecutionListener {

    private OpenCsvFileUtils openCsvFileUtils;


    /**
     * 배치 시작시 파라메터로 받은 file의 값으로 File객체를 생성 합니다.
     * - OpenCsvFileUtils 객체 생성 
     * @param stepExecution
     */
    @Override
    public void beforeStep(StepExecution stepExecution) {
        String file = stepExecution.getJobExecution().getJobParameters().getString("file");
        openCsvFileUtils = new OpenCsvFileUtils(file);
        log.debug("#### item -> StepItemReader beforeStep");
    }

    /**
     * 파일을 읽어서 객체에 담아서 반환 
     * - 반환된 객체는 processor의 입력으로 전달 됨 
     * @return
     * @throws Exception
     * @throws UnexpectedInputException
     * @throws ParseException
     * @throws NonTransientResourceException
     */
    @Override
    public TbDeployVO read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
        List<String> line = openCsvFileUtils.readLine();

        if (line == null) {
            return null;
        }
        log.debug("#### item -> StepItemReader read");
        TbDeployVO tbDeployVO = TbDeployVO.builder()
                .empUuid(line.get(0))
                .createDateTime(line.get(1))
                .createUserId(line.get(2))
                .updateDateTime(line.get(3))
                .updateUserId(line.get(4))
                .deployDiv(line.get(5))
                .deployEndDate(line.get(6))
                .deployInDate(line.get(7))
                .deployRsvDate(line.get(8))
                .empName(line.get(9))
                .empNo(line.get(10))
                .projId(line.get(11))
                .build();
        return tbDeployVO;
    }

    /**
     * Step 실행이 완료 되었을 때 실행 하는 것으로 CSVReader, fileReader
     * 객체는 닫는고 완료를 반환합니다.
     * @param stepExecution
     * @return
     */
    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        openCsvFileUtils.closeReader();
        log.debug("#### item -> StepItemReader afterStep");
        return ExitStatus.COMPLETED;
    }
}
```
{% endcode %}

### **3-9. itemPrcessor**

ItemReader에서 반환된 값을 전달 받아 업무 처리를 하고 결과를 반환 하면 ItemWriter로 전달 됩니다. 이때 ItemWriter로 전달 되는 객체의 크기는 Step 선언시 정의한 chunkSize 만큼 Collection에 감사 반환 합니다.

{% code title="StepItemProcessor.java" lineNumbers="true" %}
```java
/**
 * ItemProcessor 상속 받아 process 구현 
 * - ItemProcessor 인터페이스는 input, output으로 정의 되어 있으며
 * - output은 chucksize로 선언된 크기 만큼 Collection객체에 담아서 Writer로 전달 됨
 * - 비지니스 구현은 proecess에 구현한다.
 * StepExecutionListener 인터페이스를 상속 받아 process 처리 이전, 이후 작업이 필요
 * 시 코드를 작성 할 수 있다.
 */
@Component
@Slf4j
public class StepItemProcessor implements ItemProcessor<TbDeployVO, TbDeployWriteVO>,
        StepExecutionListener {


    /**
     * process 이전 작업 처리 
     * @param stepExecution
     */
    @Override
    public void beforeStep(StepExecution stepExecution) {
        log.debug("#### item -> StepItemProcessor beforeStep.");
    }

    /**
     * 실제 업무 처리 하는 곳
     * - input : ItemReader에서 반환 된 값을 전달 받음
     * - output : ItemWriter로 보내기 위해 반환 가는 값으로 chunksize 만큼 
     *            모아서 Collection<> 객체로 ItemWriter에 전달 
     * @param tbDeployVO
     * @return
     * @throws Exception
     */
    @Override
    public TbDeployWriteVO process(TbDeployVO tbDeployVO) throws Exception {
        log.debug("#### item -> StepItemProcessor process ");
        if (tbDeployVO == null) return null;
        // mapStruct를 사용해 변환 처리 
        return Case01Mapper.INSTANCE.tbDeployVOToTbDeployWriteVO(tbDeployVO);
    }

    /**
     * proceee 처리 이후 작업 
     * @param stepExecution
     * @return
     */
    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        log.debug("#### item -> StepItemProcessor afterStep ");
        return ExitStatus.COMPLETED;
    }
}
```
{% endcode %}

### **3-10. itemPrcessor**

opencsv를 사용한 File 적재 로직으로 db에 넣을 경우 이곳에 작성을 합니다.

{% code title="StepItemWriter.java" lineNumbers="true" %}
```java
/**
 *
 */
@Component
@Slf4j
public class StepItemWriter implements ItemWriter<TbDeployWriteVO>, StepExecutionListener {

    /**
     * opencsv 라이브러리를 사용한 Utility
     */
    private OpenCsvFileUtils openCsvFileUtils;

    /**
     * 파일에 적재 하기 위해 File 관련 객체 생성 
     * out file은 배치 실행 시 전달 받음 
     *  예) --fileOut=D:/Code/Spring/abacus/acube-svc-batch/file/out/TB_DEPLOY_OUT.csv
     * @param stepExecution
     */
    @Override
    public void beforeStep(StepExecution stepExecution) {
        String file = stepExecution.getJobExecution().getJobParameters().getString("fileOut");
        openCsvFileUtils = new OpenCsvFileUtils(file);
        log.debug("#### item -> StepItemWriter beforeStep ");
    }

    /**
     * 파일애 쓰기 기능
     * - input : ChunkSize 크기의 Collection<>으로 전달 받음 
     * @param tbDeployWriteVOs
     * @throws Exception
     */
    @Override
    public void write(Chunk<? extends TbDeployWriteVO> tbDeployWriteVOs) throws Exception {
        log.debug("#### item -> StepItemWriter write ");
        if (tbDeployWriteVOs == null) return;
        // Collection 객체 이므로 반복 하면서 스티링 배열로 변환 후 파일에 추가 
        for (TbDeployWriteVO tbDeployWriteVO : tbDeployWriteVOs) {
            log.debug("Wrote line " + tbDeployWriteVO.toString());
            if (tbDeployWriteVO == null) continue;
            String[] line = toStringArray(tbDeployWriteVO);
            openCsvFileUtils.writeLine(line);
        }

    }

    /**
     * write 이루 처리로 종료 반환
     * @param stepExecution
     * @return
     */
    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        openCsvFileUtils.closeWriter();
        log.debug("#### item -> StepItemWriter afterStep ");
        return ExitStatus.COMPLETED;
    }

    /**
     * 파일에 적재 하기 위해 배열로 반환 
     * @param tbDeployWriteVO
     * @return
     */
    private String[] toStringArray(TbDeployWriteVO tbDeployWriteVO) {
        String[] line = new String[] { tbDeployWriteVO.getEmpUuid(),
                tbDeployWriteVO.getDeployDiv(),
                tbDeployWriteVO.getDeployEndDate(),
                tbDeployWriteVO.getDeployInDate(),
                tbDeployWriteVO.getDeployRsvDate(),
                tbDeployWriteVO.getEmpName(),
                tbDeployWriteVO.getEmpNo(),
                tbDeployWriteVO.getProjId() };
        return line;
    }

}
```
{% endcode %}
