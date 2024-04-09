# Hello World

## 1. Hello World 콘솔 출력

초간단 Hello World를 출력 하는 예제를 통해서 스프링 배치 5를 이해하고자 합니다.

## 2. 요구사항

* Java  Java 17+
* Spring Boot: spring-boot-starter-parent: v3.2.4
* Spring Batch: spring-boot-starter-batch
* DB: 마리아 DB&#x20;

## 3. 코드

### 3-1. 프로젝트 생성  - 의존성 구성&#x20;

[https://start.spring.io/](https://start.spring.io/) 에서 프로젝트를 생성하고 다운로드 받는다. ( Maven )

<figure><img src="../../.gitbook/assets/image (195).png" alt=""><figcaption><p>프로젝트 구</p></figcaption></figure>

```xml
<dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-batch</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <scope>runtime</scope>
      <optional>true</optional>
    </dependency>
    
    <dependency>
      <groupId>org.mariadb.jdbc</groupId>
      <artifactId>mariadb-java-client</artifactId>
      <scope>runtime</scope>
    </dependency>
    
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    
    <dependency>
      <groupId>org.springframework.batch</groupId>
      <artifactId>spring-batch-test</artifactId>
      <scope>test</scope>
    </dependency>
</dependencies>
```

### 3-2. 환경 구성&#x20;

