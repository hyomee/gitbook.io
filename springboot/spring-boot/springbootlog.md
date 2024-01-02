# SpringBoot 로그 출력

스프링 부트에서 로그는 log4j2, logbak을 사용하여 출력 할 수 있습니다.

| spring-boot-starter-log4j2  | 로깅을 위해 Log4j2를 사용하기 위한 스타터.             |
| --------------------------- | --------------------------------------- |
| spring-boot-starter-logging | Logback을 사용하여 로깅하기 위한 스타터입니다. 기본 로깅 스타터 |

참고 : [https://docs.spring.io/spring-boot/docs/3.1.2/reference/html/features.html#features.logging](https://docs.spring.io/spring-boot/docs/3.1.2/reference/html/features.html#features.logging)

기본은 logbak 기반으로 application.properties( or yml) 에 설정한 속성이 반영 될 수 있습니다. 로깅 레벨은 TRACE, DEBUG, INFO, WARN, ERROR, FATAL 또는 OFF 중 하나를 사용이 되면 속성은 다음과 같다.

## 1. 파일 지정&#x20;

| logging.file.name | logging.file.path  | 예           | 설명                                                            |
| ----------------- | ------------------ | ----------- | ------------------------------------------------------------- |
| (없음)              | (없음)               | <p><br></p> | 콘솔 전용 로깅.                                                     |
| 특정 파일             | (없음)               | my.log      | 지정된 로그 파일에 씁니다. 이름은 정확한 위치이거나 현재 디렉터리에 상대적일 수 있습니다.           |
| (없음)              | 특정 디렉토리            | /var/log    | spring.log지정된 디렉토리에 씁니다 . 이름은 정확한 위치이거나 현재 디렉터리에 상대적일 수 있습니다. |

## 2. 파일 속성&#x20;

| 이름                                                   | 설명                                    |
| ---------------------------------------------------- | ------------------------------------- |
| logging.logback.rollingpolicy.file-name-pattern      | 로그 아카이브를 만드는 데 사용되는 파일 이름 패턴입니다.      |
| logging.logback.rollingpolicy.clean-history-on-start | 애플리케이션이 시작될 때 로그 아카이브 정리가 발생해야 하는 경우. |
| logging.logback.rollingpolicy.max-file-size          | 아카이브되기 전 로그 파일의 최대 크기.                |
| logging.logback.rollingpolicy.total-size-cap         | 로그 아카이브가 삭제되기 전에 취할 수 있는 최대 크기입니다.    |
| logging.logback.rollingpolicy.max-history            | 보관할 아카이브 로그 파일의 최대 수(기본값은 7)입니다.      |

## 3. 로그 그룹&#x20;

특정 팩키지의 로그 레벨을 조정 하는 것을 의미 합니다.

```
logging:
  level:
     com.xxx.ooo: DEBUG
```

## 4, 로그 후크

war 로 배포 하지 않으면 자동으로 JVM이 종료 되면 리소스 해제

그외 속성은 Spring-Boot 문서를 참고 하면 됩니다.

예제는 파일 기반으로 작성 합니다.

```yaml

application.yml

spring:
  application:
    name: hyomee
  profiles:
    active: local
    
logging:
  level:
    root: info
  config: classpath:logback-spring.xml
```

\* logback-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  
  <!-- log level을 application.yml에 지정한 레벨로 기본 설정 -->
  <springProperty name="LOG_LEVEL" source="logging.level.root" scope="context"/>
  
  <!-- SERVICE_NAME을 application.yml에 지정한 레벨로 기본 설정 -->
  <springProperty name="SERVICE_NAME" source="spring.application.name" scope="context"/>
  
  <!-- log file이 적재될 위치 지정 -->
  <property name="LOG_PATH" value="${LOG_PATH:-./logs}"/>
  
  <!-- 실행 중 로그 파일  -->
  <property name="LOG_FILE" value="${LOG_FILE:-}${SERVICE_NAME}_app"/>
  
  <!-- 콘솔 로그 파일 패턴  -->
  <property name="LOG_PATTERN" value="[${SERVICE_NAME}] [%d{yyyy-MM-dd HH:mm:ss.SSS}] [%thread] %-5level %logger{36} - %msg%n"/>
 
 <!-- rolling 파일 사이즈 -->
  <property name="MAX_FILESIZE" value="5MB"/>

  <appender name="CONSOLE_APPENDER" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>${LOG_PATTERN}</pattern>
    </encoder>
  </appender>

  <appender name="FILE_APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <encoder>
      <pattern>${LOG_PATTERN}</pattern>
    </encoder>
    <file>${LOG_PATH}/${LOG_FILE}.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- rolling 파일 이름  -->
      <fileNamePattern>${LOG_PATH}/${LOG_FILE}.log.%d{yyyy-MM-dd}.gz</fileNamePattern>
      <maxHistory>${MAX_HISTORY}</maxHistory>
      <maxFileSize>${MAX_FILESIZE}</maxFileSize>
    </rollingPolicy>
  </appender>

  <root level="${LOG_LEVEL}">
    <appender-ref ref="FILE_APPENDER"/>
    <springProfile name="local">
      <appender-ref ref="CONSOLE_APPENDER"/>
    </springProfile>
  </root>


  <!-- Logger -->
  <logger name="com.hyomee.*" level="DEBUG" appender-ref="console" />
  <logger name="com.zaxxer.hikari.HikariConfig" level="DEBUG" appender-ref="console" />
  <logger name="org.hibernate.SQL" level="DEBUG" appender-ref="console" />
  <logger name="org.hibernate.type.descriptor.sql" level="TRACE" appender-ref="console"/>
</configuration>
```

\


로그내용\[hyomee] \[2023-08-15 13:17:12.633] \[Test worker] INFO org.hibernate.Version - HHH000412: Hibernate ORM core version 6.2.6.Final

\
