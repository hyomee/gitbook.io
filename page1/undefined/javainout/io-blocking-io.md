# IO(Blocking IO)

Java IO 패키지는Java에서 입출력을 다루기 위한 패키지로 하나의 Thread가 IO 작업에 의존적이기 때문에 하나의 Thread로 하나의 IO를 처리하는 **Blocking 방식  구현**으로 InputStream, OutputStream, Reader, Writer 클래스를 사용합니다.&#x20;

* **바이트 단위** : IputStream, OutputStream\
  \- 이미지, 비디오 및 직렬화 된 객체와 같은 이진 데이터\
  \- InputStream.read() : 바이트 스트림의 원시 내용에 해당하는 0에서 255 사이의 바이트 값을 반환
* **문자 단위** : Reader,  Writer \
  \-  문자 스트림이므로 문자 데이터 \
  \- Reader.read() : 0에서 65357 사이의 문자 값을 반환&#x20;

&#x20;<mark style="color:red;">Java NIO와 달리 기존 IO 패키지는 JVM 내부 버퍼로 복사 시 발생하는 CPU 연산, GC 관리, IO 요청에 대한 스레드 블록이 발생하게 되는 현상 때문에 효율이 좋지 못한 점이 있습니다</mark>

## 1. Java IO 패키지의 핵심 용어

* **InputStream**: 바이트 단위로 입력을 처리하는 클래스입니다.
* **OutputStream**: 바이트 단위로 출력을 처리하는 클래스입니다.

<figure><img src="../../../.gitbook/assets/자바IO객체 (1).jpg" alt=""><figcaption><p>자바 IO 팩키지</p></figcaption></figure>

* **Reader**: 문자 단위로 입력을 처리하는 클래스입니다.
* **Writer**: 문자 단위로 출력을 처리하는 클래스입니다

## 2. 파일 읽기

파일을 읽기 위해서는 다음 순서로 코드를 작성해야 합니다.

1. **파일 객체 생성**: \
   \- `File file = new File("filename.txt");`
2. **파일 입력 스트림 생성**: \
   \- `FileInputStream inputStream = new FileInputStream(file);`
3. **버퍼 생성**: \
   \- `byte[] buffer = new byte[1024];`
4. **입력 스트림으로부터 데이터를 버퍼에 읽어옴**: \
   \- `inputStream.read(buffer);`
5. **버퍼에서 읽어온 데이터를 문자열로 변환**: \
   \- `String data = new String(buffer);`
6. **입력 스트림 닫기**: \
   \-`inputStream.close();`

### 2-1. Byte 단위로 읽기

{% code title="예제 1." lineNumbers="true" %}
```java
FileInputStream fileInputStream = null;
try {
  fileInputStream =  new FileInputStream(fileName) ;
  int readChar = 0;
  while ((readChar = fileInputStream.read()) != -1) {
    System.out.println((char)readChar);
  }
} catch (IOException e) {
  System.out.println("파일을 확인해 주세요");
  throw new RuntimeException(e);
} finally {
  if (fileInputStream != null  ) {
    try {
      fileInputStream.close();
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
  }
}
```
{% endcode %}

* 3 line : 파일 객체 생성 합니다.
  * FileInputStream 생성자의 인수로 파일을 넘겨주면 내부에서 File를 객체를 생성합니다.&#x20;
  *   파일 읽기의 1, 2번 사항 입니다.

      ```java
      public FileInputStream(String name) 
                      throws FileNotFoundException {
          this(name != null ? new File(name) : null);
      }
      ```
* 4 line : read() 메서드는 Data를 한 Byte씩 읽어서 값이 없으면 -1로 돌려 주는 메서드로 파일을 읽어 마지막 인지 체크하기 위해 선언한 변수로 초기값 0으로 설정합니다.
* 5 line : Data를 한 Byte씩 읽어서 없으면 while문을 벗어납니다.
* 6 line : 읽은 문자는 Char로 변환 해서 출력합니다.

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

* 8 - 11 line : File에 관한 예외 처리 로직
* 12 - 17 line : File을 열어서 작업을 했으므로 열려 있는 파일을 닫아야 합니다.

#### 2-1-1. 예제 1의 문제점

한글 파일인 경우 <mark style="color:red;">**한글이 깨지는 문제ㄷ**</mark> 입니다. 해당 문제를 해결하기 위해서는 문자셋을 지정해야 하로&#x20;

<pre class="language-java" data-line-numbers data-full-width="false"><code class="lang-java">public static void CharReadFile(String fileName, 
                                String charsetName) {
    
    InputStreamReader inputStreamReader = null;
    
    try {
<strong>      inputStreamReader = new InputStreamReader(
</strong>               new FileInputStream(fileName),
               charsetName);
               
      int readChar = 0;
      while ((readChar = inputStreamReader.read()) != -1) {
        System.out.println((char)readChar);
      }
      
    } catch (UnsupportedEncodingException e) {
      System.out.println("엔코딩에 문제가 있습니다.");
      throw new RuntimeException(e);
    } catch (IOException e) {
      System.out.println("파일을 확인해 주세요");
      throw new RuntimeException(e);
    } finally {
      if (inputStreamReader != null  ) {
        try {
          inputStreamReader.close();
        } catch (IOException e) {
          throw new RuntimeException(e);
        }
      }
    }
    
  }
</code></pre>

* 6 Line : 파일 객체 생성
  * FileInputStream 생성자의 인수로 파일을 넘겨주면 내부에서 File를 객체를 생성합니다.&#x20;
  *   파일 읽기의 1, 2번 사항 입니다.

      ```java
      public FileInputStream(String name) 
                      throws FileNotFoundException {
          this(name != null ? new File(name) : null);
      }
      ```
*
