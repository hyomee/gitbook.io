# Path/Files

파일 시스템의 절대 또는 상대 경로의 파일/디렉토리를 나타냅니다.&#x20;

* 절대 경로 : 파일 시스템의 루트에서 파일 까지의 전체 경로
* 상대 경로 : 다른 경로에 상대적인 파일 또는 디렉터리의 경로

## 1. Path 생성

paths(java.nio.file.Paths) 클래스의 get() 메서드를 사용해서 Path 인스턴스를 생성 합니다.

```java
// 절대 경로
Path path = Paths.get("c:\\data\\myfile.txt");

// 상대 경로
Path file = Paths.get("d:\\data", "projects\\a-project\\myfile.txt");
```

## 2.&#x20;
