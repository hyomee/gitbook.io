# ItemReader

Spring Batch가 Chunk 지향 처리를 하는데 중요한 역할을 하는것으로 데이터를 읽어들이는 컴포넌트로  다양한 데이터 소스 데이터베이스, 파일, XML, JSON 등이 있으며, 필요에 따라 사용자가 직접 커스텀한 Reader를 만들어 사용할 수도 있다.

## 1. I**temReader**의 종류

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

## 2.  Cursor 과 Paging 차이점&#x20;

### 2-1.  Cursor

* 배치 처리가 완료될때까지 DB Connection이 연결한다.
* 하나의 Connection에서 처리 되기 때문에 Thread Safe하지 않다.
* 데이터베이스 결과 집합을 한 줄씩 읽어오는 구조로 데이터를 순차적으로 처리를 해야 한다.&#x20;
* Cursor를 사용하면 JVM 메모리에 한 번에 모든 결과를 올려둘 필요가 없으므로, 대량 데이터를 효율적으로 처리할 수 있다.
* Cursor의 크기를 직접 가져오는 기능은 없으므로, Cursor를 사용할 때는 전체 데이터를 순회하며 처리해야한다.

<figure><img src="../../.gitbook/assets/image (1).png" alt="" width="563"><figcaption></figcaption></figure>

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

    <figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>
* 쿼리: SELECT BATCH\_SEQ, MEMBER\_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12 FROM TB\_BATCH\_LIST ORDER BY BATCH\_SEQ ASC LIMIT 20
*   결과: SELECT 쿼리가  한번 수행 됨



    <figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### 2-1.  Paging

* `LIMIT`, `OFFSET` 쿼리를 사용하여 페이지 단위로 데이터를 구분하여 요청/응답하는 방식이다.
* JVM 메모리에 한 번에 모든 결과를 올리는 것으로 크기를 계산 하여야 한다.

<figure><img src="../../.gitbook/assets/image (4).png" alt="" width="563"><figcaption></figcaption></figure>

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


    <figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
