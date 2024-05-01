# Path/Files

파일 시스템의 절대 또는 상대 경로의 파일/디렉토리를 나타냅니다.

* 절대 경로 : 파일 시스템의 루트에서 파일 까지의 전체 경로
* 상대 경로 : 다른 경로에 상대적인 파일 또는 디렉터리의 경로

## 1. Path 생성 <a href="#id-1.-path" id="id-1.-path"></a>

paths(java.nio.file.Paths) 클래스의 get() 메서드를 사용해서 Path 인스턴스를 생성 합니다.

Copy

```java
// 절대 경로
Path path = Paths.get("c:\\data\\myfile.txt");

// 상대 경로
Path file = Paths.get("d:\\data", "projects\\a-project\\myfile.txt");
```

## 2. 파일 존재 여부 <a href="#id-2" id="id-2"></a>

* Files.exists() : 파일 또는 디렉토리 존재 여부

```java
Path currentFilePath = Paths.get("D:\\Code\\niodata.txt");
boolean isFile = Files.exists(currentFilePath);
System.out.println(String.format("파일 존재 : %s", isFile));
```

* 두번째 파라메터는 Link Option으로 심볼릭을 제외 하는 경우 사용합니다

```java
Files.exists(currentFilePath, new LinkOption[]{ LinkOption.NOFOLLOW_LINKS});
```

## 3. 파일/디렉토리 생성 <a href="#id-3" id="id-3"></a>

Files.createDirectory() 메서드를 통해서 생성 합니다. Files.create로 시작하는 메서드를 API를 통해서 확인 하면 다음과 같은 메서드가 있습니다,

![](https://hyomee.gitbook.io/\~gitbook/image?url=https%3A%2F%2F1889142648-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FVfLaXPTe0t3ptNpFbmH5%252Fuploads%252Fc9G9uUQ3pjoaqZPZDiGj%252Fimage.png%3Falt%3Dmedia%26token%3D11419055-f698-47bd-898e-3758ca1526ad\&width=768\&dpr=4\&quality=100\&sign=a0d8caf16188ae444ea9bfc2d8f6e5e84842d02f5d8df90877b2b72c70df08b4)Copy

```
Path currentPath = Paths.get("D:\\Code\\SampleDir");
try {
  Path createPath = Files.createDirectory(currentPath);
  boolean isCreatePath = Files.exists(createPath);
  System.out.println(String.format("파일 존재 : %s", isCreatePath));
} catch (IOException e) {
  throw new RuntimeException(e);
}
```

* 1 line : 생성할 디렉토리
* 3 line : 디렉토리 생성
* 4\~5 line : 생성한 디렉토리가 존재 하는 지 여부 체크 후 출력

## 4. 파일 복사/이동 <a href="#id-3-1" id="id-3-1"></a>

### 4-1. 파일 복사 : Files.copy() <a href="#id-3-1.-files.copy" id="id-3-1.-files.copy"></a>

```java
Path sourceFile = Paths.get("D:\\Code\\niodata.txt");
Path targetFile = Paths.get("D:\\Code\\SampleDir\\niodata.txt");
try {
  Files.copy(sourceFile, targetFile, StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) {
  throw new RuntimeException(e);
}
```

* 4 line : 복사 기능을 세 번째 파라메터는 옵션으로 파일이 존재 하면 기존 파일을 덮어쓰기 입니다.

### 4-2. 파일 이동 : Files.move() <a href="#id-3-2.-files.move" id="id-3-2.-files.move"></a>

copy 메서드와 동일 하지만 원본 파일을 다른 곳으로 이동 시키는 것 입니다.

```java
Path sourceFile = Paths.get("D:\\Code\\niodata.txt");
Path targetFile = Paths.get("D:\\Code\\SampleDir\\niodata.txt");
try {
  Files.copy(sourceFile, targetFile, StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) {
  throw new RuntimeException(e);
}
```

* 4 line : 복사 기능을 세 번째 파라메터는 옵션으로 파일이 존재 하면 기존 파일을 덮어쓰기 입니다.

## 5. 파일 삭제 <a href="#id-4" id="id-4"></a>

```java
Path targetFile = Paths.get("D:\\Code\\SampleDir\\niodata.txt");
try {
  Files.delete(targetFile);
} catch (IOException e) {
  throw new RuntimeException(e);
}
```
