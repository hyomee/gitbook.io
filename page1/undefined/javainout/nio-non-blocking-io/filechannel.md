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



## 3. 데이터 쓰기

전체소스&#x20;

```java
public static void main(String[] args) {
    RandomAccessFile aFile     = null;
    try {
        // 
        aFile = new RandomAccessFile("data/nio-data.txt", "rw");
                FileChannel inChannel = aFile.getChannel();
    } catch (IOException e) {
        throw new RuntimeException(e);
    } finally {
        // 파일은 항상 닫아야 합니다.
        if ( aFile != null  ) {
            try { 
                aFile.close();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```
