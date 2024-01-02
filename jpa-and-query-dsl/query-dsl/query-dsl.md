# Query-DSL 개요 및 설정

## **1. 개요** <a href="#id-1" id="id-1"></a>

* Hibernate용 HQL은 Querydsl의 첫 번째 대상 언어였지만 현재는 JPA, JDO, JDBC, Lucene, Hibernate Search, MongoDB, Collections 및 RDFBean을 백엔드로 지원합니다.
* 필요 라이브러리
  * querydsl-jpa : QueryDSL JPA 라이브러리
  * querydsl-apt : 쿼리 타입을 생성 할 때 필요한 라이브러리
* Querydsl은 JPQL과 Criteria 쿼리를 모두 대체할 수 있다

## **2. 장점** <a href="#id-2" id="id-2"></a>

* IDE의 코드 자동 완성 기능 사용
* 문법적으로 잘못된 쿼리를 허용하지 않음
* 도메인 타입과 프로퍼티를 안전하게 참조할 수 있음
* 도메인 타입의 리팩토링을 더 잘 할 수 있음
* JPA의 연관 관계 매핑이 없이 바로 객체로 바인딩 할 수 있음
* 다이나믹쿼리 지원

## **3. 의존성 설정** <a href="#id-3" id="id-3"></a>

* pom.xml에 환경 설정을 하고 mvn compile 을 하면 target/generated-sources에 Qxxxx.java로 시작하는 쿼리 타입이 생성됩니다.

```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <version>${querydsl.version}</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <version>${querydsl.version}</version>
</dependency>
```

* Maven APT 플러그인 구성을 다음과 같이 합니다

```xml
<project>
    <build>
        <plugins>
        ...
            <plugin>
                <groupId>com.mysema.maven</groupId>
                <artifactId>apt-maven-plugin</artifactId>
                <version>1.1.3</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>process</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>target/generated-sources/java</outputDirectory>
                            <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        ...
        </plugins>
    </build>
</project>
```

JPAAnnotationProcessor는 Entity 어노테이션이 달란 도메인 유형을 찾아 Qxxxx 파일의 쿼리 유형으로 생성해주는 기능으로 target/generated-sources/java에 파일이 생성됩니다.(outputDirectory: 파일이 생성되는 위치)

> Gradle 설정

```gradle
dependencies {
    runtimeOnly 'com.h2database:h2:2.1.214' 
    implementation project(':project-jpa')
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa:3.0.4'
    // QueryDSL Implementation
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'

    annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"
    annotationProcessor 'jakarta.annotation:jakarta.annotation-api:2.1.1'
    annotationProcessor 'jakarta.persistence:jakarta.persistence-api:3.1.0'
}
// Querydsl 설정부
def generatedDir = "$buildDir/generated/querydsl" // 'src/main/generated' 

sourceSets {
    main.java.srcDirs += [ generatedDir ]
}

tasks.withType(JavaCompile).configureEach {
    options.getGeneratedSourceOutputDirectory().set(file(generatedDir))
}

clean.doLast {
    file(generatedDir).deleteDir() 
}
```
