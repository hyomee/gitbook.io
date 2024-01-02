# Entity 매핑 및 전략

JPA Entity 매핑이란 JPA가 관리하는 클래스와 데이터베이스의 테이블을 연결하는 것입니다. JPA는 다양한 어노테이션을 사용하여 엔티티 클래스의 속성과 테이블의 컬럼, 기본키, 외래키, 관계 등을 매핑할 수 있다.

## **1. @Entity** <a href="#id-1-entity" id="id-1-entity"></a>

* 테이블과 매핑할 Class
* Default : Class Name과 같은 보통 클래스와 같은 이름
* 변경할 때는 @Entity(name = “tableName”)
* 기본 생성자 필수 ( 파라메터 없는 public or peotected )
* final 생성자, enum, interface, inner class 사용 금지
* 지정할 필드에 final 사용 금지

## **2. @Table** <a href="#id-2-table" id="id-2-table"></a>

* 앤티티와 매핑할 Table 지정
* 생략시 엔티티이름을 테이블 이름으로 사용
* 속성
  * name : 매핑할 테이블 이름
  * catalog : catalog 기능이 있는 DB에서 catalog
  * schema : schema 기능이 있는 DB에서 schma
  * uniqueConstraints(DDL) : DDL 생성시 제약 조건

```
@Table(name="TB_MEMNBER",
         uniqueConstraints={@uniqueConstraints
                                name = "TB_MEMNBER_UNIQE"
                                column = { "id", "name" }})

=>   
 ALTER TABLE TB_MEMNBER ADD CONSTRAINT
     TB_MEMNBER_UNIQE UNIQE(ID, NAME)
```

## **3. @id** <a href="#id-3-id" id="id-3-id"></a>

* 엔티티의 주키 ( 기본키 매핑 )

## **4. @GenerateValue** <a href="#id-4-generatevalue" id="id-4-generatevalue"></a>

* 엔티티의 주키 ( 기본키 매핑 )
* 생성 전략과 생성기를 설정 할 수 있음, 기본 전력은 AUTO

### **4-1. GenerationType.AUTO** <a href="#id-4-1-generationtypeauto" id="id-4-1-generationtypeauto"></a>

* 직접 할당
  * @Id만 사용

### **4-2. GenerationType.IDENTITY** <a href="#id-4-2-generationtypeidentity" id="id-4-2-generationtypeidentity"></a>

* 키 생성을 데이터베이스에 위임
* @GeneratedValue(strategy = GenerationType.IDENTITY)
* MySQL, PostgreSQL, SQL Server DB2
* 커밋 시점에 Insert SQL 실행 한 이후에 얻을 수 있음
* 예외적으로 .persist()가 호출되는 시점에 db insert 발생
* 모아서 insert 하지 못함

```java
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id; 
```

#### **4-3. GenerationType.SEQUENCE** <a href="#id-4-3-generationtypesequence" id="id-4-3-generationtypesequence"></a>

* 데이터베이스 Sequence Object를 사용
* @GeneratedValue(strategy = GenerationType.SEQUENCE)
* Oracle, PostgreSQL, DB2, H2
* 테이블 마다 시퀀스 오브젝트를 따로 관리
  * @SequenceGenerator에 sequenceName 속성을 추가

```java
@Entity
@SequenceGenerator( name = "MEMBER_SEQ_GENERATOR", // 식별자 생성기 이름
                    sequenceName = "MEMBER_SEQ", // 매핑할 데이터베이스 시퀀스 이름
                    initialValue = 1, // DDL 생성 시에만 사용됨
                    allocationSize = 1) //시퀀스 한 번 호출에 증가하는 수 (성능 최적화에 사용)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
    generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
}
```

### **4-4. GenerationType.TABLE** <a href="#id-4-4-generationtypetable" id="id-4-4-generationtypetable"></a>

* 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스
* @GeneratedValue(strategy = GenerationType.TABLE)
* 테이블 마다 시퀀스 오브젝트를 따로 관리@TableGenerator 필요
* 모든 데이터 베이스 적용 가능 하나 성능상의 이슈 있음 -> 운영에 적합하지 않다

```java
@Entity
@TableGenerator ( name = "MEMBER_SEQ_GENERATOR",
                  table = "MY_SEQUENCES", // 데이터베이스 이름
                  pkColumnValue = "MEMBER_SEQ",
allocationSize = 1)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
    generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
}
```

### **4-5. GenerationType.UUID** <a href="#id-4-5-generationtypeuuid" id="id-4-5-generationtypeuuid"></a>

Indicates that the persistence provider must assign primary keys for the entity by generating an RFC 4122 Universally Unique IDentifier.

* UUID 생성

```java
@GeneratedValue(strategy = GenerationType.UUID)
```

## **5. @Column** <a href="#id-5-column" id="id-5-column"></a>

맴버 변수와 DB Table의 컬럼에 매핑

* 속성
  * name : 객체명과 DB 컬럼명을 다르게 하고 싶은 경우, DB 컬럼명으로 설정할 이름을 name 속성
  * insertable : insert 여부 ( true/false )
  * updatable : 컬럼을 수정했을 때 DB에 추가를 할 것인지 여부 ( true/false ), @Column(updatable = false)
  * nullable : @Column(nullable = false): NOT NULL 제약조건
  * unique : unique index (잘 사용 하지 않음 ) -> @Table 사용
  * columnDefinition : @Column(columnDefinition = “varchar(100) default ‘EMPTY’”): 직접 컬럼 정보를 작성
  * length : 문자 길이 제약 조건, String Type에서만 사용
  * precision : 숫자가 엄청 큰 BigDecimal 의 경우 (Ex.private BigDecimal age;)

## **6. @Temporal** <a href="#id-6-temporal" id="id-6-temporal"></a>

* 날짜 Type (java.util.Date, java.util.Calendar) 매핑
* 날짜 타입 사용 시 해당 annotation을 사용
* TemporalType enum class에는 세 가지가 존재.
  * DATE: 날짜, DB의 date 타입과 매핑, Ex. 2019-08-13
  * TIME: 시간, DB의 time 타입과 매핑, Ex. 14:20:38
  * TIMESTAMP: 날짜와 시간, DB의 timestamp 타입과 매핑, Ex. 2019-08-13 14:20:
* java 8의 LocalDate(date), LocalDateTime(timestamp) 을 사용할 때는 생략이 가능

## **7. @Transient** <a href="#id-7-transient" id="id-7-transient"></a>

* 특정 필드를 컬럼에 매핑하지 않음.
* DB에 관계없이 메모리에서만 사용하고자 하는 객체에 해당 annotation을 사용
* 해당 annotation이 붙은 필드는 DB에 저장, 조회가 되지 않는다.

## **8. @Enumerated** <a href="#id-8-enumerated" id="id-8-enumerated"></a>

* Enum Type 매핑, Enum 객체 사용 시 해당 annotation을 사용
* 속성
  * value :
    * EnumType.ORDINAL : enum 순서를 데이터베이스에 저장 (기본값), 사용하지 않는다. ( 추가 , 수정 시 문제 발생 할 수 있음 )
    * EnumType.String : enum 이름을 데이터베이스에 저장

## **9. @Lob** <a href="#id-9-lob" id="id-9-lob"></a>

* DB에서 varchar를 넘어서는 큰 내용을 넣고 싶은 경우에 해당 annotation을 사용 - @Lob에는 지정할 수 있는 속성이 없다.
* 매핑하는 필드의 타입에 따라 DB의 BLOB, CLOB과 매핑
* 필드의 타입이 문자열: CLOB -> String, char\[]
* java.sql.CLOB그 외 나머지: BLOB -> byte\[], java.sql.BLOB
