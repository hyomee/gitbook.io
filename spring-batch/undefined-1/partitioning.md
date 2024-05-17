# Partitioning

**Partitioner**를 사용하여 분할된 범위를 각 Step에 전달하여 병렬처리하는 방식으로 각 분할 실행을 PartitionHandler를 통해 각 Step를 실행한다.

참고 : [https://docs.spring.io/spring-batch/reference/scalability.html](https://docs.spring.io/spring-batch/reference/scalability.html)

<figure><img src="../../.gitbook/assets/image (267).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:purple;">**Step Manager를 통해서 Step을 분할(GridSize)하고 각 Step을 처리할 비동기 Executer, Step에서 사용할 값을 stepExecutionContext에 저장 한다.**</mark>

<figure><img src="../../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

```java
@Bean
public Flow jpaPartitionFlow(Step partitionManager) {
    return  new FlowBuilder<SimpleFlow>("JPA_FLOW_PARTI")
            .start(partitionManager)
            .build();

}
    
@Bean
public Step partitionManager(JobRepository jobRepository,
                         TaskExecutorPartitionHandler partitionHandler,
                         JpaPartitionBatchPartitioner partitioner,
                         Step jpaPartitionStep) {
    return new StepBuilder("JPA_PAGE_READER_PARTI_MNA", jobRepository)
            .partitioner("JPA_PAGE_READER_PARTI_jpaPartitionStep", partitioner)
            .step(jpaPartitionStep)
            .partitionHandler(partitionHandler)
            .build();
}

@Bean
public TaskExecutorPartitionHandler partitionHandler(Step jpaPartitionStep) {
    TaskExecutorPartitionHandler partitionHandler = new TaskExecutorPartitionHandler(); // (1)
    partitionHandler.setStep(jpaPartitionStep); // (2)
    partitionHandler.setTaskExecutor(partitionExecutor()); // (3)
    partitionHandler.setGridSize(10); // (4)
    return partitionHandler;
}

```

```java
public class JpaPartitionBatchPartitioner implements Partitioner {
    @Override
    public Map<String, ExecutionContext> partition(int gridSize) {
        Map<String, ExecutionContext> result = new HashMap<>();
        int number = 0;

        for (int i = 0; i < gridSize; i++) {
            ExecutionContext value = new ExecutionContext();
            result.put("partition" + number, value);

            value.putLong("remainder", i); // 각 파티션마다 사용될 remainder
            number++;
        }

        return result;
    }
}
```

* <mark style="color:purple;">**그외 Step의 모든 기능은 Spring Batch에서 제공하는 기능을 재사용 할 수 있다. 즉 단일 서비스로 개발한 코드를 병행 처리에서도 재사용 가능하여 성능에 대응 할 수 있다.**</mark>

<details>

<summary>전체 소스</summary>

```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class JpaPartitionBatchConfig {
    public static final String JOB_NAME = "JPA_JOB_PARTITION";
    public static final int POOL_SIZE = 11;


    private final TbBatchListMapper tbBatchListMapper;


    @Bean
    public Job jpaPartitionJob(JobRepository jobRepository, Flow jpaPartitionFlow) {
        return new JobBuilder(JOB_NAME, jobRepository)
                .start( jpaPartitionFlow)
                .end()
                .build();
    }


    @Bean
    public Flow jpaPartitionFlow(Step partitionManager) {
        return  new FlowBuilder<SimpleFlow>("JPA_FLOW_PARTI")
                .start(partitionManager)
                .build();

    }

    @Bean
    public Step partitionManager(JobRepository jobRepository,
                             TaskExecutorPartitionHandler partitionHandler,
                             JpaPartitionBatchPartitioner partitioner,
                             Step jpaPartitionStep) {
        return new StepBuilder("JPA_PAGE_READER_PARTI_MNA", jobRepository)
                .partitioner("JPA_PAGE_READER_PARTI_jpaPartitionStep", partitioner)
                .step(jpaPartitionStep)
                .partitionHandler(partitionHandler)
                .build();
    }

    @Bean
    public TaskExecutorPartitionHandler partitionHandler(Step jpaPartitionStep) {
        TaskExecutorPartitionHandler partitionHandler = new TaskExecutorPartitionHandler(); // (1)
        partitionHandler.setStep(jpaPartitionStep); // (2)
        partitionHandler.setTaskExecutor(partitionExecutor()); // (3)
        partitionHandler.setGridSize(10); // (4)
        return partitionHandler;
    }

    @Bean
    public Step jpaPartitionStep(JobRepository jobRepository,
                        PlatformTransactionManager transactionManager,
                        ItemReader jpaPartitionPagingItemReader,
                        ItemProcessor jpaPartitionItemProcessor,
                        ItemWriter jpaPartitionBatchItemWriter) {


        return new StepBuilder("JPA_PAGE_READER_PARTI_CNT", jobRepository)
                .chunk(5, transactionManager)
                .reader(jpaPartitionPagingItemReader)
                .processor(jpaPartitionItemProcessor)
                .writer(jpaPartitionBatchItemWriter)
                .taskExecutor(partitionExecutor())
                .build();

    }

    @Bean
    public JpaPartitionBatchPartitioner partitioner() {
        return new JpaPartitionBatchPartitioner();
    }

    @Bean
    public TaskExecutor partitionExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor(); // (2)
        executor.setCorePoolSize(POOL_SIZE);
        executor.setMaxPoolSize(POOL_SIZE);
        executor.setThreadNamePrefix("multi-thread-");
        executor.setWaitForTasksToCompleteOnShutdown(Boolean.TRUE);
        executor.initialize();
        return executor;
    }

    @Bean
    @StepScope
    public JpaPagingItemReader<TbBatchListEntity> jpaPartitionPagingItemReader(EntityManagerFactory en,
                                  @Value("#{stepExecutionContext[remainder]}") int remainder ) {
        int batchSeq = tbBatchListMapper.getBatchListByBatchSeq("MIN");

        TbBatchListProvider tbBatchListProvider = new TbBatchListProvider();
        String sql = tbBatchListProvider.findJpaModAll();

        Map<String, Object> parameterValues = new HashMap<>();
        parameterValues.put("batchSeq", batchSeq);
        parameterValues.put("division", 10);
        parameterValues.put("remainder", remainder);

        log.info("reader division={}, remainder={}", 10, remainder);
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
    public ItemProcessor<TbBatchListEntity, TbBatchListWriteEntity> jpaPartitionItemProcessor() {
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
    public ItemWriter<TbBatchListDTO> jpaPartitionIitemWriter() {
        return items -> {
            for (TbBatchListDTO item : items) {
                System.out.println(item.toString());
            }
        };
    }

    @Bean
    public ItemWriter<TbBatchListWriteEntity> jpaPartitionBatchItemWriter(EntityManagerFactory en) {

        JpaItemWriter<TbBatchListWriteEntity> writer = new JpaItemWriter();
        writer.setEntityManagerFactory(en);

        return writer;
    }
}
```

</details>
