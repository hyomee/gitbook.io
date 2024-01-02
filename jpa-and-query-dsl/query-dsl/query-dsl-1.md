# Query-DSL 기본

QueryDSL은 JPQL이 제공하는 모든 기능을 제공하기 때문에 JPQL와 혼용해서 사용할 필요는 없습니다. 또한 Union, UnionAll과 같은 기능은 제공되지 않습니다. 즉 JPQL에서 제공하지 않는 것은 제공하지 않습니다.

## **1. 인스턴스 변수 선언, JPAQuery 생성** <a href="#id-1-jpaquery" id="id-1-jpaquery"></a>

### **1-1. 인스턴스 변수 선언** <a href="#id-1-1" id="id-1-1"></a>

Entity 명이 DemoEntity.java 파일이면 QDemoEntity.class파일로 생성이 되고 사용하기 위해서는 다음과 같이 인스턴스 변수를 만들어 사용할 수 있습니다.

```java
QDemoEntity  demoentity = QDemoEntity.demoentity;
or
QDemoEntity demoentity = new QDemoEntity( "demoentity" );

```

### **1-2. JPA API 사용** <a href="#id-1-2-jpa-api" id="id-1-2-jpa-api"></a>

JAP API를 사용하기 위해서는 JPAQuery 또는 HibernateQuery 인스턴스 변수를 사용해야 합니다.

```java
// JAP API 사용시
JPAQuery<?> query = new JPAQuery<Void>(entityManager);

//  Hibernate API 사용시
HibernateQuery<?> query = new HibernateQuery<Void>(session);
```

## **2. 쿼리 생성 및 결과** <a href="#id-2" id="id-2"></a>

### **2-1. 쿼리 생성** <a href="#id-2-1" id="id-2-1"></a>

| 문법                                      | 설명                                      |
| --------------------------------------- | --------------------------------------- |
| from                                    | 테이블 ( Q엔티티객체 ).                         |
| innerJoin, join, leftJoin, fullJoin, on | 조인,                                     |
|                                         | - 첫 번째 인자는 조인 소스                        |
|                                         | - 두 번재 인자는 대상(별칭)                       |
| where                                   | 조건절. 가변인자나 and/or 메서드를 이용해서 필터 사용       |
| groupBy                                 | 그룹                                      |
| having                                  | Predicate 표현식을 이용해서 “group by” 그룹핑의 필터  |
| orderBy                                 | 정렬 표현식을 이용해서 정렬 순서를 지정                  |
|                                         | - 숫자나 문자열에 대해서는 asc()나 desc()           |
|                                         | - OrderSpecifier에 접근하기 위해 다른 비교 표현식을 사용 |
| limit, offset, restrict                 | 결과의 페이징을 설정                             |
|                                         | - limit은 최대 결과 개수                       |
|                                         | - offset은 결과의 시작 행                      |
|                                         | restrict는 limit과 offset을 함께 정의          |

### **2-2. 검색 조건 (JPQL과 동일)** <a href="#id-2-2-jpql" id="id-2-2-jpql"></a>

| 문법                      | SQL 쿼리               |
| ----------------------- | -------------------- |
| eq(“something”)         | = ‘something’        |
| ne(“something”)         | != ‘something’       |
| eq(“something”).not()   | != ‘something’       |
| like(“%something”)      | like ‘%something’    |
| startsWith(“something”) | like ‘something%’    |
| contains(“something”)   | like ‘%something%’   |
| isNull()                | is null              |
| isNotNull()             | is not null          |
| isEmpty()               | 길이가 0                |
| isNotEmpty()            | 길이가 0이 아님            |
| in(“foo”, “bar”)        | in(“foo”, “bar”)     |
| notIn(“foo”, “bar”)     | not in(“foo”, “bar”) |
| in(“foo”, “bar”).not()  | not in(“foo”, “bar”) |
| between(20, 30)         | between 20, 30       |
| notBetween(20, 30)      | not between 20, 30   |
| between(20, 30).not()   | not between 20, 30   |
| gt(28)                  | > 28                 |
| goe(28)                 | >= 28                |
| lt(28)                  | < 28                 |
| loe(28)                 | <= 28                |

### **2-3. Query 반환** <a href="#id-2-3-query" id="id-2-3-query"></a>

| 문법             | 설명                                        |
| -------------- | ----------------------------------------- |
| fetch()        | 리스트로 결과를 반환                               |
|                | - 데이터가 없으면 빈 리스트를 반환                      |
| fetchOne()     | 단건 조회                                     |
|                | - 결과가 없을때는 null 을 반환                      |
|                | - 결과가 둘 이상일 경우에는 NonUniqueResultException |
| fetchFirst()   | 처음의 한건 조회                                 |
| fetchResults() | 페이징을 위해 사용                                |
|                | - 페이징을 위해서 total contents                 |
| fetchCount()   | count 쿼리                                  |

> 예제에 사용할 객체

```java
// 데모 객체
public class DemoEventDTO extends AuditDTO {
  private int id;
  private String title;
  private String content;
  private String name;
}

// 데모 Entity
public class DemoEventEntity extends AuditVO {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private int id;
    private String title;
    private String content;
    private String name;
}

// 선택적 조회 컬럼 객체
public class DemoEventSelectDTO   {
  private String title;
  private String content;
}
```

> 아래 작성된 SQL Query를 QueryDSL로 코드로 작성합니다.

```java
SELECT * 
  FROM TB_EVENT_DEMO
 WHERE ID = 1
```

## **3. 쿼리 예제** <a href="#id-3" id="id-3"></a>

SQL Query에 대응되는 QueryDSL 코드로 작성하기 위해서는 다음 순서로 코드를 작성하면 됩니다.

1. QDemoEventEntity 객체를 인스턴스 변수 선언한다.
2. JAP API 사용하는 코드를 작성 한다.
   * selectForom : 테이블 전체 조회
   * select : 원하는 컬럼만 조회

### **3-1. selectForom** <a href="#id-3-1-selectforom" id="id-3-1-selectforom"></a>

SQL Query에서 테이블의 전체 컬럼을 조회 하는 것과 동일한 기능읗 합니다.

```java
QDemoEventEntity qDemoEventEntity = QDemoEventEntity.demoEventEntity;

// JAP API 사용 : JPAQuery을 선언 과 실행 분리 
JPAQuery<DemoEventEntity> countQuery = jpaQueryFactory.selectFrom(qDemoEventEntity)
                .where(qDemoEventEntity.id.eq(id));        
DemoEventEntity demoEntity = countQuery.fetchOne();

// OR 

// JAP API 사용 : JPAQuery을 선언 과 실행 통합 
DemoEventEntity demoEntity = jpaQueryFactory.selectFrom(qDemoEventEntity)
                .where(qDemoEventEntity.id.eq(id))
                .fetchOne()
```

### **3-2. select** <a href="#id-3-2-select" id="id-3-2-select"></a>

조회 쿼리에서 원하는 컬럼만 가지고 오는 경우 사용 하는 방법으로 예제에서는 타입에 안전한 Map인 Tuple로 쿼리 결과를 받아 원하는 객체에 변환하는 방법입니다.

```java
 JPAQuery<Tuple> selectedColumnQuery = jpaQueryFactory.select(qDemoEventEntity.title
                        , qDemoEventEntity.content)
                .from(qDemoEventEntity)
                .where(qDemoEventEntity.id.eq(Id))
                .limit(1)

Tuple selectEdTuple =  selectedColumnQuery.fetchOne();

return DemoEventSelectDTO.builder()
            .title(selectEdTuple.get(qDemoEventEntity.title ))
            .content(selectEdTuple.get(qDemoEventEntity.content))
            .build();
}
```

* 수행 순서
  1. JPAQuery 문장 생성, limit 1을 지정 하여 한 건 조회
  2. fetchOne을 사용해서 조회하여 Tuple에 저장
  3. 결과를 DemoEventSelectDTO 객체에 값을 저장 후 반환

다른 방법으로는 Projections등 다양한 방법이 있습니다.

조회 조건절에는 and, or등을 사용해서 복합 조건식을 추가할 수 있습니다.

```java
// SQL Query : NAME = '홍길동' AND TITLE = '타이틀'
where( (qDemoEventEntity.name.eq(name)).and(qDemoEventEntity.title.eq(title)) )

// SQL Query : TITLE = '%타이틀%' OR NAME = '홍길동'
where( (qDemoEventEntity.title.contains(title)).or(qDemoEventEntity.name.eq(name)) )
```

## **4. 조인 쿼리** <a href="#id-4" id="id-4"></a>

관광계획(TB\_PLANNER\_TOURLIST)에 있는 관광지(TB\_TOURLIST)정보를 조회하는 쿼리는 다음과 같습니다.

```java
SELECT A.* 
    FROM TB_TOURLIST A, 
         TB_PLANNER_TOURLIST B
 WHERE A.CONTENTID = B.CONTENTID
      AND B.PLANNER_NO = 1001;
```

QueryDSL의 조인을 사용하면 다음과 같은 코드를 작성할 수 있습니다.

1. 관광계획(TB\_PLANNER\_TOURLIST), 관광지(TB\_TOURLIST)에 대한 Q파일 인스턴스 변수를 생성합니다.
2. selectFrom 쿼리 생성 코드를 작성 후 fetch을 사용하는 코드를 작성합니다.

```java
QTourlistEntity qTourlistEntity = QTourlistEntity.tourlistEntity
QPlannerTourlistEntity qPlannerTourlistEntity = QPlannerTourlistEntity.plannerTourlistEntity;

List<TourlistEntity> tourlistEntities = jpaQueryFactory.selectFrom(qTourlistEntity)
                .from(qTourlistEntity)
                .leftJoin(qPlannerTourlistEntity)
                .on(qTourlistEntity.contentid.eq(qPlannerTourlistEntity.contentid))
                .where(qPlannerTourlistEntity.plannerNo.eq(1001))
                .fetch();
```

조인 쿼리 사용시 원하는 필드만 조회하는 방법은 기본 쿼리에 있는 Tuple을 이용하는 방법을 우선 소개 합니다,.

1. Join 쿼리를 사용하는 기본 방법 동일하고 List\<Tuple>로 반환 값을 받습니다.
2. 조회되 List를 Stream을 사용해서 반환 객체로 변환 합니다.

```java
// 1. 조인 할 대상 Entity 
QTourlistEntity qTourlistEntity = QTourlistEntity.tourlistEntity
QPlannerTourlistEntity qPlannerTourlistEntity = QPlannerTourlistEntity.plannerTourlistEntity;

// 2. 쿼리 생성 및 실행 : Tuple 로 반환 : 선택된 컬럼만 
List<Tuple> tourTuples = jpaQueryFactory.select(qTourlistEntity.contentid ,
                qTourlistEntity.title ,
                qTourlistEntity.zipcode ,
                qTourlistEntity.addr,
                qTourlistEntity.overview )
        .from(qTourlistEntity)
        .leftJoin(qPlannerTourlistEntity)
        .on(qTourlistEntity.contentid.eq(qPlannerTourlistEntity.contentid))
        .where(qPlannerTourlistEntity.plannerNo.eq(plannerNo))
        .fetch();

// 3. 반환 객체로 변환 : Stream 사용 
List<TourlistSelectedDTO> tourlists = tourTuples.stream().map(tourTuple-> {
    return TourlistSelectedDTO.builder()
            .contentid(tourTuple.get(qTourlistEntity.contentid))
            .title(tourTuple.get(qTourlistEntity.title))
            .zipcode(tourTuple.get(qTourlistEntity.zipcode))
            .addr(tourTuple.get(qTourlistEntity.contentid))
            .overview(tourTuple.get(qTourlistEntity.contentid))
            .build();
}).collect(Collectors.toList());
```

그외 groupBy, orderBy 등도 쿼리 작성 하듯이 작성하면 됩니다.

```
.orderBy(qTourlistEntity.zipcode.asc(), qTourlistEntity.firstName.desc())
.groupBy(qTourlistEntity.zipcode, qTourlistEntity.areacode)
```

## **5. SubQuery** <a href="#id-5-subquery" id="id-5-subquery"></a>

Querydsl에서 서브 쿼리는 JPAExpressions 클래스를 통해 생성할 수 있습니다.

* select 절의 서브 쿼리에서는 ExpressionUtils.as() 메서드로 Alias를 지정할 수 있습니다.
* where 절의 서브 쿼리에서는 JPAExpressions.select() 메서드로 조건을 지정할 수 있습니다.
* 단, from 절에서의 서브 쿼리는 지원하지 않습니다

### **5-1. Selet절 SubQuery** <a href="#id-5-1-selet-subquery" id="id-5-1-selet-subquery"></a>

동일 Entity에 접근시 Alias명을 지정하여 객체를 생성하고 Alias명을 통해 객체에 접근하는 코드를 작성해야 합니다. ( Alias : tourList )

```java
QTourlistEntity qTourlistEntity = QTourlistEntity.tourlistEntity

// 동일 Entity 접근을 하여 alias 지정 
QTourlistEntity qtourList = new QTourlistEntity("tourList");

// 관광지 이름과 최대추천수 만 조회 
// TourlistDTO - title, maxAge 컬럼 있음
List<TourlistDTO> jpaQueryFactory.select(Projections.fields(TourlistDTO.class,)
        usqTourlistEntityer.title,
        ExpressionUtils.as(JPAExpressions.select(qtourList.recommendCount.max())
            .from(recommendCount), "maxAge")))
        .fetch();
```

### **5-2. where절 SubQuery** <a href="#id-5-2-where-subquery" id="id-5-2-where-subquery"></a>

```java
QTourlistEntity qTourlistEntity = QTourlistEntity.tourlistEntity

// 동일 Entity 접근을 하여 alias 지정 
QTourlistEntity qtourList = new QTourlistEntity("tourList");

// 관광지 (Tourlist)의 추천수 평균 보다 큰 관광지 정보 조회 
jpaQueryFactory.selectFrom(qTourlistEntity)
        .where(qTourlistEntity.recommendCount.gt(
                JPAExpressions.select(tourList.recommendCount.avg())
                        .from(tourList)))
        .fetch();
```

SubQuery는 쿼리의 SUB-QUERY와 같은 기능을 하므로 필요한 경우이외는 사용하지 말아야 합니다.
