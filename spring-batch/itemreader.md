# ItemReader

Spring Batch가 Chunk 지향 처리를 하는데 중요한 역할을 하는것으로 데이터를 읽어들이는 컴포넌트로  다양한 데이터 소스 데이터베이스, 파일, XML, JSON 등이 있으며, 필요에 따라 사용자가 직접 커스텀한 Reader를 만들어 사용할 수도 있다.

## 1. I**temReader**의 종류

주요 **ItemReader** 구현체로는 다음과 같은 것들이 있다.

* **FlatFileItemReader**: 파일 시스템의 플랫 파일에서 데이터를 읽어 오는데 사용한다.
* **Jdbc 관련 ItemReader**
  * **JdbcCursorItemReader**: 데이터베이스에서 JDBC 커서를 사용하여 데이터를 읽어오는 데 사용한다.
  * **JdbcPagingItemReader**: 데이터베이스에서 페이징 쿼리를 사용하여 데이터를 읽어오는 데 사용한다.
* **MyBatisItemReader**
  * **MyBatisCursorItemReader**
  * **MyBatisPagingItemReader**
* **영속성 관련 ItemReader**
  * **JPAItemReader:**
  * **HibernateCursorItemReader**: Hibernate 세션을 통해 데이터베이스에서 데이터를 읽어오는 데 사용한다.
* **MongoItemReader**: MongoDB에서 데이터를 읽어오는 데 사용한다.
* **JmsItemReader**: JMS 큐에서 메시지를 읽어오는 데 사용한다.
* **StaxEventItemReader**: XML 파일에서 데이터를 읽어오는 데 사용한다
