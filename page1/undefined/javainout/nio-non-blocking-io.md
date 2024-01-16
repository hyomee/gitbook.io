# NIO(Non-blocking IO)

JAVA NIO(New IO)는 JAVA에서 IO를 처리하는 패키지 중 하나로 기존 IO 패키지를 개선하기 위해 나온 패키지로 비동기 처리 가능하게 해준다. 즉 하나의 스레드로 다수의 IO를 처리 할 수 있으므로 대용량 데이터 처리가 가능합니다.

<figure><img src="../../../.gitbook/assets/자바NIO_이미지 (1).jpg" alt=""><figcaption></figcaption></figure>

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

## 2. 채널

* 채널을 읽고 쓸 수 있습니다. 스트림은 일반적으로 단방향(읽기 또는 쓰기)입니다.
* 채널은 비동기적으로 읽고 쓸 수 있습니다.
* 채널은 항상 버퍼를 읽거나 씁니다.

<figure><img src="../../../.gitbook/assets/자바NIO_이미지 (1).jpg" alt=""><figcaption></figcaption></figure>

### 2-1. Channel 읽기

* FileChannel : 파일에서 데이터를 읽습니다.
* DatagramChannel : UDP를 통해 네트워크를 통해 데이터를 읽고 쓸 수 있습니다
* SocketChannel : TCP를 통해 네트워크를 통해 데이터를 읽고 쓸 수 있습니다
* ServerSocketChannel : 소겟 서버를 만들어 TCP를 수신 할 수 있습니다.

**소스에서 아래 부분이 채널에 관계된 부분입니다.**

```java
RandomAccessFile aFile = new RandomAccessFile(fileName, rwMode);
FileChannel inChannel = aFile.getChannel();
```

## 3.  Buffer

데이터는 채널을 버퍼로, 버퍼에서 채널로 쓰는데  데이터를 쓸 수 있는 메모리 블록이 Buffer 입니다. Buffer는 4단계 프로세스를   통해  데이터를 읽고 씁니다.

<figure><img src="../../../.gitbook/assets/Nio_버퍼.jpg" alt=""><figcaption><p>버퍼 쓰기/읽기</p></figcaption></figure>

1. 버퍼에 데이터 쓰기
2. buffer.flip()
   * 데이터를 읽기 위해서는 buffer을 쓰기 모드에서 읽기 모드 변경해야 하는데 이것을 수헹하는 메소드 입니다.
3. 버퍼에서 데이터 읽기
4. buffer.clear() 또는 buffer.compact()
   * 모든 데이터를 읽은 후에는 버퍼를 지워야 합니다.
   * buffer.clear() : 모든 데이터를 읽은 후에는 버퍼를 지워야 합니다
   * buffer.compact() : 이미 읽은 데이터 만 지웁니다.
   * 읽지 않은 데이터는 시작 부분으로 이동하고. 데이터는 이제 읽지 않은 데이터 다음에 버퍼에 기록됩니다.

**소스에서 아래 부분이 buffer에 관계된 부분입니다.**

```java
// byte 단위로 메모리 할당
ByteBuffer buf =  ByteBuffer.allocate(allocate) ;

// 할당된 크기 만큼 버퍼에 읽어 드
int bytesRead = inChannel.read(buf)
// 버퍼를 읽기 준비 상태로 변경
buf.flip();

// 버퍼에 있는 내용 읽기 
(char) buf.get()
CharBuffer charBuffer = charset.decode(buf);

// 버퍼에 다시 쓰기 위해서 쓰기 모드로 변경 
buf.clear();
```

### 3-1. 버퍼 용량, 위치 및 한계

버퍼는 데이터를 쓸 수 있는 매모리 블럭으로 NIO Buffer 객체와 래핑되어 있어 메모리 블럭을 쉽게 작업할 수 있도록 하는 메서드로 다음과 같은 속성이 있습니다.

* capacity : 한번에 쓰기 위한 메모리의 크기로 bytes, longs, chars 등만 사용할 수 있어고 데이터를  쓰기전에 비워야 합니다.
* position : 메모리 블럭에 읽고 쓰는 위치를 의미하며 처음 위치는 0입니다. 쓰기 모드에서 읽기모드로 변경이 되면 0으로 재 설정 됩니다.
* limit : 읽고 쑈기 위한 최대 값 입니다.&#x20;

### 3-2.  Buffer Types

* ByteBuffer
* MappedByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer

### 3-3.  버퍼 할당&#x20;

개체를 가져오려면 먼저 개체를 할당해야 합니다.   48 Byte를 할당하며면 다음과 같이 작성하면 됩니다.

```java
 ByteBuffer buf =  ByteBuffer.allocate(48) ;
 
 // 1024 문자를 할당 
 CharBuffer buf = CharBuffer.allocate(1024)
```

### 3-4. 버퍼에 쓰기

Channel에서 buffer로 쓰기 위해서는 다음과 같은 방법아 있습니다.

*   생성 후 채널에 있는 데이터를 읽으면 버퍼에 쓰게 됩니다.  &#x20;

    ```java
     int bytesRead = inChannel.read(buf);
    ```
*   직접 버퍼에 쓰기

    ```java
    buf.put(127);  
    ```

### 3-5. 모드 전환 ( flip() )

일기/쓰기 모드 전환은 통해서 데이터를 버퍼에 쓴 후 읽기를 하기 위해서는 읽기 모드로 변경 해야 하고 읽은 후에 다시 쓰기를 위해서는 쓰기 모드로 변경을 헤야 합니다.

```java
buf.flip(); 
```

### 3-6. 버퍼 읽기

버퍼에서 데이터를 읽기 위해서는 다음과 같은 두 가지 방법이 있습니다.

*   get() 메서드를 사용하여 데이터 읽기   &#x20;

    ```java
    byte aByte = buf.get();
    ```
*   채널로 버퍼의 내용을 읽은 후 체널에 쓰기

    <pre class="language-java"><code class="lang-java"><strong>int bytesWritten = inChannel.write(buf);
    </strong></code></pre>

### 3-7. 버퍼 읽기 ( rewind() )

position을 0으로 설정하여 다시 읽을 수 있게 합니다.

### 3-8. 버퍼 지우기 ( clear() and compact())

버퍼에 쓰기 위해서는 버퍼의 내용을 지워야 하는데 사용되는 메서드 입니다.

* buffer.clear() : 모든 데이터를 읽은 후에는 버퍼를 지워야 합니다
* buffer.compact() : 이미 읽은 데이터 만 지웁니다. 즉 남아 있는 데이터를 덮어 쓰지는 않습니다.

### 3-9. mark() 및 reset()

* mark() : 현재 위치를 표시한다.
* reset() : 표시한 마커 위치 지운다.

```java
buffer.mark();

call buffer.get() // 읽어 들임 

buffer.reset();  // set position back to mark.  
```

## 4. Selector

스레드는 운영 채제애 비용을 먾이 들이는 작업으로 하나의 스레드에서 여러 채널을 관리하며 비용 소모가 적게 들게 되니다. 이런 경우 Selector에 여러개의 스레드를 묶어서 하나의 스레드로 처리하게 됩니다.

<figure><img src="../../../.gitbook/assets/image.png" alt="" width="306"><figcaption></figcaption></figure>



### 4-1. selector 셍성&#x20;







참고 : [https://jenkov.com/tutorials/java-nio/index.html](https://jenkov.com/tutorials/java-nio/index.html)
