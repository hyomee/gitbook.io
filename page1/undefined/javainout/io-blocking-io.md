# IO(Blocking IO)

Java IO 패키지는Java에서 입출력을 다루기 위한 패키지로 하나의 Thread가 IO 작업에 의존적이기 때문에 하나의 Thread로 하나의 IO를 처리하는 **Blocking 방식  구현**으로 InputStream, OutputStream, Reader, Writer 클래스를 사용합니다.&#x20;

* **바이트 단위** : IputStream, OutputStream
* **문자 단위** : Reader,  Writer&#x20;

## 1. Java IO 패키지의 핵심 용어

* **InputStream**: 바이트 단위로 입력을 처리하는 클래스입니다.
* **OutputStream**: 바이트 단위로 출력을 처리하는 클래스입니다.
* **Reader**: 문자 단위로 입력을 처리하는 클래스입니다.
* **Writer**: 문자 단위로 출력을 처리하는 클래스입니다

<figure><img src="../../../.gitbook/assets/자바IO객체 (1).jpg" alt="" width="511"><figcaption><p>자바 IO 팩키지</p></figcaption></figure>

