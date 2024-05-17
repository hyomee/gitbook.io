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
