# MyBatis

참고: [https://mybatis.org/spring/batch.html](https://mybatis.org/spring/batch.html)

* MyBatisPagingItemReader
* MyBatisCursorItemReader
* MyBatisBatchItemWriter

<figure><img src="../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure>

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
    public MyBatisPagingItemReader<TbBatchListDTO> myBatisPagingItemReader(SqlSessionFactory sqlSessionFactory) {


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

        // 
        reader.setPageSize(5);
        reader.setSqlSessionFactory(sqlSessionFactory);
        reader.setParameterValues(parameterValues);
        // reader.setQueryId(TbBatchListMapper.class.getName() + ".findByBatchSeq");

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
    
    
    @Bean
    public ItemWriter myBatisBatchItemWriter(SqlSessionFactory sqlSessionFactory) {
        MyBatisBatchItemWriter writer = new MyBatisBatchItemWriter();
        writer.setSqlSessionFactory(sqlSessionFactory);
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
