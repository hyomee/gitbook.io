# Jdbc

JdbcCursorItemReader, JdbcPagingItemReader를 사용하며 두 클래스 모두 객체에 beanMapper, rowMapper 설정이 가능하다.

**Jdbc 관련 ItemReader 종류**

* **JdbcCursorItemReader**: 데이터베이스에서 JDBC 커서를 사용하여 데이터를 읽어오는 데 사용한다.
* **JdbcPagingItemReader**: 데이터베이스에서 페이징 쿼리를 사용하여 데이터를 읽어오는 데 사용한다.

## 1. **JdbcCursorItemReader**

Spring Batch에서 Cursor기반의 JDBC 구현채로 데이터베이스에서 데이터를 Streaming 방식으로 읽어오며 다음과 같은 특징이 있다.

* JDBC ResultSet의 **기본 메커니즘을 활용**하여 현재 행에 커서를 유지하며 다음 데이터를 호출하면 다음 행으로 커서를 이동하며 데이터를 반환한다.
* DB Connection이 연결된 상태에서 테이터를 읽어오기 때문에  **DB와 SocketTimeout을 충분한 값으로 설정**하고 모든 결과를 메모리에 할당하므로 **메모리 사용량이 많아진다**.
* 멀티 스레드 환경에서 **Thread 안정성을 보장을 보장하지 못하므로** 동시성 이슈를 피하기 위해 별도의 동기화 처리가 필요하다

### 1-1. 속성

<table><thead><tr><th width="206">메서드</th><th>설명</th></tr></thead><tbody><tr><td>name</td><td>ItemReader 이름</td></tr><tr><td>dataSource</td><td>연결할 DB의 dataSource</td></tr><tr><td>queryArguments</td><td>sql 쿼리에 사용될 쿼리 파라미터 설정</td></tr><tr><td>sql</td><td>실행할 쿼리</td></tr><tr><td>beanRowMapper</td><td>객체와 자동으로 매핑해주는 Mapper<br><strong>new BeanPropertyRowMapper&#x3C;>(TbBatchListDTO.class)</strong></td></tr><tr><td>rowMapper</td><td>ResultSet을 객체에 매핑해주는 Mapper로 RowMapper를  상속받아 구현해야 한다.</td></tr><tr><td>maxRows</td><td>ResultSet이 포함할 수 있는 최대 row 수</td></tr><tr><td>fetchSize</td><td>한번에 읽어올 데이터 갯수로 commit단위로 보면 chunk size와 동일 하게 설정하는 것을 좋다.</td></tr><tr><td>maxItemCount</td><td>조회할 최대 아이템 갯수 ( 처리할 최대 수 )</td></tr><tr><td>currentItemCount</td><td>조회 Item의 시작 시점<br>- 현재 ItemCount 갯수를 센다. MaxItemCount와 연동되어 사용되는데 만약 MaxItemCount이 20이고, CurrentItemCount가 20이면 더이상 읽어올 데이터가 없다.</td></tr></tbody></table>

### 1-2. 예제

```java
@Bean
public JdbcCursorItemReader<TbBatchListDTO> jdbcCursorItemReader(DataSource dataSource,
                                                                 TbBatchListDTOResultMapper tbBatchListDTOResultMapper)  {

    JdbcCursorItemReader<TbBatchListDTO> itemReader = new JdbcCursorItemReader<>();
    itemReader.setDataSource(dataSource);
    itemReader.setSql(tbBatchListDTOResultMapper.cursorQueryFindAll(20));
    itemReader.setRowMapper(tbBatchListDTOResultMapper);
    itemReader.setMaxRows(5);
    itemReader.setFetchSize(5);
    itemReader.setQueryTimeout(10000);
    return itemReader;
}

```

```java
@Component
public class TbBatchListDTOResultMapper implements RowMapper<TbBatchListDTO> {

    @Override
    public TbBatchListDTO mapRow(ResultSet rs, int rowNum) throws SQLException {
        TbBatchListDTO tbBatchListDTO = new TbBatchListDTO();


        return TbBatchListDTO.builder()
                .batchSeq(rs.getInt("BATCH_SEQ"))
                .memberNo(rs.getString("MEMBER_NO"))
                .item1(rs.getString("ITEM1"))
                .item2(rs.getString("ITEM2"))
                .item3(rs.getString("ITEM3"))
                .item4(rs.getString("ITEM4"))
                .item5(rs.getString("ITEM5"))
                .item6(rs.getString("ITEM6"))
                .item7(rs.getString("ITEM7"))
                .item8(rs.getString("ITEM8"))
                .item9(rs.getString("ITEM9"))
                .item10(rs.getString("ITEM10"))
                .item11(rs.getString("ITEM11"))
                .item12(rs.getString("ITEM12"))
                .build();
    }


    public String cursorQueryFindAll(int limit) {
        TbBatchListProvider query = new TbBatchListProvider();
        return query.findAll(limit);
    }
}
```

## 2. **JdbcPagingItemReader**

**페이징 기반의 JDBC 구현체로** 데이터베이스에서 페이징 단위로 데이터를 조회하는 방식으로 동작한다.  즉  여러 페이지를 읽을 때마다 새로운 쿼리를 실행하므로 결과 데이터의 순서가 보장될 수 있도록 `order by` 구문이 작성하여야 한다. 또한 **멀티스레드 환경에서 Thread 안정성을 보장**하므로 별도의 동기화가 필요하지 않는다.

### 2-1. 속성

<table><thead><tr><th width="295">method</th><th>설명</th></tr></thead><tbody><tr><td>name</td><td>ItemReader 이름</td></tr><tr><td>fetchSize</td><td>한번에 읽어올 데이터 갯수로  chunkSize, pageSize, fetchSize 를 가급적 동일하게 설정해서 사용하는 것이 좋다.</td></tr><tr><td>pageSize</td><td>페이지 크기 설정 (쿼리당 요청할 레코드 수)<br>- fetchSize 와 pageSize 가 다를 경우 DB 에 따라 차이가 있겠지만 내부적으로 pageSize 만큼 분할 처리가 완료된 이후 fetchSize 만큼 행단위로 읽어오는 식으로 처리</td></tr><tr><td>dataSource</td><td>DB에 접근하기 위해 Datasource 설정</td></tr><tr><td>queryProvider</td><td>DB 페이징 전략에 따른 PagingQueryProvider 설정</td></tr><tr><td>rowMapper</td><td>쿼리 결과로 반환되는 데이터와 객체를 매핑하기 위한 RowMapper 설정</td></tr><tr><td>(QueryProvider API) selectClause</td><td>select절 설정</td></tr><tr><td>(QueryProvider API) fromClause</td><td>from절 설정</td></tr><tr><td>(QueryProvider API) whereClause</td><td>where절 설정</td></tr><tr><td>(QueryProvider API) groupClause</td><td>group절 설정</td></tr><tr><td>(QueryProvider API) sortKeys</td><td>정렬을 위한 유니크 키 설정 (HashMap 형태)</td></tr><tr><td>parameterValues</td><td>쿼리 파라미터 설정 (Map 형태)</td></tr><tr><td>maxItemCount</td><td>조회할 최대 item 수</td></tr><tr><td>currentItemCount</td><td>조회 item의 시작 지점</td></tr><tr><td>maxRows</td><td>ResultSet 오프젝트가 포함할 수 있는 최대 행 수</td></tr></tbody></table>

### 2-2. 예제

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class JdbcItemReader {

    // Job 정의
    @Bean
    public Job jdbcCursorJob(JobRepository jobRepository, Step jdbcItemReaderStep) {
        return new JobBuilder("JDBC_CURSOR_JOB1", jobRepository)
                .incrementer(new RunIdIncrementer())
                .start(jdbcItemReaderStep)
                .build();
    }

    // Step 정의
    @Bean
    public Step jdbcItemReaderStep(JobRepository jobRepository,
                               PlatformTransactionManager transactionManager,
                               JdbcPagingItemReader itemReader,
                               ItemProcessor itemProcessor,
                               ItemWriter fileWriter) {
        return new StepBuilder("JDBC_CURSOR_ITEMREADER", jobRepository)
                .chunk(5, transactionManager)
                .reader(itemReader)
                .processor(itemProcessor)
                .writer(fileWriter)
                .build();

    }

    // 페이지 READER
    @Bean
    public JdbcPagingItemReader<TbBatchListDTO> jdbcPagingTbBatchListItemReader(DataSource dataSource,
                                                                                PagingQueryProvider createQueryProvider) throws Exception {

        return new JdbcPagingItemReaderBuilder<TbBatchListDTO>()
                .name("JDBC_PAGING_ITEMREADER")
                .queryProvider(createQueryProvider)
                .dataSource(dataSource)
                .pageSize(5)
                .fetchSize(5)
                .maxItemCount(3)
                .rowMapper(new BeanPropertyRowMapper<>(TbBatchListDTO.class))                
                .build();
    }

    // 쿼리 생성
    @Bean
    public PagingQueryProvider createQueryProvider(DataSource dataSource) throws Exception {
        SqlPagingQueryProviderFactoryBean queryProvider = new SqlPagingQueryProviderFactoryBean();
        queryProvider.setDataSource(dataSource);
        queryProvider.setSelectClause("BATCH_SEQ,MEMBER_NO,ITEM1,ITEM2,ITEM3,ITEM4,ITEM5,ITEM6,ITEM7,ITEM8,ITEM9,ITEM10,ITEM11,ITEM12");
        queryProvider.setFromClause("FROM TB_BATCH_LIST");
        queryProvider.setWhereClause(" BATCH_SEQ < 89807 ");

        Map<String, Order> sortKeys = new HashMap<>(1);
        sortKeys.put("BATCH_SEQ", Order.ASCENDING);
        queryProvider.setSortKeys(sortKeys);

        return queryProvider.getObject();
    }

    @Bean
    public ItemProcessor<TbBatchListDTO, TbBatchListDTO> itemProcessor() {
        return (tbBatchListDTO) -> {
            tbBatchListDTO.setItem1(tbBatchListDTO.getItem1() + "Test ... ");
            return tbBatchListDTO;
        };
    }


    @Bean
    public ItemWriter<TbBatchListDTO> itemWriter() {
        return items -> {
            for (TbBatchListDTO item : items) {
                System.out.println(item.toString());
            }
        };
    }

    // CSV 파일 생성
    @Bean
    public FlatFileItemWriter<TbBatchListDTO> fileWriter() {
        FlatFileItemWriter<TbBatchListDTO> writer = new FlatFileItemWriter<>();
        writer.setResource(new FileSystemResource("D:\\ABACUS_PRJ\\abacus\\acube-svc-batch\\file\\out\\TbBatchListDTO.csv"));
        writer.setLineAggregator(getDelimitedLineAggregator());
        return writer;
    }


    // CSV 파일 포맷
    private DelimitedLineAggregator<TbBatchListDTO> getDelimitedLineAggregator() {
        BeanWrapperFieldExtractor<TbBatchListDTO> beanWrapperFieldExtractor = new BeanWrapperFieldExtractor<>();
        beanWrapperFieldExtractor.setNames(new String[]{"batchSeq",
                "memberNo",
                "item1",
                "item2",
                "item3",
                "item4",
                "item5",
                "item6",
                "item7",
                "item8",
                "item9",
                "item10",
                "item11",
                "item12" });

        DelimitedLineAggregator<TbBatchListDTO> aggregator = new DelimitedLineAggregator<>();
        aggregator.setDelimiter(",");
        aggregator.setFieldExtractor(beanWrapperFieldExtractor);
        return aggregator;

    }
}
```
