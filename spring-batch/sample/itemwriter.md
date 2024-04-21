# ItemWriter 이해

ItemWriter는 Spring Batch의 요소 중 하나로 데이터를 쓰는 기능을 담당한다. 즉 reader가 File, DB, Message Queue와 같은 것에서 데이터를 읽어들이는 것과 반대로  File, DB, Message Queue에 데이터를 쓰거나 출력하는 역활을 한다.

## 1. Chunk 단위 처리

```java
public interface ItemWriter<T> {
    void write(Chunk<? extends T> items) throws Exception;
}
```

Spring Batch의 ItemWriter는 item 하나를 처리 하는 것에서 출발 하여 현재 (Spring Batch v5.1.1)에서는 Chuck 단위로 처리한다.

<figure><img src="../../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>

Reader와 Processor에서 처리된 Item을 지정된 Chunk 단위로 모아서 Writer에 보내서 처리한다.

## 2. ItemWriter 전략

Spring Batch에서 ItemWriter는 다음과 같은 전략을 제공한다.

* FlatFileItemWriter:  CSV 및 구분된 파일과 같은 플랫 파일에 데이터를 쓰는 데 사용되며 사용자 지정 서식 및 레코드 구분 기호에 대한 지원을 제공한다.
* DB Writer: Database의 영속성(JPA, Hibernate)는과 항상 마지막에 Flush를 해야 하며 모든 Item을 처리 한 후 Spring Batch는 현재 트랜잭션을 커밋한다.
  * JdbcBatchItemWriter: JDBC를 사용하여 관계형 데이터베이스에 데이터를 쓰는 데 사용되며 Batch 기능(한번에 DB에 전달)을 사용하여 성능을 항상 시킬수 있다.
  * JpaItemWriter: JPA 기술을  사용하여 관계형 데이터베이스에 데이터를 쓰는 데 사용한다.
  * MyBatisBatchItemWriter: MyBatis를 사용하여 관계형 데이터베이스에 데이터를 쓰는 데 사용한다.
  * HibernateItemWriter: Hibernate를 사용하여 관계형 데이터베이스에 데이터를 쓰는 데 사용한다.
* JmsItemWriter:  JMS 큐 또는 토픽에 데이터를 쓰는 데 사용되며 분산 처리 및 비동기 메시징을 지원한다.
* StaxEventItemWriter: StAX API를 사용하여 XML 파일에 데이터를 쓰는 데 사용되고. 복잡한 XML 구조를 생성하고 대용량 XML 파일을 처리할 수 있도록 지원한다.

### 2-1. FlatFileItemWriter

텍스트 파일(예: CSV, 고정 너비 파일)에 데이터를 쓰는 데 사용되며. 이 클래스는 **리소스**를 설정하여 파일의 위치를 지정하고, LineAggregator를 사용하여 객체를 문자열로 변환한 다음 파일에 쓰게 된다.

FlatFileItemWriter의 주요 기능은 다음과 같다:

* Resource를 통해 파일의 위치를 지정하고, Writable Resource를 나타내야 한다.
* LineAggregator를 사용하여 객체를 문자열로 변환한다
* FieldExtractor와 LineAggregator를 조합하여, 객체의 필드를 추출하고 이를 기반으로 문자열을 생성한다.
* headerCallback과 footerCallback을 사용하여 파일의 시작과 끝에 헤더와 푸터를 추가할 수 있다.
* append 옵션을 사용하여 파일에 데이터를 추가할지, 덮어쓸지를 결정할 수 있다.

FlatFileItemWriter는 성능을 위해 BufferedWriter를 사용하며, 재시작 가능한 기능도 제공하여 대량의 데이터를 텍스트 파일 형식으로 쉽고 효율적으로 출력할 수 있다.

### 2-2. JdbcBatchItemWriter

<table><thead><tr><th width="174">Property</th><th width="153">Parameter Type</th><th>설명</th></tr></thead><tbody><tr><td>assertUpdates</td><td>boolean</td><td>적어도 하나의 항목이 행을 업데이트하거나 삭제하지 않을 경우 예외를 throw할지 여부를 설정, 기본값은 <code>true</code>, Exception:<code>EmptyResultDataAccessException</code></td></tr><tr><td>columnMapped</td><td>void</td><td>Key,Value 기반으로 Insert SQL의 Values를 매핑 (ex: <code>Map&#x3C;String, Object></code>)</td></tr><tr><td>beanMapped</td><td>void</td><td>Pojo 기반으로 Insert SQL의 Values를 매핑</td></tr></tbody></table>

### 2-3. JpaItemWriter

Java Persistence API (JPA)를 사용하여 데이터베이스에 접근하는 전략으로 데이터베이스에 대한 영속성을 관리하며, flush()와 clear() 메소드를 통해 영속성 컨텍스트를 관리한다.

JpaItemWriter를 사용하기 위해서는  spring-boot-starter-data-jpa 의존성을 가지고 있어야  한다.

JpaItemWriter의 주요 작업은 다음과 같습니다:

* **Chunk 단위**로 처리된 아이템들을 데이터베이스에 일괄적으로 쓰기(flush) 한다.
* **트랜잭션**이 성공적으로 완료되면, 현재 트랜잭션을 커밋(commit)한다.
* **영속성 컨텍스트**를 관리하여, 각 Chunk의 처리가 끝난 후에는 **flush**와 **clear**를 호출하여 JPA의 1차 캐시를 비우고, 데이터베이스와의 동기화를 수행한다.

### 2-4. MyBatisBatchItemWriter

MyBatis를 사용하여 데이터베이스에 대한 배치 작업을 수행하는 것으로 SqlSessionTemplate의 배치 기능을 활용하여 주어진 아이템들을 처리하며  Java 소스 코드 내에 쿼리를 작성하는 대신, MyBatis 설정 파일에 쿼리를 정의하여 사용할 수 있어, 코드의 가독성과 관리가 용이하다.&#x20;

MyBatis를 사용하기 위해서는 mybatis-spring-boot-starter 의존성을 가지고 있어야  한다.

MyBatisBatchItemWriter의 주요 특징은 다음과 같습니다:

* MyBatis의 SQL statement ID를 사용하여, MyBatis설정 파일에 정의된 SQL 문을 실행한다.
* Chunk 단위로 데이터를 처리하며, 각 Chunk가 처리될 때마다 배치로 데이터베이스에 쓰기 작업을 수행한다..
* ExecutorType을 BATCH로 설정하여, Step에서 정의한 FetchSize만큼씩 처리해주는 기능을 가지고 있다.

### 2-5. HibernateItemWriter

Hibernate 세션을 사용하여 현재 Hibernate 세션의 일부가 아닌 엔티티를 저장하거나 업데이트하는 기능으로 Writer는 Chunk 기반 처리에 사용되며, 각 청크의 경계에서 세션을 flush하고, 기본적으로 쓰기 작업 후에 세션을 한다.

HibernateItemWriter의 주요 기능은 다음과 같습니다:

* Chunk 단위로 처리된 아이템들을 데이터베이스에 일괄적으로 저장하거나 업데이트한다..
* 트랜잭션이 성공적으로 완료되면, 현재 트랜잭션을 커밋(commit)한다.
* 영속성 컨텍스트를 관리하여, 각 Chunk의 처리가 끝난 후에는 flush와 clear를 호출하여 Hibernate의 1차 캐시를 비우고, 데이터베이스와의 동기화를 수행한다.

### 2-6. JmsItemWriter

Messaging Service (JMS)를 사용하여 메시지를 보내는 데 사용되며, JmsTemplate을 사용하여 작성된 아이템들을 JMS 대기열에 전송하는 기능을 제공하며 **분산 시스템**에서 메시지 기반의 비동기 통신을 필요로 하는 배치 처리에 매우 유용하다. &#x20;

JmsItemWriter를 사용하면, 배치 처리 과정에서 생성된 데이터를 다른 시스템이나 애플리케이션으로 신뢰성 있게 전송할 수 있다

JmsItemWriter의 주요 기능은 다음과 같습니다:

* **JmsTemplate**을 설정하여, 기본 대상(destination)이 있는 경우 해당 대상으로 아이템들을 전송.
* **Chunk 단위**로 처리된 아이템들을 **JMS 대기열**에 일괄적으로 전송합니다.
* 설정된 속성들이 확정된 후에는 \*\*스레드 안전(thread-safe)\*\*하게 동작한다.

### 2-7. StaxEventItemWriter

XML 파일로 데이터를 쓰기 위해 사용되는 **ItemWriter** 구현체로 StAX (Streaming API for XML)와 Marshaller를 사용하여 객체를 XML로 직렬화하는 기능을 제공한다.  또한 재시작, 통계 및 트랜잭션 기능을 제공하며, 해당 인터페이스를 구현함으로써 이러한 기능을 지원한다.

StaxEventItemWriter의 주요 기능은 다음과 같습니다:

* Resource, marshaller, rootTagName 등을 설정하여 사용한다.
* Marshaller를 통해 Java 객체를 XML로 변환하고, 각 fragment마다 StartDocument와 EndDocument 이벤트를 필터링한다.
* 커스텀 이벤트 writer를 사용하여 Resource에 데이터를 쓰게 한다.
* 헤더와 푸터 콜백을 제공하여, 파일이 열릴 때 한 번 실행되는 헤더를 추가하거나, 모든 아이템이 쓰여진 후 파일을 닫기 전에 푸터를 추가할 수 다.
