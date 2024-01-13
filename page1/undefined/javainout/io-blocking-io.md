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

<figure><img src="../../../.gitbook/assets/자바IO객체 (1).jpg" alt="" width="511"><figcaption><p>자바 IO 팩키지</p></figcaption></figure>

* **Reader**: 문자 단위로 입력을 처리하는 클래스입니다.
* **Writer**: 문자 단위로 출력을 처리하는 클래스입니다

## 2. **InputStream**

