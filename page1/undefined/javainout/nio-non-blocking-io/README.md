# NIO(Non-blocking IO)

JAVA NIO(New IO)는 JAVA에서 IO를 처리하는 패키지 중 하나로 기존 IO 패키지를 개선하기 위해 나온 패키지로 비동기 처리 가능하게 해준다. 즉 하나의 스레드로 다수의 IO를 처리 할 수 있으므로 대용량 데이터 처리가 가능합니다.

<figure><img src="../../../../.gitbook/assets/자바NIO_이미지 (1).jpg" alt=""><figcaption></figcaption></figure>

* **Buffer**: 커널에 관리되는 시스템 메모리를 직접 사용할 수 있는 클래스입니다.
* **Channel**: 입출력 클래스로, 읽기/쓰기를 하나씩 할 수 있는 단방향, 둘 다 가능한 양방향을 지원합니다.
* **Selector**: 하나의 스레드로 여러 채널을 관리하고 처리할 수 있는 클래스입니다

## 1. 간단한 예제

readUTF8.txt 파일을읽기 모드로 48Byte 씩 버퍼로 읽어 들이기 위해서 charReadFileNioByte() 메서드를 호출하는 코드 입니다.&#x20;

**NIO의 buffer, channel을 살펴보기위해 RandomAccessFile, FileChannel 클래스를 사용하여 파일 입력을 처리하도록 합니다. 다음은 RandomAccessFile, FileChannel클래스의 간단한 설명입니다.**

* <mark style="color:purple;">**RandomAccessFile 클래스 :**</mark> &#x20;
  * &#x20;<mark style="color:purple;">파일의 임의 위치에서 읽기/쓰기 작업을 지원하여 순차적 파일 입출력 보다 유연한 입출력을 제공합니다.</mark>
  * &#x20;<mark style="color:orange;">그러나 비동기적 으로 대용량 파일로 작업할 때 적합하지 않습니다.</mark>
* <mark style="color:purple;">**FileChannel 클래스 :**</mark>&#x20;
  * <mark style="color:purple;">파일에 대한 순차적 및 비순차적(임의) 액세스를 모두 제공 합니다.</mark>&#x20;
  * <mark style="color:purple;">입출력 작업이 블로킹되지 않는 비동기 방식으로 동작하는 클래스이며</mark> <mark style="color:purple;"></mark><mark style="color:purple;">`ByteBuffer`</mark><mark style="color:purple;">와 함께 사용되며, 파일 입출력을 위한 다양한 메소드를 사용할 수 있습니다.</mark>
  * <mark style="color:purple;">읽기 및 쓰기 작업의 성능 향상을 위해 파일 영역을 메모리에 직접 매핑하는 메모리 매핑을 지원합니다.</mark>
  * <mark style="color:purple;">여러 프로세스가 제어된 방식으로 파일에 액세스할 수 있도록 하는 잠금 메커니즘을 제공하여 한 번에 하나의 프로세스만 파일을 수정할 수 있도록 합니다.</mark>
  * <mark style="color:purple;">transfer() 메서드를 사용하여 채널 간에 직접 데이터를 전송할 수 있으므로 데이터를 버퍼로 읽은 다음 다시 쓸 필요가 없습니다.</mark>

### 2-1. 호출 방법

```java
JavaNIO.charReadFileNioByte("D:\\문서\\교육\\txt\\readUTF8.txt",
                "r", 
                48);
```

{% code title="* 실행 결과 : 한글 째짐" %}
```
Read :: 48 => J
Read :: 48 => a
Read :: 48 => v
Read :: 48 => a
Read :: 48 => ￬
....
```
{% endcode %}

### 2-2. 소스&#x20;

<pre class="language-java" data-title="JavaNIO.java" data-line-numbers><code class="lang-java"><strong>public static void CharReadFileNioByte(String fileName,
</strong>                                         String rwMode,
                                         int allocate ) {

  rwMode = rwMode == null || rwMode.length() == 0 ? "r" : rwMode;

  RandomAccessFile aFile = null;
  FileChannel inChannel = null;

  try {
    aFile = new RandomAccessFile(fileName, rwMode);
    inChannel = aFile.getChannel();

    // byte 단위로 메모리 할당
    ByteBuffer buf =  ByteBuffer.allocate(allocate) ;
    int bytesRead = inChannel.read(buf);
    while (bytesRead != -1) {
      buf.flip();
      while(buf.hasRemaining()){
        System.out.println(String.format("Read :: %s => %s", bytesRead, (char) buf.get()));
      }
      buf.clear();
      bytesRead = inChannel.read(buf);
    }

  } catch (IOException e) {
    throw new RuntimeException(e);
  } finally {
    try {
      if (inChannel != null) inChannel.close();
      if (aFile != null) aFile.close();
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
  }
}
</code></pre>

* 11 line : RandomAccessFile 클래스로 파일에 접근합니다.
* 12 line : 파일에F대해서   FileChannel을 생성 합니다.&#x20;
* 15 line : 읽어 들인 버퍼 사이즈 설정
* 16 - 24 line : 채널에서 버퍼 사이즈 만큼 읽어서 버퍼에 데이터가 있으면 콘솔에 표시 하고 다음 버퍼에 다시 읽는 작업을 반복 합니다.
* 30 - 31 line : 채널과 파일을 닫는다.

### 2-3. 한글 깨짐 해결&#x20;

2-2 코드를 실행 하면 한글이 정상 출력 되지  않는 문제가 있습니다, 이것은 파일이 생성되었을 떄 문자셋과 자바에서 읽어들였을떄 문자셋을 해석 하지 않아서 발생한 문제 입니다. 문자셋을 설정 하도록 다음을 수정 하면 됩니다.

수정된 부분은 다음과 같습니다.

{% code lineNumbers="true" %}
```java
Charset charset = pCharSet == null || pCharSet.length() == 0  ?
    Charset.forName("UTF-8"):
    Charset.forName(pCharSet);
    
while (bytesRead != -1) {
    buf.flip();
    CharBuffer charBuffer = charset.decode(buf);
    System.out.println(String.format("Read :: %s => %s", bytesRead, charBuffer));
    buf.clear();
    bytesRead = inChannel.read(buf);
}
```
{% endcode %}

* 1  line : 문자셋 지정을 위해 파라메터로 문자셋을 받아 null 이면 기본으로 UTF-8로 지정합니다.
* 7 line : 버퍼에서 있는 데이터를 지정한 문자셋으로 encode 합니다.

<details>

<summary>문자셋을 받아 처리하도록 수정 한 전체 소스</summary>

{% code lineNumbers="true" %}
```java
public static void CharReadFileNio(String fileName,
                                     String rwMode,
                                     int allocate,
                                     String pCharSet) {

  rwMode = rwMode == null || rwMode.length() == 0 ? "r" : rwMode;
  Charset charset = pCharSet == null || pCharSet.length() == 0  ?
          Charset.forName("UTF-8"):
          Charset.forName(pCharSet);


  RandomAccessFile aFile = null;
  FileChannel inChannel = null;

  try {
    aFile = new RandomAccessFile(fileName, rwMode);
    inChannel = aFile.getChannel();

    // byte 단위로 메모리 할당
    ByteBuffer buf =  ByteBuffer.allocate(allocate) ;
    int bytesRead = inChannel.read(buf);
    while (bytesRead != -1) {
      buf.flip();
      CharBuffer charBuffer = charset.decode(buf);
      System.out.println(String.format("Read :: %s => %s", bytesRead, charBuffer));
      buf.clear();
      bytesRead = inChannel.read(buf);
    }

  } catch (IOException e) {
    throw new RuntimeException(e);
  } finally {
    try {
      if (inChannel != null) inChannel.close();
      if (aFile != null) aFile.close();
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
  }
}
```
{% endcode %}

</details>







참고 : [https://jenkov.com/tutorials/java-nio/index.html](https://jenkov.com/tutorials/java-nio/index.html)
