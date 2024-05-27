# Reader/Writer

## 1. ItemReader

Spring Batch가 Chunk 지향 처리를 하는데 중요한 역할을 하는것으로 데이터를 읽어들이는 컴포넌트로  다양한 데이터 소스 데이터베이스, 파일, XML, JSON 등이 있으며, 필요에 따라 사용자가 직접 커스텀한 Reader를 만들어 사용할 수도 있다.

### 1-1. I**temReader**의 종류

주요 **ItemReader** 구현체로는 다음과 같은 것들이 있다.

* **FlatFileItemReader**: 파일 시스템의 플랫 파일에서 데이터를 읽어 오는데 사용한다.
* **Jdbc 관련 ItemReader**
  * **JdbcCursorItemReader**: 데이터베이스에서 JDBC 커서를 사용하여 데이터를 읽어오는 데 사용한다.
  * **JdbcPagingItemReader**: 데이터베이스에서 페이징 쿼리를 사용하여 데이터를 읽어오는 데 사용한다.
* **MyBatisItemReader**
  * **MyBatisCursorItemReader:** 한 번에 조회해온 결과를 Chunk만큼 트랜잭션을 분할하여 대용량 처리
  * **MyBatisPagingItemReader:** MyBatis를 사용하여  Paging 기반으로 동작하며, 데이터베이스에서 데이터를 읽어오는 방식
* **영속성 관련 ItemReader**
  * **JPAItemReader:** **JPA**를 기반으로 데이터베이스 레코드를 읽어오는 역할을 합니다. **JPQL** 쿼리를 실행하여 요청한 데이터를 검색
  * **HibernateCursorItemReader**: Hibernate 세션을 통해 데이터베이스에서 데이터를 읽어오는 데 사용한다.
* **MongoItemReader**: MongoDB에서 데이터를 읽어오는 데 사용한다.
* **JmsItemReader**: JMS 큐에서 메시지를 읽어오는 데 사용한다.
* **StaxEventItemReader**: XML 파일에서 데이터를 읽어오는 데 사용한다

### 1-2.  Cursor 과 Paging 차이점&#x20;

#### 1-2-1.  Cursor

* 배치 처리가 완료될때까지 DB Connection이 연결한다.
* 하나의 Connection에서 처리 되기 때문에 Thread Safe하지 않다.
* 데이터베이스 결과 집합을 한 줄씩 읽어오는 구조로 데이터를 순차적으로 처리를 해야 한다.&#x20;
* Cursor를 사용하면 JVM 메모리에 한 번에 모든 결과를 올려둘 필요가 없으므로, 대량 데이터를 효율적으로 처리할 수 있다.
* Cursor의 크기를 직접 가져오는 기능은 없으므로, Cursor를 사용할 때는 전체 데이터를 순회하며 처리해야한다.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

**fetchSize:** 데이터베이스에서 한 번에 가져올 데이터의 행 수를 설정하는 속성으로 최적화하여 데이터를 가져오는 횟수를 줄임으로써 성능을 향상시킬 수 있다.

{% code lineNumbers="true" %}
```java
    public void findCursorAll(int limit) {

        try (Cursor<TbBatchListDTO> tbBatchListDTOs = tbBatchListMapper.findCursorAll(limit)) {

            tbBatchListDTOs.forEach(tbBatchListDTO -> {
                System.out.println(tbBatchListDTO.toString());
            });

//        Iterator<TbBatchListDTO> iterator = tbBatchListDTOs.iterator();
//        while (iterator.hasNext()) {
//            System.out.println(iterator.next());
//        }
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
    
    @SelectProvider(type = TbBatchListProvider.class,
            method = "findAll")
    @Options(fetchSize=5)
    Cursor<TbBatchListDTO> findCursorAll(int limit);
```
{% endcode %}

*   9 \~ 12 Line: 주석을  풀면 오류가 발생한다.

    <figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
* 쿼리: SELECT BATCH\_SEQ, MEMBER\_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12 FROM TB\_BATCH\_LIST ORDER BY BATCH\_SEQ ASC LIMIT 20
*   결과: SELECT 쿼리가  한번 수행 됨



    <figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### 1-2-2.  Paging

* `LIMIT`, `OFFSET` 쿼리를 사용하여 페이지 단위로 데이터를 구분하여 요청/응답하는 방식이다.
* JVM 메모리에 한 번에 모든 결과를 올리는 것으로 크기를 계산 하여야 한다.

<figure><img src="../.gitbook/assets/image (4) (1).png" alt="" width="563"><figcaption></figcaption></figure>

{% code lineNumbers="true" %}
```java
public void findByBatchSeq(int loopExitCnt, int pageSize)  {

        int batchSeqMin = tbBatchListMapper.getBatchListByBatchSeq("MIN");
    int batchSeqMax = tbBatchListMapper.getBatchListByBatchSeq("MAX");
    int batchSeqLast = 0;
    int loopcnt = 0;
    PagingDTO pagingDTO = PagingDTO.builder()
            .zero(true)
            .start(0)
            .pageSize(pageSize)
            .build();

    do{
        List<TbBatchListDTO> tbBatchListDTOs = tbBatchListMapper.findByBatchSeq(batchSeqMin, pagingDTO);
        long count = tbBatchListDTOs.stream().count();
        TbBatchListDTO batchListDTO =  tbBatchListDTOs.stream().skip(count - 1).findFirst().get();
        batchSeqLast = batchListDTO.getBatchSeq() + 1;
        batchSeqMin = batchSeqLast;
        loopcnt = loopcnt + 1;
        tbBatchListDTOs.forEach(tbBatchListDTO -> {
            System.out.println(tbBatchListDTO.toString());
        });

//            Iterator<TbBatchListDTO> iterator = tbBatchListDTOs.iterator();
//            while (iterator.hasNext()) {
//                System.out.println(iterator.next());
//            }

        if (loopcnt == loopExitCnt) break;
    } while (batchSeqMax > batchSeqLast );
}
    
    
@SelectProvider(type = TbBatchListProvider.class,
            method = "findByBatchSeq")
List<TbBatchListDTO> findByBatchSeq(int batchSeq, PagingDTO pagingDTO);
```
{% endcode %}

* 25 \~ 28 Line: 주석을  풀어도 오류 발생하지 않는다.
* 쿼리: SELECT BATCH\_SEQ, MEMBER\_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12 FROM TB\_BATCH\_LIST WHERE BATCH\_SEQ >= 89797 ORDER BY BATCH\_SEQ ASC LIMIT 0 , 5
*   결과: PageSize 만큼  쿼리 실행 후 결과 리턴 한다.\


    <figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>



## 2. ItemWriter

ItemWriter는 Spring Batch의 요소 중 하나로 데이터를 쓰는 기능을 담당한다. 즉 reader가 File, DB, Message Queue와 같은 것에서 데이터를 읽어들이는 것과 반대로  File, DB, Message Queue에 데이터를 쓰거나 출력하는 역활을 한다.

### 2-1. Chunk 단위 처리

```java
public interface ItemWriter<T> {
    void write(Chunk<? extends T> items) throws Exception;
}
```

Spring Batch의 ItemWriter는 item 하나를 처리 하는 것에서 출발 하여 현재 (Spring Batch v5.1.1)에서는 Chuck 단위로 처리한다.

<figure><img src="../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>

Reader와 Processor에서 처리된 Item을 지정된 Chunk 단위로 모아서 Writer에 보내서 처리한다.

### 2-2. ItemWriter 전략

Spring Batch에서 ItemWriter는 다음과 같은 전략을 제공한다.

* FlatFileItemWriter:  CSV 및 구분된 파일과 같은 플랫 파일에 데이터를 쓰는 데 사용되며 사용자 지정 서식 및 레코드 구분 기호에 대한 지원을 제공한다.
* DB Writer: Database의 영속성(JPA, Hibernate)는과 항상 마지막에 Flush를 해야 하며 모든 Item을 처리 한 후 Spring Batch는 현재 트랜잭션을 커밋한다.
  * JdbcBatchItemWriter: JDBC를 사용하여 관계형 데이터베이스에 데이터를 쓰는 데 사용되며 Batch 기능(한번에 DB에 전달)을 사용하여 성능을 항상 시킬수 있다.
  * JpaItemWriter: JPA 기술을  사용하여 관계형 데이터베이스에 데이터를 쓰는 데 사용한다.
  * MyBatisBatchItemWriter: MyBatis를 사용하여 관계형 데이터베이스에 데이터를 쓰는 데 사용한다.
  * HibernateItemWriter: Hibernate를 사용하여 관계형 데이터베이스에 데이터를 쓰는 데 사용한다.
* JmsItemWriter:  JMS 큐 또는 토픽에 데이터를 쓰는 데 사용되며 분산 처리 및 비동기 메시징을 지원한다.
* StaxEventItemWriter: StAX API를 사용하여 XML 파일에 데이터를 쓰는 데 사용되고. 복잡한 XML 구조를 생성하고 대용량 XML 파일을 처리할 수 있도록 지원한다.

### 2-3. FlatFileItemWriter

텍스트 파일(예: CSV, 고정 너비 파일)에 데이터를 쓰는 데 사용되며. 이 클래스는 **리소스**를 설정하여 파일의 위치를 지정하고, LineAggregator를 사용하여 객체를 문자열로 변환한 다음 파일에 쓰게 된다.

FlatFileItemWriter의 주요 기능은 다음과 같다:

* Resource를 통해 파일의 위치를 지정하고, Writable Resource를 나타내야 한다.
* LineAggregator를 사용하여 객체를 문자열로 변환한다
* FieldExtractor와 LineAggregator를 조합하여, 객체의 필드를 추출하고 이를 기반으로 문자열을 생성한다.
* headerCallback과 footerCallback을 사용하여 파일의 시작과 끝에 헤더와 푸터를 추가할 수 있다.
* append 옵션을 사용하여 파일에 데이터를 추가할지, 덮어쓸지를 결정할 수 있다.

FlatFileItemWriter는 성능을 위해 BufferedWriter를 사용하며, 재시작 가능한 기능도 제공하여 대량의 데이터를 텍스트 파일 형식으로 쉽고 효율적으로 출력할 수 있다.

### 2-4. JdbcBatchItemWriter

<table><thead><tr><th width="174">Property</th><th width="153">Parameter Type</th><th>설명</th></tr></thead><tbody><tr><td>assertUpdates</td><td>boolean</td><td>적어도 하나의 항목이 행을 업데이트하거나 삭제하지 않을 경우 예외를 throw할지 여부를 설정, 기본값은 <code>true</code>, Exception:<code>EmptyResultDataAccessException</code></td></tr><tr><td>columnMapped</td><td>void</td><td>Key,Value 기반으로 Insert SQL의 Values를 매핑 (ex: <code>Map&#x3C;String, Object></code>)</td></tr><tr><td>beanMapped</td><td>void</td><td>Pojo 기반으로 Insert SQL의 Values를 매핑</td></tr></tbody></table>

### 2-5. JpaItemWriter

Java Persistence API (JPA)를 사용하여 데이터베이스에 접근하는 전략으로 데이터베이스에 대한 영속성을 관리하며, flush()와 clear() 메소드를 통해 영속성 컨텍스트를 관리한다.

JpaItemWriter를 사용하기 위해서는  spring-boot-starter-data-jpa 의존성을 가지고 있어야  한다.

JpaItemWriter의 주요 작업은 다음과 같습니다:

* **Chunk 단위**로 처리된 아이템들을 데이터베이스에 일괄적으로 쓰기(flush) 한다.
* **트랜잭션**이 성공적으로 완료되면, 현재 트랜잭션을 커밋(commit)한다.
* **영속성 컨텍스트**를 관리하여, 각 Chunk의 처리가 끝난 후에는 **flush**와 **clear**를 호출하여 JPA의 1차 캐시를 비우고, 데이터베이스와의 동기화를 수행한다.

### 2-6. MyBatisBatchItemWriter

MyBatis를 사용하여 데이터베이스에 대한 배치 작업을 수행하는 것으로 SqlSessionTemplate의 배치 기능을 활용하여 주어진 아이템들을 처리하며  Java 소스 코드 내에 쿼리를 작성하는 대신, MyBatis 설정 파일에 쿼리를 정의하여 사용할 수 있어, 코드의 가독성과 관리가 용이하다.&#x20;

MyBatis를 사용하기 위해서는 mybatis-spring-boot-starter 의존성을 가지고 있어야  한다.

MyBatisBatchItemWriter의 주요 특징은 다음과 같습니다:

* MyBatis의 SQL statement ID를 사용하여, MyBatis설정 파일에 정의된 SQL 문을 실행한다.
* Chunk 단위로 데이터를 처리하며, 각 Chunk가 처리될 때마다 배치로 데이터베이스에 쓰기 작업을 수행한다..
* ExecutorType을 BATCH로 설정하여, Step에서 정의한 FetchSize만큼씩 처리해주는 기능을 가지고 있다.

### 2-7. HibernateItemWriter

Hibernate 세션을 사용하여 현재 Hibernate 세션의 일부가 아닌 엔티티를 저장하거나 업데이트하는 기능으로 Writer는 Chunk 기반 처리에 사용되며, 각 청크의 경계에서 세션을 flush하고, 기본적으로 쓰기 작업 후에 세션을 한다.

HibernateItemWriter의 주요 기능은 다음과 같습니다:

* Chunk 단위로 처리된 아이템들을 데이터베이스에 일괄적으로 저장하거나 업데이트한다..
* 트랜잭션이 성공적으로 완료되면, 현재 트랜잭션을 커밋(commit)한다.
* 영속성 컨텍스트를 관리하여, 각 Chunk의 처리가 끝난 후에는 flush와 clear를 호출하여 Hibernate의 1차 캐시를 비우고, 데이터베이스와의 동기화를 수행한다.

### 2-8. JmsItemWriter

Messaging Service (JMS)를 사용하여 메시지를 보내는 데 사용되며, JmsTemplate을 사용하여 작성된 아이템들을 JMS 대기열에 전송하는 기능을 제공하며 **분산 시스템**에서 메시지 기반의 비동기 통신을 필요로 하는 배치 처리에 매우 유용하다. &#x20;

JmsItemWriter를 사용하면, 배치 처리 과정에서 생성된 데이터를 다른 시스템이나 애플리케이션으로 신뢰성 있게 전송할 수 있다

JmsItemWriter의 주요 기능은 다음과 같습니다:

* **JmsTemplate**을 설정하여, 기본 대상(destination)이 있는 경우 해당 대상으로 아이템들을 전송.
* **Chunk 단위**로 처리된 아이템들을 **JMS 대기열**에 일괄적으로 전송합니다.
* 설정된 속성들이 확정된 후에는 \*\*스레드 안전(thread-safe)\*\*하게 동작한다.

### 2-9. StaxEventItemWriter

XML 파일로 데이터를 쓰기 위해 사용되는 **ItemWriter** 구현체로 StAX (Streaming API for XML)와 Marshaller를 사용하여 객체를 XML로 직렬화하는 기능을 제공한다.  또한 재시작, 통계 및 트랜잭션 기능을 제공하며, 해당 인터페이스를 구현함으로써 이러한 기능을 지원한다.

StaxEventItemWriter의 주요 기능은 다음과 같습니다:

* Resource, marshaller, rootTagName 등을 설정하여 사용한다.
* Marshaller를 통해 Java 객체를 XML로 변환하고, 각 fragment마다 StartDocument와 EndDocument 이벤트를 필터링한다.
* 커스텀 이벤트 writer를 사용하여 Resource에 데이터를 쓰게 한다.
* 헤더와 푸터 콜백을 제공하여, 파일이 열릴 때 한 번 실행되는 헤더를 추가하거나, 모든 아이템이 쓰여진 후 파일을 닫기 전에 푸터를 추가할 수 다.
