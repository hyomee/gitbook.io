# IO(Blocking IO)

Java IO 패키지는Java에서 입출력을 다루기 위한 패키지로 하나의 Thread가 IO 작업에 의존적이기 때문에 하나의 Thread로 하나의 IO를 처리하는 **Blocking 방식 구현**으로 InputStream, OutputStream, Reader, Writer 클래스를 사용합니다.

* **바이트 단위** : IputStream, OutputStream - 이미지, 비디오 및 직렬화 된 객체와 같은 이진 데이터 - InputStream.read() : 바이트 스트림의 원시 내용에 해당하는 0에서 255 사이의 바이트 값을 반환
* **문자 단위** : Reader, Writer - 문자 스트림이므로 문자 데이터 - Reader.read() : 0에서 65357 사이의 문자 값을 반환

Java NIO와 달리 기존 IO 패키지는 JVM 내부 버퍼로 복사 시 발생하는 CPU 연산, GC 관리, IO 요청에 대한 스레드 블록이 발생하게 되는 현상 때문에 효율이 좋지 못한 점이 있습니다

### 1. Java IO 패키지의 핵심 용어 <a href="#id-1.-java-io" id="id-1.-java-io"></a>

* **InputStream**: 바이트 단위로 입력을 처리하는 클래스입니다.
* **OutputStream**: 바이트 단위로 출력을 처리하는 클래스입니다.

자바 IO 팩키지

<figure><img src="https://hyomee.gitbook.io/~gitbook/image?url=https%3A%2F%2F1889142648-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FVfLaXPTe0t3ptNpFbmH5%252Fuploads%252FtLT5XAZIESwYbwElmQ5t%252F%25EC%259E%2590%25EB%25B0%2594IO%25EA%25B0%259D%25EC%25B2%25B4%2520%281%29.jpg%3Falt%3Dmedia%26token%3D7b7220b9-165f-4c31-b793-65c1dcc2cb85&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=53eb0787b4ae2dfe9a7096b54628b43c77d02b78b30be889f40007dde5cf8d17" alt=""><figcaption></figcaption></figure>

* **Reader**: 문자 단위로 입력을 처리하는 클래스입니다.
* **Writer**: 문자 단위로 출력을 처리하는 클래스입니다.

<figure><img src="https://hyomee.gitbook.io/~gitbook/image?url=https%3A%2F%2F1889142648-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FVfLaXPTe0t3ptNpFbmH5%252Fuploads%252FPIKWGqDIaVxXpquwM7rp%252F%25EC%259E%2590%25EB%25B0%2594IO_READER.jpg%3Falt%3Dmedia%26token%3D519de965-c864-4cab-979f-db88f4509e9e&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=163a7f2a62d8c06172af34c6f3e117149dbeeb081bceca22ba892b8ef2954e70" alt=""><figcaption></figcaption></figure>
