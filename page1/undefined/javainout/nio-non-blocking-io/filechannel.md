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

## 2. 데이터 읽기

채널에 있는 파일을 buffer로 읽어 오기 위해서 사이즈할당(allocate)하고 read로 읽어 옵니다.

```java
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
```

바이트가 . -1이 반환되면 파일 끝에 도달합니다.

한글을 출력 할 떄 byte 단위로 할당 하면 문제가 발생 합니다.  해당 문제를 해결 하기 위해서는 라인단위로 읽어서 변환 하거나 전체 채널의 크기 만큼 읽어서 변환 하는 방법이 있습니다.

여기서는 채널의 사이즈 만큼 읽어서 변환하는 예제를 작성 합니다.&#x20;

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

            String koreanText = StandardCharsets.UTF_8.decode(buf).toString();
            String[] lines = koreanText.split("\\r?\\n");
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

* 8 line :  ByteBuffer.allocate는 바이트 단위로 버퍼에 읽기 위해 할당 하는 것인데 한글(UTF-8) 파일은 2 Byte로 문재가 발생 하기 때문에 채널 크기 만큼 버퍼을 할당 합니다.
* 9 line : 채널 크기 만큼 버퍼에 할당 하였으므로 파일의 전체 내용을 읽어 옵니다.
* 14 line : 한글(UTF-8)로 변환 합니다.
* 15 line \~ 18 line : 라인 단위로 처리 하기 위해 split를 하고 출력 합니다. 라인 단위로 다른 처리를 하기 위해서는 이곳에서 업무 로직을 두어야 합니다.  ( 비동기 )

### 2-1  힙 메모리 사용

allocate는   System Memory를 사용 합니다.&#x20;

```java
ByteBuffer buf = ByteBuffer.allocate((int) inChannel.size());
```

만약 자바의 힙메모리를 사용하기 위해서는 allocateDirect 메서드를  사용하여음과 같이 변경 하면 됩니다.

```java
ByteBuffer buf = ByteBuffer.allocateDirect((int) inChannel.size());
```

## 3. 데이터 쓰기

