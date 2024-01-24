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

## 2. 파일 존재 여부&#x20;

* Files.exists() : 파일  또는 디렉토리 존재 여부&#x20;

```java
Path currentFilePath = Paths.get("D:\\Code\\niodata.txt");
boolean isFile = Files.exists(currentFilePath);
System.out.println(String.format("파일 존재 : %s", isFile));
```

* 두번째 파라메터는 Link Option으로 심볼릭을 제외 하는 경우 사용합니다

```java
Files.exists(currentFilePath, new LinkOption[]{ LinkOption.NOFOLLOW_LINKS});
```

## 3. 파일/디렉토리 생성&#x20;

Files.createDirectory() 메서드를 통해서 생성 합니다. Files.create로 시작하는 메서드를 API를 통해서 확인 하면 다음과 같은 메서드가 있습니다,

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

{% code lineNumbers="true" %}
```java
Path currentPath = Paths.get("D:\\Code\\SampleDir");
try {
  Path createPath = Files.createDirectory(currentPath);
  boolean isCreatePath = Files.exists(createPath);
  System.out.println(String.format("파일 존재 : %s", isCreatePath));
} catch (IOException e) {
  throw new RuntimeException(e);
}
```
{% endcode %}

* 1 line : 생성할 디렉토리&#x20;
* 3 line : 디렉토리 생성&#x20;
* 4\~5 line : 생성한 디렉토리가 존재 하는 지 여부 체크 후 출력

## 3. 파일 복사/이동

### 3-1.  파일 복사 : Files.copy()

{% code lineNumbers="true" %}
```java
Path sourceFile = Paths.get("D:\\Code\\niodata.txt");
Path targetFile = Paths.get("D:\\Code\\SampleDir\\niodata.txt");
try {
  Files.copy(sourceFile, targetFile, StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) {
  throw new RuntimeException(e);
}
```
{% endcode %}

* 4 line : 복사 기능을 세 번째 파라메터는 옵션으로 파일이 존재 하면 기존 파일을 덮어쓰기 입니다.

### 3-2.  파일 이동 : Files.move()

copy 메서드와 동일 하지만 원본 파일을 다른 곳으로 이동 시키는 것 입니다.

{% code lineNumbers="true" %}
```java
Path sourceFile = Paths.get("D:\\Code\\niodata.txt");
Path targetFile = Paths.get("D:\\Code\\SampleDir\\niodata.txt");
try {
  Files.copy(sourceFile, targetFile, StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) {
  throw new RuntimeException(e);
}
```
{% endcode %}

* 4 line : 복사 기능을 세 번째 파라메터는 옵션으로 파일이 존재 하면 기존 파일을 덮어쓰기 입니다.

## 4. 파일 삭제

```java
Path targetFile = Paths.get("D:\\Code\\SampleDir\\niodata.txt");
try {
  Files.delete(targetFile);
} catch (IOException e) {
  throw new RuntimeException(e);
}
```
