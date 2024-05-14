# Jdbc ItemReader

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

<table><thead><tr><th width="206">메서드</th><th>설명</th></tr></thead><tbody><tr><td>name</td><td>ItemReader 이름</td></tr><tr><td>fetchSize</td><td>한번에 읽어올 데이터 갯수<br>- chunk size와 동일하게 설정하는 것을 추천한다. 모두 5라고 가정했을때, 5개의 데이터를 한번에 가져와서 5개씩 Commit을 수행하게된다.</td></tr><tr><td>sql</td><td>실행할 쿼리</td></tr><tr><td>beanRowMapper</td><td>객체와 자동으로 매핑해주는 Mapper</td></tr><tr><td>queryArguments</td><td>sql 쿼리에 사용될 쿼리 파라미터 설정</td></tr><tr><td>maxItemCount</td><td>조회할 최대 아이템 갯수</td></tr><tr><td>currentItemCount</td><td>조회 Item의 시작 시점<br>- 현재 ItemCount 갯수를 센다. MaxItemCount와 연동되어 사용되는데 만약 MaxItemCount이 20이고, CurrentItemCount가 20이면 더이상 읽어올 데이터가 없다.</td></tr><tr><td>maxRows</td><td>ResultSet이 포함할 수 있는 최대 row 수</td></tr><tr><td>dataSource</td><td>연결할 DB의 dataSource</td></tr></tbody></table>

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
