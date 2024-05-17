# Jpa

JpaItemWriter는 JPA를 사용하여 데이터베이스에 영속화하는데 사용되는 것으로 다음과 같은 기능을 한다.

* Chunk 단위로 처리된 아이템들을 데이터베이스에 일괄적으로 쓰기 (flush)한다.
* 트랜잭션이 성공적으로 완료되면, 현재 트랜잭션을 커밋한다.



<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<pre class="language-java" data-line-numbers><code class="lang-java">@Configuration
@RequiredArgsConstructor
@Slf4j
public class JpaItemBatchConfig {

    private final TbBatchListMapper tbBatchListMapper;

    // job
    @Bean
    public Job jpaJob(JobRepository jobRepository, Flow jpaFlow) {
        return new JobBuilder("JPA_JOB1", jobRepository)
                .start( jpaFlow)
                .end()
                .build();
    }


    // flow
    @Bean
    public Flow jpaFlow(Step jpaStep ) {
        return  new FlowBuilder&#x3C;SimpleFlow>("JPA_FLOW")
                .start(jpaStep)
                .build();

    }

    // step : chunk job
    @Bean
    public Step jpaStep(JobRepository jobRepository,
                        PlatformTransactionManager transactionManager, 
                        ItemReader jpaPagingItemReader,
                        ItemProcessor jpaItemProcessor,
                        ItemWriter jpaBatchItemWriter) {
        return new StepBuilder("JPA_PAGE_READER", jobRepository)
                .chunk(5, transactionManager)
                .reader(jpaPagingItemReader)
                .processor(jpaItemProcessor)
                .writer(jpaBatchItemWriter)
                .build();

    }


<strong>    @Bean
</strong>    public JpaPagingItemReader&#x3C;TbBatchListEntity> jpaPagingItemReader(EntityManagerFactory en)  {
        int batchSeq = tbBatchListMapper.getBatchListByBatchSeq("MIN");

        // 쿼리 
        TbBatchListProvider tbBatchListProvider = new TbBatchListProvider();
        String sql = tbBatchListProvider.findJpaAll();

        Map&#x3C;String, Object> parameterValues = new HashMap&#x3C;>();
        parameterValues.put("batchSeq", batchSeq);
        
        // JpaNativeQueryProvider을 통한 쿼리 설정 
        JpaNativeQueryProvider&#x3C;TbBatchListEntity> queryProvider = new JpaNativeQueryProvider&#x3C;>();
        queryProvider.setSqlQuery(sql);
        queryProvider.setEntityClass(TbBatchListEntity.class);

        // JpaPagingItemReaderBuilder를 통한 JpaPagingItemReader 생성
        return new JpaPagingItemReaderBuilder&#x3C;TbBatchListEntity>()
                .name("TbBatchListEntity")
                .entityManagerFactory(en)
                .queryProvider(queryProvider)
                .parameterValues(parameterValues)
                .pageSize(5)
                .saveState(true)
                .build();
    }
    
    
    // 
    @Bean
    public ItemProcessor&#x3C;TbBatchListEntity, TbBatchListWriteEntity> jpaItemProcessor() {
        return (tbBatchListEntity) -> {
             TbBatchListDTO tbBatchListDTO =
                     JpaMapper.INSTANCE.tbBatchListEntityToTbBatchListDTO(tbBatchListEntity);
             TbBatchListWriteEntity tbBatchListWriteEntity =
                     JpaMapper.INSTANCE.tbBatchListDTOToTbBatchListWriteEntity(tbBatchListDTO);

            tbBatchListWriteEntity.setBatchSeq(0);
            tbBatchListWriteEntity.setBatchSeqList(tbBatchListDTO.getBatchSeq());

            return tbBatchListWriteEntity;
        };
    }

    @Bean
    public ItemWriter&#x3C;TbBatchListDTO> jpaIitemWriter() {
        return items -> {
            for (TbBatchListDTO item : items) {
                System.out.println(item.toString());
            }
        };
    }

    @Bean
    public ItemWriter&#x3C;TbBatchListWriteEntity> jpaBatchItemWriter(EntityManagerFactory en) {

        JpaItemWriter&#x3C;TbBatchListWriteEntity> writer = new JpaItemWriter();
        writer.setEntityManagerFactory(en);

        return writer;
    }

}
</code></pre>

* 실행하면 다음 쿼리와 같이 5개 조회 하여 처리하는 것을 확인 할수 있다.

```sql
2024-05-18 00:27:20,756
SELECT BATCH_SEQ, MEMBER_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12  
  FROM TB_BATCH_LIST  
 WHERE BATCH_SEQ >= 89787 AND BATCH_SEQ < 89812  ORDER BY BATCH_SEQ ASC limit 5 
 
2024-05-18 00:27:21,177  
SELECT BATCH_SEQ, MEMBER_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12  
  FROM TB_BATCH_LIST  
WHERE BATCH_SEQ >= 89787 AND BATCH_SEQ < 89812  ORDER BY BATCH_SEQ ASC limit 5,5

.......
 

```

```
2. SELECT BATCH_SEQ, MEMBER_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12  FROM TB_BATCH_LIST  WHERE BATCH_SEQ >= 89787 AND BATCH_SEQ < 89812  ORDER BY BATCH_SEQ ASC limit 5
 {executed in 20 msec}
2024-05-18 00:43:09,172  .....ItemProcessor.process(Object) executed in 0ms
2024-05-18 00:43:09,173  .....ItemProcessor.process(Object) executed in 0ms
2024-05-18 00:43:09,173  .....ItemProcessor.process(Object) executed in 0ms
2024-05-18 00:43:09,173  .....ItemProcessor.process(Object) executed in 0ms
2024-05-18 00:43:09,173  .....ItemProcessor.process(Object) executed in 0ms
2024-05-18 00:43:09,218 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)
1. insert into TB_BATCH_LIST_WRITE (BATCH_SEQ_LIST,ITEM1,ITEM10,ITEM11,ITEM12,ITEM2,ITEM3,ITEM4,ITEM5,ITEM6,ITEM7,ITEM8,ITEM9,MEMBER_NO) values (89787,'아이템 1 : 0','아이템 10 : 0','아이템 11 : 0','아이템 12 : 0','아이템 2 : 0','아이템 3 : 0','아이템 4 : 0','아이템 5 : 0','아이템 6 : 0','아이템 7 : 0','아이템 8 : 0','아이템 9 : 0','0')
 {executed in 18 msec}
2024-05-18 00:43:09,249 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)
1. insert into TB_BATCH_LIST_WRITE (BATCH_SEQ_LIST,ITEM1,ITEM10,ITEM11,ITEM12,ITEM2,ITEM3,ITEM4,ITEM5,ITEM6,ITEM7,ITEM8,ITEM9,MEMBER_NO) values (89788,'아이템 1 : 1','아이템 10 : 1','아이템 11 : 1','아이템 12 : 1','아이템 2 : 1','아이템 3 : 1','아이템 4 : 1','아이템 5 : 1','아이템 6 : 1','아이템 7 : 1','아이템 8 : 1','아이템 9 : 1','1')
 {executed in 21 msec}
2024-05-18 00:43:09,271 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)
1. insert into TB_BATCH_LIST_WRITE (BATCH_SEQ_LIST,ITEM1,ITEM10,ITEM11,ITEM12,ITEM2,ITEM3,ITEM4,ITEM5,ITEM6,ITEM7,ITEM8,ITEM9,MEMBER_NO) values (89789,'아이템 1 : 2','아이템 10 : 2','아이템 11 : 2','아이템 12 : 2','아이템 2 : 2','아이템 3 : 2','아이템 4 : 2','아이템 5 : 2','아이템 6 : 2','아이템 7 : 2','아이템 8 : 2','아이템 9 : 2','2')
 {executed in 18 msec}
2024-05-18 00:43:09,290 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)
1. insert into TB_BATCH_LIST_WRITE (BATCH_SEQ_LIST,ITEM1,ITEM10,ITEM11,ITEM12,ITEM2,ITEM3,ITEM4,ITEM5,ITEM6,ITEM7,ITEM8,ITEM9,MEMBER_NO) values (89790,'아이템 1 : 3','아이템 10 : 3','아이템 11 : 3','아이템 12 : 3','아이템 2 : 3','아이템 3 : 3','아이템 4 : 3','아이템 5 : 3','아이템 6 : 3','아이템 7 : 3','아이템 8 : 3','아이템 9 : 3','3')
 {executed in 18 msec}
2024-05-18 00:43:09,312 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)
1. insert into TB_BATCH_LIST_WRITE (BATCH_SEQ_LIST,ITEM1,ITEM10,ITEM11,ITEM12,ITEM2,ITEM3,ITEM4,ITEM5,ITEM6,ITEM7,ITEM8,ITEM9,MEMBER_NO) values (89791,'아이템 1 : 4','아이템 10 : 4','아이템 11 : 4','아이템 12 : 4','아이템 2 : 4','아이템 3 : 4','아이템 4 : 4','아이템 5 : 4','아이템 6 : 4','아이템 7 : 4','아이템 8 : 4','아이템 9 : 4','4')
 {executed in 19 msec}
2024-05-18 00:43:09,334 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)
.....
```
