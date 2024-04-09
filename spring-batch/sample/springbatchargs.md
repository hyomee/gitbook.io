# 배치 파라메터 전달

Hello World예제 Job을 동적으로 수행 하기 위헤서는 여러J방법이  있지만 여기에서는  실행시 파라메터로 받아서 처리 하는 방법에 대해서 소개 하고자 합니다.

## 1. 요구사항

1. 스프링 배치 실행시 파라메터를 받아서 Job를 실행 할 수 있아야 하고 전달 받은 파라메터는 Tasklet 까지 전달되어야 한다.&#x20;
2. 베치 실행시 현재 일자가 자동 설정 되어야 한다.

## 2. 생각하기

Java는 main 메서드의 인자(args)를 통해서 Java가 실행 될 때 데이터를 받을 수 있습니다. 이것은 스프링 부트도 동일 하므로 인자를 받아서 다음과 같은 코드를 작성 합니다.

1. ain Method로 받은 Argument를 Map에 저장 한다.
2. Map Data를 Json으로 변환 해서 JobParameter로 전달 한다.
3. 배치는 즉시 수행, 스케줄 수행 등 여러 요소가 있어 application.yml에 배치에 타입을 정의 해서 direct인 경우만 즉시 수행 한다.
4. Argument는 Key, Value 구조로 설정 하며, -- 을 사용해서 여러 속성을 받을 수 있게 한다.
5. 스프링에서 스프링 애플리케이션이 실행되면서 다른 명령을 실행하기 위해서는 여러 방법이 있지만 여기에서는 "ApplicationRunner"를 상속 받아 run 메서드를 재 정의 하는 방법을 사용한다.

## 3. 코드

### 3-1.    ApplicationRunner

#### 3-1-1. HelloWorldApplication.java 파일에서 JobLauncher 실행 코드를 다음과 같이 제거 합니다.

{% code title="HelloWorldApplication.java" lineNumbers="true" %}
```java
@SpringBootApplication
public class HelloWorldApplication {

    public static void main(String[] args) {
      SpringApplication.run(HelloWorldApplication.class, args);
   }

}
```
{% endcode %}

#### 3-1-2. ApplicationRunner 를 상속 받아 run 메서드를 다음과 같이 작성한다.

{% code lineNumbers="true" %}
```java
@Order(0)
@Component
@Slf4j
public class AcubeBatchCommandLineRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        if (args.getSourceArgs().length == 0) {
            log.debug("Arguments 는 없습니다.");
            return;
        }
        for (String arg: args.getSourceArgs()) {
            log.debug(String.format("arg : %s", arg ));
        }
    }
}
```
{% endcode %}

* 1 Line:  CommandLineRunner 객체가 많은 경우 실행 우선 순위 지정&#x20;
* 2 Line:  스프링 빈으로 선언&#x20;
* 7 Line: Arguments 이 비어 있으면 로그 출력 후 리턴, 스프링 배치에서 기본으로 설정된 속성으로 진행 됨
* 11\~13 Line: Arguments 츨력

#### 3-1-3. 실행&#x20;

arguments:  --Job=myjob --key=1

<figure><img src="../../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

### 3-2.    Map로 변환 메서드 작성

```java
private Map<String, Object> argumentsToMap(ApplicationArguments args) {
    Map<String, Object> argMap = new HashMap<>();
    for(String str: args.getSourceArgs()) {
        String[] keyValue = str.split("=");

        if (keyValue.length == 2) {
            String key =  keyValue[0];
            if (key.contains("--")) {
                // Remove the leading "--"
                key = keyValue[0].substring(2);
            }

            String value = keyValue[1];
            argMap.put(key, value);
        }
    }
    return argMap;
}
```

arguments 객체를 받아서 Map으로 변경하는 코드로 key=value 구조로 받은 데이터를 split 메서드를 사용해서 String 배열로 변환 후 0번째 요소에 "--"이 포함 되어 있으먄 제외 후 Map에 저장 후 반환한다.

run 메서드를 다음과 같이 변경 후 실행 합니다.

```java
public void run(ApplicationArguments args) throws Exception {
    if (args.getSourceArgs().length == 0) {
        log.debug("Arguments 는 없습니다.");
        return;
    }
    Map<String, Object> argsMap = argumentsToMap( args);
    argsMap.forEach( (pa, cnt)-> log.debug(pa + " = " + (String) argsMap.get(pa))) ;
}
```

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### 3-3.    Map로 변환 메서드 작성
