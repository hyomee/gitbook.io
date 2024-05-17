# Thread Safe한 PagingItemReader

JpaPagingItemReader 소스에 비동기를 추가하기 위해 다음과 같이 수정한다.

* public TaskExecutor executor(): 비동기 풀을 만들기 위해 메서드 추가
* POOL\_SIZE = 3: 비동기 풀 크기 지정
* jpaStep: 메서드에서 StepBuilder 객체 생성시 taskExecutor(executor()) 추가
* JpaPagingItemReaderBuilder로 생성 하는 JpaPagingItemReader 객체에 saveState(false)로 설정 해서 오류 발생시 처음 부터 하게 한다. 병렬 수행으로 어디부터 재 시작 할지 판단을 할 수 없으므로 상태 저장을 하지 않는다

{% code lineNumbers="true" %}
```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class JpaItemBatchConfig {
    public static final String JOB_NAME = "JPA_JOB_MULTI";
    public static final int POOL_SIZE = 3;


    private final TbBatchListMapper tbBatchListMapper;


    @Bean
    public Job jpaJob(JobRepository jobRepository, Flow jpaFlow) {
        return new JobBuilder(JOB_NAME, jobRepository)
                .start( jpaFlow)
                .end()
                .build();
    }


    @Bean
    public Flow jpaFlow(Step jpaStep ) {
        return  new FlowBuilder<SimpleFlow>("JPA_FLOW_MULTI")
                .start(jpaStep)
                .build();

    }


    @Bean
    public Step jpaStep(JobRepository jobRepository,
                        PlatformTransactionManager transactionManager,
                        ItemReader jpaPagingItemReader,
                        ItemProcessor jpaItemProcessor,
                        ItemWriter jpaBatchItemWriter) {


        return new StepBuilder("JPA_PAGE_READER_MULTI_CNT", jobRepository)
                .chunk(5, transactionManager)
                .reader(jpaPagingItemReader)
                .processor(jpaItemProcessor)
                .writer(jpaBatchItemWriter)
                .taskExecutor(executor())
                .build();

    }

    @Bean(name = JOB_NAME+"taskPool")
    public TaskExecutor executor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor(); // (2)
        executor.setCorePoolSize(POOL_SIZE);
        executor.setMaxPoolSize(POOL_SIZE);
        executor.setThreadNamePrefix("multi-thread-");
        executor.setWaitForTasksToCompleteOnShutdown(Boolean.TRUE);
        executor.initialize();
        return executor;
    }

    @Bean
    public JpaPagingItemReader<TbBatchListEntity> jpaPagingItemReader(EntityManagerFactory en)  {
        int batchSeq = tbBatchListMapper.getBatchListByBatchSeq("MIN");

        TbBatchListProvider tbBatchListProvider = new TbBatchListProvider();
        String sql = tbBatchListProvider.findJpaAll();

        Map<String, Object> parameterValues = new HashMap<>();
        parameterValues.put("batchSeq", batchSeq);

        JpaNativeQueryProvider<TbBatchListEntity> queryProvider = new JpaNativeQueryProvider<>();
        queryProvider.setSqlQuery(sql);
        queryProvider.setEntityClass(TbBatchListEntity.class);

        return new JpaPagingItemReaderBuilder<TbBatchListEntity>()
                .name("TbBatchListEntity")
                .entityManagerFactory(en)
                .queryProvider(queryProvider)
                .parameterValues(parameterValues)
                .pageSize(5)
                .saveState(false)
                .build();
    }


    @Bean
    public ItemProcessor<TbBatchListEntity, TbBatchListWriteEntity> jpaItemProcessor() {
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
    public ItemWriter<TbBatchListDTO> jpaIitemWriter() {
        return items -> {
            for (TbBatchListDTO item : items) {
                System.out.println(item.toString());
            }
        };
    }

    @Bean
    public ItemWriter<TbBatchListWriteEntity> jpaBatchItemWriter(EntityManagerFactory en) {

        JpaItemWriter<TbBatchListWriteEntity> writer = new JpaItemWriter();
        writer.setEntityManagerFactory(en);

        return writer;
    }

}
```
{% endcode %}

* 결과 : Pool Size 를 3으로 설정 하여 시작과 동시에 3개의 쿼리가 실행된다.

```sql
2024-05-18 01:00:21,518 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)
4. SELECT BATCH_SEQ, MEMBER_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12  
FROM TB_BATCH_LIST  WHERE BATCH_SEQ >= 89787 AND BATCH_SEQ < 89812  
ORDER BY BATCH_SEQ ASC limit 5
 {executed in 31 msec}
2024-05-18 01:00:21,748 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)
4. SELECT BATCH_SEQ, MEMBER_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12  
FROM TB_BATCH_LIST  WHERE BATCH_SEQ >= 89787 AND BATCH_SEQ < 89812  
ORDER BY BATCH_SEQ ASC limit 5,5
 {executed in 19 msec}
2024-05-18 01:00:21,834 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)
4. SELECT BATCH_SEQ, MEMBER_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12  
FROM TB_BATCH_LIST  WHERE BATCH_SEQ >= 89787 AND BATCH_SEQ < 89812  
ORDER BY BATCH_SEQ ASC limit 10,5
 {executed in 22 msec}

2024-05-18 01:00:22,227 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)
4. SELECT BATCH_SEQ, MEMBER_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12  
 FROM TB_BATCH_LIST  WHERE BATCH_SEQ >= 89787 AND BATCH_SEQ < 89812  
 ORDER BY BATCH_SEQ ASC limit 15,5
 {executed in 22 msec}
 
2024-05-18 01:00:22,335 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)
4. SELECT BATCH_SEQ, MEMBER_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12  
FROM TB_BATCH_LIST  WHERE BATCH_SEQ >= 89787 AND BATCH_SEQ < 89812  
ORDER BY BATCH_SEQ ASC limit 20,5
 {executed in 25 msec}
 
2024-05-18 01:00:22,457 DEBUG [jdbc.sqltiming]  com.zaxxer.hikari.pool.ProxyPreparedStatement.executeQuery(ProxyPreparedStatement.java:52)
4. SELECT BATCH_SEQ, MEMBER_NO, ITEM1, ITEM2, ITEM3, ITEM4, ITEM5, ITEM6, ITEM7, ITEM8, ITEM9, ITEM10, ITEM11, ITEM12  
FROM TB_BATCH_LIST  WHERE BATCH_SEQ >= 89787 AND BATCH_SEQ < 89812  
ORDER BY BATCH_SEQ ASC limit 25,5
 {executed in 27 msec}
 
 
 
```
