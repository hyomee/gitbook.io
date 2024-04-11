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

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* Step 지정시 Chunk의 수을 지정 하면 ItemReader이 지정한 수 만큼 읽어서 Processor에 전달 합니다.
* Processor에서는 Chunk에 지정한 수 만큼 처리 하고 Writer로 전달 합니다.
* 즉. 전체가 3이고 Chunk를 2로 지정 하면 reader가 2개 읽어 Processor로 전달하고 Processor는 2개를 처리 하고 writer로 Array 형태 (Chunk)로 전달 하여 처리 하고 다시 Reader를 읽어 Step을 처리 합니다.&#x20;
* 요구사항을 만족하기 위한 기능 처리&#x20;
  1. [classpath에 지정한 파일이 없는 경우 지정한 파일을 읽는다](chunk.md#id-3-2.-in-out).
     * opencsv 의존성을 pom.xml에 선언한다.
     * 파일을 읽을 때 ClassLoader 객체르 사용해서 파일을 검사하여 있으면 해당 위치에서 파일을 읽고 그렇지 않으면 지정한 위치에서 읽게 한다.
  2. [읽을 파일과 쓰기 파일은 외부에서 받아서 차리해야 한다.](chunk.md#id-3-3.-file-read-write)
  3. chunk의 크기는 외부에서 받아서 동적으로 처리해아 한다.
  4. 배치가 수행 되고 나면 총 커밋 수, 오류 수등 집계를 제공해야 한다.

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
      \- 지정한 파일은 ClassPath에 있으먄 해당 파일의 물리적 위치를 읽어 온다. (6\~10 Line)\
      &#x20;   \- resource 폴더 파일 : file/in/TB\_DEPLOY.csv\
      &#x20;   \- 물리적 위치 파일 : D:/Code/Spring/abacus/acube-svc-batch/file/in/TB\_DEPLOY.csv
    * 13 Line: 파일 읽기 객체 생성으로 문자셋을 지정 할 수 있다,
    * 14 Line: OpenCsv에서 제공하는 기능으로 CSVReader 객체 생성\
      \- CSVReader:  OpenCSV 라이브러리를 사용하여 Java에서 CSV 파일을 읽기 위한 클래스
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

