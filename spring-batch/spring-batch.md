# Spring Batch 의존성 구성

## 1. pom.xml

```yaml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>

<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
    <version>3.3.3</version>
    <scope>runtime</scope>
</dependency>

<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-test</artifactId>
    <scope>test</scope>
</dependency>
```

## 2. jpa

```yaml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

```yaml
spring:
    application:
        name: base_springboot_bacth
    datasource:
        driver-class-name: org.mariadb.jdbc.Driver
        url: jdbc:mariadb://x.x.x.x:14302/hong
        username: hong
        password: hong1234
    jpa: open-in-view: false
        show-sql: true
        hibernate:
            dialect: org.hibernate.dialect.MariaDB102Dialect
```

## 3. mybatis

```yaml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
```

```yaml
spring:
  application:
    name: base_springboot_bacth
  batch:
    jdbc:
      initialize-schema: never # always # never
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://1.217.139.30:14302/hong
    username: hong
    password: hong1234\
mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: : kr.co.abacus.batch.**.dto
```

## 4. Jpa + mybatis

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency><dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
```

```yaml
spring:
  application:
    name: base_springboot_bacth
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://x.x.x.x:14302/hong
    username: hong
    password: hong1234
  jpa:
    open-in-view: false
    show-sql: true
    hibernate:
      dialect: org.hibernate.dialect.MariaDB102Dialect
mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: : kr.co.abacus.batch.**.dto
```
