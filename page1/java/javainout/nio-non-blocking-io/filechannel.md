# FileChannel

자바 Nio FileChannel은 파일을 읽고 쓰기 위해 파일에 연결된 채널로 표준 Java IO API를 대신하여 사용 할 수 있습니다. **FileChannel은 항상 blocking mode** 입니다.

## 1. 파일 읽기

InputStream, OutputStream , RandomAccessFile을 통해서 파일을 읽고 FileChannel을 통해서 가져 옵니다.

{% code lineNumbers="true" %}
```java
RandomAccessFile aFile  = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
```
{% endcode %}

## 2. FileChannel 읽기

채널에 있는 파일을 buffer로 읽어 오기 위해서 사이즈할당(allocate)하고 read로 읽어 옵니다.

```java
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
```

바이트가 . -1이 반환되면 파일 끝에 도달합니다.

한글을 출력 할 떄 byte 단위로 할당 하면 문제가 발생 합니다.  해당 문제를 해결 하기 위해서는 라인단위로 읽어서 변환 하거나 전체 채널의 크기 만큼 읽어서 변환 하는 방법이 있습니다.

여기서는 채널의 사이즈 만큼 읽어서 변환하는 예제를 작성 합니다.&#x20;

*   ByteBuffer.allocate 메서드를 사용해서 읽을 버퍼의 크기를 설정 합니다. 한글은 바이트 계산이 힘들어서 채널의 전체 크기 만큼 할당 하고 읽도록 합니다.\


    ```java
    ByteBuffer buf = ByteBuffer.allocate((int) inChannel.size());
    int bytesRead = inChannel.read(buf);
    ```


*   14 line : 한글(UTF-8)로 변환 합니다.\


    ```java
    String koreanText = StandardCharsets.UTF_8.decode(buf).toString();
    ```


*   15 line \~ 18 line : 라인 단위로 처리 하기 위해 시스템 프로퍼티의 라인 개행(line.separator)으로 분리하고 한 라인씩 출력 합니다.\


    ```java
    // 개행"\\r?\\n"
    String[] lines = koreanText.split(System.getProperty("line.separator"));  
    for (String line : lines) {
        System.out.println(String.format("Read :: %s => %s", bytesRead, koreanText));
    } 
    ```

<details>

<summary>전체 소스 </summary>

{% code lineNumbers="true" %}
```java
public static void main(String[] args) {
    RandomAccessFile aFile = null;
    FileChannel inChannel = null;
    try {
        aFile = new RandomAccessFile("D:\\Code\\niodata.txt", "rw");
        inChannel = aFile.getChannel();

        ByteBuffer buf = ByteBuffer.allocate((int) inChannel.size());
        int bytesRead = inChannel.read(buf);

        while (bytesRead != -1) {
            buf.flip();
            // 한글 변환 
            String koreanText = StandardCharsets.UTF_8.decode(buf).toString();
            String[] lines = koreanText.split(System.getProperty("line.separator"));
            for (String line : lines) {
                System.out.println(String.format("Read :: %s => %s", bytesRead, koreanText));
            }

            buf.clear();
            bytesRead = inChannel.read(buf);
        }
    } catch (IOException e) {
        throw new RuntimeException(e);
    } finally {

        try {
            if ( aFile != null  ) aFile.close();
            if ( inChannel != null  ) inChannel.close();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
} 
```
{% endcode %}

</details>

### 2-1  힙 메모리 사용

allocate는   System Memory를 사용 합니다.&#x20;

```java
ByteBuffer buf = ByteBuffer.allocate((int) inChannel.size());
```

만약 자바의 힙메모리를 사용하기 위해서는 allocateDirect 메서드를  사용하여음과 같이 변경 하면 됩니다.

```java
ByteBuffer buf = ByteBuffer.allocateDirect((int) inChannel.size());
```

## 3. FileChannel 쓰기

파일 쓰기는 FileChannel.write()를 사용해서 작성 합니다. 읽기와 같이 파일을 쓰기 모드로 설정 하고 버퍼를 통해 채널로 쓰기를 합니다.

아래  전체 소스는 UTF-8 포맷을 읽어서 EUC-KR로 변환하여 파일을 생성 하는 예제 입니다.  데이터 읽기 예제에서 파일 출력 부분을 추가 합니다.

*   4 \~ 5 line : RandomAccessFile , FileChannel 으로 out file 객체 선언 \


    ```java
    RandomAccessFile wFile = null;
    FileChannel outChannel = null;
    ```


*   11 \~ 12 line : RandomAccessFile , FileChannel   객체  생성\


    ```java
    wFile = new RandomAccessFile("D:\\Code\\niodatawrite.txt", "rw");
    outChannel = wFile.getChannel();
    ```


*   26 \~ 32 line :  한글  포함 문자로 읽은 라인의 길이 \* 3, 개행 문자  2 바이트를 더해 버퍼사이즈로 할당하고  읽은  문자에 대해서  EUC-KR로  변환 한 후 개행문자를 추가 하고 채널에 씁니다.\


    ```java
    outbuf = ByteBuffer.allocate( (line.length() * 3 )  + 2 );
    outbuf.clear();

    outbuf.put(line.getBytes(Charset.forName("EUC-KR")));
    outbuf.put(System.getProperty("line.separator").getBytes());
    outbuf.flip();
    outChannel.write(outbuf);
    ```

<details>

<summary>UTF-8을 읽어서 EUC-KR로 변환 하여 파일 생성 전체 소스</summary>

{% code lineNumbers="true" %}
```java
RandomAccessFile aFile = null;
FileChannel inChannel = null;

RandomAccessFile wFile = null;
FileChannel outChannel = null;

try {
    aFile = new RandomAccessFile("D:\\Code\\niodata.txt", "rw");
    inChannel = aFile.getChannel();

    wFile = new RandomAccessFile("D:\\Code\\niodatawrite.txt", "rw");
    outChannel = wFile.getChannel();


    ByteBuffer buf = ByteBuffer.allocate((int) inChannel.size());
    int bytesRead = inChannel.read(buf);

    ByteBuffer outbuf;

    while (bytesRead != -1) {
        buf.flip();

        String koreanText = StandardCharsets.UTF_8.decode(buf).toString();
        String[] lines = koreanText.split(System.getProperty("line.separator"));
        for (String line : lines) {
            outbuf = ByteBuffer.allocate( (line.length() * 3 )  + 2 );
            outbuf.clear();

            outbuf.put(line.getBytes(Charset.forName("EUC-KR")));
            outbuf.put(System.getProperty("line.separator").getBytes());
            outbuf.flip();
            outChannel.write(outbuf);

        }

        buf.clear();
        bytesRead = inChannel.read(buf);
    }
} catch (IOException e) {
    throw new RuntimeException(e);
} finally {

    try {
        if (aFile != null) aFile.close();
        if (inChannel != null) inChannel.close();
        if (wFile != null) wFile.close();
        if (outChannel != null) outChannel.close();
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```
{% endcode %}

</details>

## 4. FileChannel 닫기

```java
channel.close();   
```

## 5. FileChannel  위치

개체의 현재 위치를 얻거나 설정 합니다.

```java
long pos channel.position();

channel.position(pos +123);
```

## 6. FileChannel 크기

파일의 크기를 반환합니다.

```java
long fileSize = channel.size();
```

## 7. FileChannel 자르기

1024  바이트로 자릅니다.

```java
channel.truncate(1024);
```

## 8. FileChannel focus

채널에서 기록되지 않은 모든 데이터를 플러시합니다.

운영 체제는 성능상의 이유로 메모리에 데이터를 캐시할 수 있으므로 채널에 기록된 데이터가 실제로 디스크에 기록된다는 보장은 없으므로 모든 데이터를 플러시합니다.

```java
channel.force(true);
```
