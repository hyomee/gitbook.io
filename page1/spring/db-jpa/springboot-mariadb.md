# SpringBoot - MariaDB 연동

Springboot Version : 3.1.3

## 1. 의존성 추가

<pre class="language-gradle"><code class="lang-gradle"><strong>// MariaDB Client
</strong><strong>runtimeOnly 'org.mariadb.jdbc:mariadb-java-client:2.7.6' 
</strong>
// Spring-boot-starter-data-jpa 
implementation 'org.springframework.boot:spring-boot-starter-data-jpa:3.0.4'
 
// QueryDSL Implementation
implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'

annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"
annotationProcessor 'jakarta.annotation:jakarta.annotation-api:2.1.1'
annotationProcessor 'jakarta.persistence:jakarta.persistence-api:3.1.0'

</code></pre>

## 2. Application.yml 설정

```yaml
  datasource:
    driverClassName: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://localhost:3306/hyomeedb     
    username: hyomee
    password: sang0237!
    type: com.zaxxer.hikari.HikariDataSource  # 결 풀이 HikariDataSource의 전용 구현을 사용하고 반환하도록 강제
    hikari:                             # DB Connection Pool
      auto-commit: true                 # 자동커밋 여부. (default: true)
      connection-test-query: SELECT 1   # connection 유효성 검사 쿼리
      pool-name: pool-hyomee            # connection pool 이름
      leak-detection-threshold: 300000
      idle-timeout: 300000
      max-lifetime: 600000
      minimum-idle: 10                  # pool에 유지할 유휴 connection 최소 개수
      maximum-pool-size: 50             # pool에 유지시킬 수 있는 최대 connection 수
  jpa:
#    database-platform: org.hibernate.dialect.H2Dialect  # 접속할 database의 SQL Dialect 설정, JPA를 활용하여 동적 쿼리 생성시, database에 맞는 방언 sql을 생성
    open-in-view: false
    show-sql: true
    naming:
      physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
      implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MariaDBDialect
        show-sql: true                  # Hibernate이 DB에 날리는 모든 쿼리(DDL, DML) 표출
        format_sql: true                # 쿼리 들여쓰기 등의 포맷에 맞춰 표출
        hbm2ddl.auto: update            # 앱 시작시 @Entity로 정의한 테이블의 create 문 실행
        highlight_sql: true
        use_sql_comments: true          # 디버깅이 용이하도록 SQL문 이외에 추가적인 정보를 출력
#        implicit_naming_strategy: org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy
#        physical_naming_strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl

```

