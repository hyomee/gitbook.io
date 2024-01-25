# 라인단위읽기(Stream 사용)

자바를 이용해서 코드 작성 시 한 줄씩 파일을 읽는 방법은 다음과 같다.

## 1. NIO - Stream 사용

BufferedReader lines()과 Java 8 의 Stream을 이용한 방법으로 Stream을 사용하면 map, count, filter 등을 통해서 변환 할 수 있습니다.

{% code lineNumbers="true" %}
```java
public class BufferedReaderLinesStream {
  public static void main(String[] args) {

    try (ReadFileLines readFileLines = new ReadFileLines()) {
      readFileLines.readLines();
    } catch (Exception e) {
      System.out.println(e.getMessage());
    }

  }

}

// 자동 헤제 확인을 위한 AutoCloseable 상속 
class ReadFileLines  implements AutoCloseable {
  @Override
  public void close()  {
    System.out.println("close");
  }
  public void readLines() throws IOException {
    // 원본 파일
    Path sourceFile = Paths.get("D:\\Code\\niodata1.txt");

    // try - with - resource를 통한 자동 해제 
    try (
      // newBufferedReader로 버퍼 할당 
      BufferedReader bufferedReader = Files.newBufferedReader(sourceFile, 
            Charset.forName("UTF-8"));
    ) {
      
      // bufferedReader.lines().forEach(System.out::println);
      // System.out.println("라인수 : " + bufferedReader.lines().count());
      int maxLine = bufferedReader.lines()
              .mapToInt(String::length)
              .max()
              .getAsInt();
      System.out.println(String.format("가장긴라인 : %s" , maxLine));
    }
  }
}
```
{% endcode %}

## 2. FileReader + BufferedReader

java.io.BufferedReader 클래스는 텍스트 파일에서 데이터를 읽을 수 있는read()메소드의 4 가지 버전을 제공합니다.

1. read() - 단일 문자를 읽으려면 이 메서드가 int를 반환하므로 이를 문자로 캐스팅해야 합니다.
2. read(char\[] cbuf) - 문자를 배열로 읽습니다. 이 메서드는 일부 입력을 사용할 수 있거나, I/O 오류가 발생하거나, 스트림의 끝에 도달할 때까지 차단됩니다. 이 메서드는 읽은 문자 수를 반환하거나 파일 끝에 도달한 경우 -1을 반환합니다. 이 메서드는 Reader 클래스에서 제공됩니다.
3. read(CharBuffer cbuffer) - CharBuffer로 문자를 읽으려면 배열 대신 CharBuffer 객체로 문자를 읽는다는 점을 제외하고는 이전 방법과 유사합니다. 또한 이 메서드는 읽은 총 문자 수를 반환하거나 파일 끝에 도달한 경우 -1을 반환합니다. 이 메소드는 java.io.Reader 클래스에도 속합니다.
4. read(char\[] cbuf, int off, int len) - 문자를 배열로 읽지만 파일에서 읽은 문자를 저장할 위치를 제어할 수 있습니다. offset , 즉 시작할 인덱스와 길이, 저장할 문자 수를 지정할 수 있습니다.
5. readLine() - 텍스트 줄을 읽습니다. 이 방법을 사용하여 [Java에서 파일을 한 줄씩 읽을](https://javarevisited.blogspot.com/2012/07/read-file-line-by-line-java-example-scanner.html) 수 있습니다. 라인은 줄 바꿈('\n'), 캐리지 리턴('\r') 또는 캐리지 리턴 바로 뒤에 줄 바꿈이 오는 것 중 하나에 의해 종료되는 것으로 간주됩니다. 이 메서드는 줄 종료 문자를 포함하지 않고 줄의 내용을 포함하는 문자열을 반환하거나, 스트림의 끝에 도달한 경우 null을 반환합니다. 많은 Java 개발자는이 메소드에만 BufferedReader 클래스를 사용합니다.

```java
  public void readFileReaderLines() throws IOException {
   
    try (
            FileReader textFileReader = new FileReader("D:\\Code\\niodata.txt", Charset.forName("UTF-8"));
            BufferedReader bufferedReader = new BufferedReader(textFileReader );

    ) {

      String line = bufferedReader.readLine();
      while (line != null) {
        System.out.println(line);
        line = bufferedReader.readLine();
      }
    }
  }
```

try-with-resource 구문을 사용했기 때문에 BufferedReader의 close() 메서드를 수동으로 호출할 필요가 없으며 Java에서 자동으로 호출됩니다. catch 절은 close() 메서드에서 throw된 IOException을 catch하기 위한 것입니다.\


## 3. InputStream ( JAVA IO )

Java에서 InputStream사용 할 때는 다음 단계를 따라야 합니다.

1. FileInputStream을 열어 파일의 내용을 InputStream으로 읽습니다.
2. 바이트를 문자로 읽기 위해 문자 인코딩으로 InputStreamReader를 만듭니다.
3. 파일을 한 줄씩 읽습니다.
4. 스트링으로 변환 합니다.

try-with-resources문을 사용하여 작성 한 예제 입니다.

```java
String readFile =  "D:\\Code\\niodata.txt";

try (FileInputStream fileInputStream = new FileInputStream(readFile);
     BufferedReader bufferedReader = new BufferedReader(
        new InputStreamReader(fileInputStream, StandardCharsets.UTF_8)
     )
    ) {
  String str = null;
  while ( (str = bufferedReader.readLine() ) != null) {
      System.out.println(str);
  }

} catch (FileNotFoundException e) {
    throw new RuntimeException(e);
} catch (IOException e) {
    throw new RuntimeException(e);
}
```

