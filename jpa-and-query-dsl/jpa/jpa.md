---
description: JPA 기본 개념
---

# JPA 기본

## **1. 기본용어** <a href="#id-1" id="id-1"></a>

### **1-1. 영속성(Persitence)** <a href="#id-1-1-persitence" id="id-1-1-persitence"></a>

* 데이터를 생성한 프로그램이 종료되더라도 사라지지 않는 데이터의 특성
* 메모리 상의 데이터를 파일시스템, 관계형 데이터 베이스 혹은 객체 데이터베 이스 등을 활용 하여 영구적으로 저장 하여 영속성을 부여한다.
* 데이터를 데이터베이스에 저장 하는 방법
  * JDBC
  * Spring JDBC
  * Persistence Framework ( Hibernate, Mybatis …

### **1-2. ORM(Obejct Relational Mappings)** <a href="#id-1-2-ormobejct-relational-mappings" id="id-1-2-ormobejct-relational-mappings"></a>

* 객체는 객체대로 설계 하고, 관계형 데이터 베이스는 관계형 데이터베이스로 설계하여
* ORM 프레임워크가 객체와 데이터 베이스의 데이터를 자동으로 매핑(연결)해주는 것이다.
* 객체는 Class를 사용 하고 관계형 데이터베이스는 테이블 사용하며 Persistence API 를 사용해서 코드를 작성한다.
  * 데이터 베이스의 데이터 <- 매핑 -> Object 필드
  * SQL Query가 아닌 직관적인 코드(메서드)로 데이터 조작
  * JPA, Hibernate

### **1-3. Persistencd Layer** <a href="#id-1-3-persistencd-layer" id="id-1-3-persistencd-layer"></a>

* Persistence Layer : 데이터에 영속성을 부여해 주는 계층
* Persistence Framework : JDBC프로그램의 복잡함이나 번거로움 없이 간단한 작업만으로 데이터 베이스와 연동 하여 안정정성을 보장 ( JPA, Hibernate, Mybatis ..)
  * SQL Mapper :
    * 데이터 베이스 객체를 자바 객체로 매핑함으로써 객체 간의 관계를 바탕으로 SQL을 자동 생성 해 주지만 SQL Mapper는 SQL를 명시 해 주어야 한다.
    * SQL <- 매핑 -> Object 필드
    * Mybatis, JdbcTempletes
  * ORM :
    * 데이터 베이스 객체를 자바 객체로 매핑함으로써 객체 간의 관계를 바탕으로 SQL을 자동 생성 해서 자동으로 매핑(연결) 해 주는 것
      * 데이터 베이스 데이터 <- 매핑 -> Object 필드
      * SQL Query가 아닌 직관적인 코드(메서드)로 데이터 조작
      * JPA, Hibernate

## **2. DAO, DTO, Entity 개념** <a href="#id-2-dao-dto-entity" id="id-2-dao-dto-entity"></a>

<figure><img src="../../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

### **2-1. DAO (Data Acess Object)** <a href="#id-2-1-dao-data-acess-object" id="id-2-1-dao-data-acess-object"></a>

실제로 DB에 접근하는 객체로 Service와 DB를 연결하는 고리 역할을 한다 ( Object – SQL CRUD – DB )

<figure><img src="../../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>

### **2-2. DTO (Data Transfer Object)** <a href="#id-2-2-dto-data-transfer-object" id="id-2-2-dto-data-transfer-object"></a>

* 계층간 데이터 교환을 위한 객체(Java Beans)
* 로직을 가지고 있지 않는 순수한 데이터 객체로 getter/setter 메소드만 가지고 있음
* DB에서 꺼낸 값을 임으로 변경할 필요가 없기 때문에 DTO Class에는 setter는 피해야 함 (대신 생성자에서 값을 할당 )
* VO(Value Object)는 DTO와 동일한 개념 이지만 read only 속성을 같음
  * VO는 특정한 비즈니스 값을 담는 객체, DTO는 Layer간의 통신 용도 객체

### **2-3. Entity Class - Domain Model** <a href="#id-2-3-entity-class---domain-model" id="id-2-3-entity-class---domain-model"></a>

* 실제 DB의 테이블과 매칭될 Class
* 테이블과 링크될 Class
* 최대한 외부에서 Entity Class의 getter method를 사용 하지 않도록 해당 Class안에 서 필요한 로직 Method 구현으로 Service Layer에서 해야함

### **2-4. DTO와 Entity 분리** <a href="#id-2-4-dto-entity" id="id-2-4-dto-entity"></a>

* View Layer와 DB Layer분리
* Entity Class의 변경은 여러 Class에 영향을 주는 반면에 View와 통신 하는 DTO는 요청과 응답에 따라서 변경이 되므로
* Domain Model에 Presentation을 위한 필드 추가 시 모델링의 순수성이 깨짐
* Presentation 요구사항들을 수용하기 위해서 여러 Domain Model 사용

## **3. JDBC, JPA, MyBatis 차이점** <a href="#id-3-jdbc-jpa-mybatis" id="id-3-jdbc-jpa-mybatis"></a>

### **3-1. JDBC ( Java DataBase Connectivity )** <a href="#id-3-1-jdbc-java-database-connectivity" id="id-3-1-jdbc-java-database-connectivity"></a>

* DB에 접근 할 수 있도록 JAVA에서 제공 하는 API
* 모든 JAVA의 Data Access 기술의 근간
* 데이터 베이스에서 CRUD

<figure><img src="../../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>

#### **3-1-1.  JAVA JDBC**

1. db 연결
2. Statement 선언
3. insert, update, delete 수행
4. 조회
5. db 연결 해제

```java
Connection connection = driverManager.getConnection(URL, ID, PWD)
Statement statement = connection.createStatement()
statement.executeUpdate(쿼리String)
Result result = statement.executeQuery(쿼리실행)
result.next()를 이용해서 복수 건 처리
finally 에 자원 반납 처리
- result.close()
- statement.close()
- connection.close()
commit, rollback 처리

```

**3-1-2. Spring JDBC**

1. jdbc template 선언
2. jdbc template 객체 생성
3. Jdbc api 실행
4. 조회 결과 처리

\*\* 환경 설정 \*\*

* Connection 설정 관리
* Configuration : DataSource 설정

<figure><img src="../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

```java
private JdbcTemplate jdbcTemplate;

@Autowired
public void setDataSource(DataSource dataSource) {
    this.jdbcTemplate = new JdbcTemplate(datasource);
}
public List<Member> getMembers() {
    return jdbcTemplate.query("쿼리". new RowMapper<Member>() {
        public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
            Member member = new Member();
            member.setXX(rs.getInt("id"));
             
            return member;
        }
    });
}
```

### **3-2. Mybatis** <a href="#id-3-2-mybatis" id="id-3-2-mybatis"></a>

* SQL Mapper
* 자바 객체를 실제 SQL문에 연결
* 자바 POJO를 설정 해서 매핑 하기 위해 XML 또는 Annotation 사용
* 단점
  * 쿼리 반복 개발
  * DB 컬럼 수정 시 -> 영향도 증가에 따른 오류 발생

<figure><img src="../../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>

**3-2-1. Spring Mybatis**

1. 쿼리 작성
2. Mapper Class 작성
3. Service 작성

\*\* 환경 설정 \*\*

* Connection 설정 관리
* Mybatis 설정 관리
* Configuration : DataSource 설정

**환경 설정**

```yaml
mybatis:
    mapper-locations: classpath*:mapper/**/*.xml
    configuration:
        log-prefix: hcfwJdbcLog.

## utl, username, password 암호화 
datasource:
    url: ENC(qsMZcX3awr8F/8mR4gSOM4HRpK5cAPR0r73YATmOiPeId77adM9h15aQAvOiX0Fb)
    username: ENC(++870+xVkBmy675wHhUJSNMK3+Ctix0YlAcDkmnA0vzjESx1S73Xkw==)
    password: ENC(LGC3fp4TgEjNFvDHK6ukjHJz5cPB9Pf5SzNFpgafPvk=)
    driver-class-name: org.h2.Driver


```

#### SQL Query

```xquery
<mapper namespace="com.xxx.samp.dao.MemberDAO">
    <!-- 사용자 목록 조회 -->
    <select id="selectMember" 
            parameterType="com.xxx.samp.dto.MemberDTO"
            resultType="com.xxx.samp.dto.MemberDTO">
        SELECT SEQ, …
          FROM MEMBER
        WHERE SEQ = #{seq}
    </select>
    ....

```

**JAVA 코드**

```java
@Mapper
public interface MemberDAO {
    public MemberDTO selectMember(MemberDTO memberDto);
    public void insertMember(MemberDTO memberDto);
    public void deleteMember(Integer seq);
    public void updateMember(MemberDTO memberDto);
    public List<MemberDTO> selectMembers();
}


public MemberDTO selectMember(MemberDTO memberDto) throws BizException {
    return memberDAO.selectMember(memberDto);
}

```

### **3-3. JPA** <a href="#id-3-3-jpa" id="id-3-3-jpa"></a>

* 자바 ORM 기술에 대한 표준 API 명세로 JAVA에서 제공 하는 API
* ORM을 사용하기 위한 표준 인터페이스를 모아둔 것
* 기존 EJB에서 제공되던 Entity Bean을 대체하는 기술
* 구성 요소
  * javax.persistence 페키지로 정의된 API
  * JPOL(Java Persistence Query Language)
  * 객체/관계 메타 데이터

<figure><img src="../../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

* 장점
  * 객체 지형 개발이 가능 하며, 비즈니스 로직에 집중 할 수 있음
  * 생산성, 유지 보수 용이
  * 테이블 생성, 변경, 관리 쉬움
  * 빠른 개발
* 단점
  * 배우기 어렵다.
  * 잘 이해하고 사용하지 않으면 데이터 손실 발생 ( Persistence context )
  * 잘 이해하고 사용자지 않으면 성능 문제 발생

JPA는 Application과 JDBC사이에서 JDBC API를 사용하여 SQL을 호출 하여 DB와 통신 한다.

<figure><img src="../../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>

