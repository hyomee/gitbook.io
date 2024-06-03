# MyBatis

참고: [https://mybatis.org/spring/batch.html](https://mybatis.org/spring/batch.html)

* MyBatisPagingItemReader
* MyBatisCursorItemReader
* MyBatisBatchItemWriter

## 1. MyBatisPagingItemReader

<figure><img src="../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure>

{% code lineNumbers="true" %}
```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class MyBatisConfig {

    private final TbBatchListMapper tbBatchListMapper;

    @Bean
    public Job myBatisJob(JobRepository jobRepository, Step myBatisJobStep ) {

        return new JobBuilder("MYBATIS_JOB1", jobRepository)
                .incrementer(new RunIdIncrementer())
                .start(myBatisJobStep)
                .build();
    }

    @Bean
    public Step myBatisJobStep(JobRepository jobRepository,
                                     PlatformTransactionManager transactionManager,
                                     MyBatisPagingItemReader myBatisPagingItemReader,
                                     ItemProcessor myBytisItemProcessor,
                                     ItemWriter myBatisBatchItemWriter) {
        return new StepBuilder("JDBC_MYBATIS_ITEMREADER", jobRepository)
                .<TbBatchListDTO, TbBatchListDTO>chunk(5, transactionManager)
                .reader(myBatisPagingItemReader)
                .processor(myBytisItemProcessor)
                .writer(myBatisBatchItemWriter)
                .build();

    }

    @Bean
    public ItemReader myBatisPagingItemReader(SqlSessionFactory sqlSessionFactory) {


        int batchseq = tbBatchListMapper.getBatchListByBatchSeq("MIN");
        int batchseqMax = tbBatchListMapper.getBatchListWriteByBatchSeq("MAX");

        if (batchseqMax > 0) {
            batchseq = batchseqMax + 1;
        }
        int batchSeqLimit = batchseq +  24;

        // 쿼리 파라메터 설정 
        MyBatisPagingItemReader reader = new MyBatisPagingItemReader();
        Map<String, Object> parameterValues = new HashMap<String, Object>();
        parameterValues.put("batchSeq", batchseq);
        parameterValues.put("batchSeqLimit", batchSeqLimit);

        // pagesize를 설정 하면 자동으로 page 계산함 
        reader.setPageSize(5);
        reader.setSqlSessionFactory(sqlSessionFactory);
        
        // 쿼리 파라메터 연결
        reader.setParameterValues(parameterValues);
        
        // 자바 API 연결시 
        // reader.setQueryId(TbBatchListMapper.class.getName() + ".findByBatchSeq");

        // 쿼리 사용용
        reader.setQueryId("kr.co.abacus.acube.mymapper.MyTbBatchListMapper.findByBatchSeq");
        return reader;
    }


    @Bean
    public ItemProcessor<TbBatchListDTO, TbBatchListDTO> myBytisItemProcessor() {
        return (tbBatchListDTO) -> {
            tbBatchListDTO.setItem1(tbBatchListDTO.getItem1() + "Test ... ");
            return tbBatchListDTO;
        };
    }
    
    // DB 저장 
    @Bean
    public ItemWriter myBatisBatchItemWriter(SqlSessionFactory sqlSessionFactory) {
        MyBatisBatchItemWriter writer = new MyBatisBatchItemWriter();
        writer.setSqlSessionFactory(sqlSessionFactory);
        // 쿼리
        writer.setStatementId("kr.co.abacus.acube.mymapper.MyTbBatchListMapper.insertTbBatchListDTO");
        return writer;
    }

    @Bean
    public ItemWriter<TbBatchListDTO> myBatisitemWriter() {
        return items -> {
            for (TbBatchListDTO item : items) {
                System.out.println(item.toString());
            }
        };
    }

   

}
```
{% endcode %}

* 56 Line: 실행 쿼리 LIMIT page, pagesize&#x20;

```xml
<mapper namespace="kr.co.abacus.acube.mymapper.MyTbBatchListMapper">
    <select id="findByBatchSeq" resultType="TbBatchListDTO">
        <![CDATA[
        SELECT
            BATCH_SEQ,
            MEMBER_NO,
            ITEM1,
            ITEM2,
            ITEM3,
            ITEM4,
            ITEM5,
            ITEM6,
            ITEM7,
            ITEM8,
            ITEM9,
            ITEM10,
            ITEM11,
            ITEM12
        FROM
            TB_BATCH_LIST
        WHERE
            BATCH_SEQ >= #{batchSeq} and BATCH_SEQ < #{batchSeqLimit}
            ORDER BY BATCH_SEQ ASC
            LIMIT
            #{_skiprows},
            #{_pagesize}
        ]]>
    </select>
</mapper>
```

```sql
FROM
    TB_BATCH_LIST
WHERE
    BATCH_SEQ >= 89859 /**P*/ and BATCH_SEQ < 89883 /**P*/
    ORDER BY BATCH_SEQ ASC
    LIMIT
    0 /**P*/,
    5 /**P*/
    
FROM
    TB_BATCH_LIST
WHERE
    BATCH_SEQ >= 89859 /**P*/ and BATCH_SEQ < 89883 /**P*/
    ORDER BY BATCH_SEQ ASC
    LIMIT
    5 /**P*/,
    5 /**P*/
    
    
```

## 2. MyBatisCursorItemReader

페이징(myBatisPagingItemReader)을 Cursor(MyBatisCursorItemReader)로 변경 한 소스 이다.

{% code lineNumbers="true" %}
```java
@Bean
public Step myBatisJobStep(JobRepository jobRepository,
                                 PlatformTransactionManager transactionManager, 
                                 ItemReader myBatisCursorItemReader,
                                 ItemProcessor myBytisItemProcessor,
                                 ItemWriter myBatisBatchItemWriter) {
    return new StepBuilder("JDBC_MYBATIS_ITEMREADER", jobRepository)
            .<TbBatchListDTO, TbBatchListDTO>chunk(5, transactionManager)
            .reader(myBatisCursorItemReader)
            .processor(myBytisItemProcessor)
            .writer(myBatisBatchItemWriter)
            .build();

}

@Bean
public ItemReader  myBatisCursorItemReader(SqlSessionFactory sqlSessionFactory) {


    int batchseq = tbBatchListMapper.getBatchListByBatchSeq("MIN");
    int batchseqMax = tbBatchListMapper.getBatchListWriteByBatchSeq("MAX");

    if (batchseqMax > 0) {
        batchseq = batchseqMax + 1;
    }
    int batchSeqLimit = batchseq +  24;

    MyBatisCursorItemReader reader = new MyBatisCursorItemReader();
    Map<String, Object> parameterValues = new HashMap<String, Object>();
    parameterValues.put("batchSeq", batchseq);
    parameterValues.put("batchSeqLimit", batchSeqLimit);

    reader.setSqlSessionFactory(sqlSessionFactory);
    reader.setParameterValues(parameterValues);

    reader.setQueryId("kr.co.abacus.acube.mymapper.MyTbBatchListMapper.findCursorByBatchSeq");
    return reader;
}
```
{% endcode %}

* 4 Line: MyBatisCursorItemReader로 변경
* 36 Line: Cursor 쿼리 사용 변경
* 결과 : 쿼리는 한번 실행 되고 Chunk  Size만큼 읽어서 처리 한다.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
