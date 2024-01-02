# Query-DSL DTO로 반환

QueryDSL에서 조회 결과를 원하는 행 기반 변환을 위한 FactoryExpressions와 집계를 위한 ResultTransformer를 제공합니다. 즉 전체 컬럼이 아닌 원하는 컬럼만 결과로 Entity 클래스가 아닌 DTO 클래스로 정의하는 방법입니다.

com.querydsl.core.types.FactoryExpression는 Bean 생성, 생성자 호출 및 보다 복잡한 객체 생성에 사용하며 com.querydsl.core.types.Projections클래스를 사용하며, com.querydsl.core.ResultTransformer인터페이스 의 경우 GroupBy에 사용합니다.

1. Tuple 사용
2. Projections 사용 - setter 사용
3. 생성자 - @QueryProjection 사용
4. 결과집계

**Projections을 사용하는 경우 다음 규칙을 준수합니다.**

* 프로퍼티 접근 (Setter) : setter와 기본 생성자 필수
* 필드 직접 접근 : 기본 생성자 필수
* 생성자 사용 : 타입이 일치하는 생성자 필수

**필드 접근시 필드애 명칭을 붙혀 사용할 수 있습니다.** ExpressionUtils.as(source, alias) : 필드나 서브 쿼리에 별칭을 적용할 사용합니다. username.as(alias) : 필드에 별칭을 적용할 때 사용합니다.

## **1. Tuple 사용** <a href="#id-1-tuple" id="id-1-tuple"></a>

[QueryDSL\_기본](https://hyomee.github.io/doc/spring/01\_Jpa/002\_QueryDSL\_%EA%B8%B0%EB%B3%B8/) 3-2. select를 참고하면 됩니다.

```java
QTourlistEntity qTourlistEntity = QTourlistEntity.tourlistEntity
QPlannerTourlistEntity qPlannerTourlistEntity = QPlannerTourlistEntity.plannerTourlistEntity;

// 1. 쿼리 생성 및 실행 : Tuple 로 반환 : 선택된 컬럼만 
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

// 2. 다른 표현식으로 다음과 같이 작성할 수 있음
List<Tuple> tourTuples = jpaQueryFactory
        .select(new QTourlistEntity(
                        contentid,
                        title,
                        zipcode,
                        addr,
                        overview))
        .from(qTourlistEntity)
        .leftJoin(qPlannerTourlistEntity)
        .on(qTourlistEntity.contentid.eq(qPlannerTourlistEntity.contentid))
        .where(qPlannerTourlistEntity.plannerNo.eq(plannerNo))
        .fetch();

```

## **2. Projections 사용** <a href="#id-2-projections" id="id-2-projections"></a>

SELECT 절에 들어가는 컬럼을 Bean을 채워야 하는 경우 Bean 프로젝션을 사용합니다.

```java
// SELECT 구문을 makeProjectiosBean 메서드 호춯 
jpaQueryFactory.select(makeProjectiosBean())
                .from(qTourlistEntity)
                .where(condition(tourCondition))
                .fetch();
```

> Projections.bean을 사용해서 DTO 객체에 직접 설정

```java
// Projections.bean을 사용해서 DTO 객체에 직접 설정 
// 컬럼명이 틀린 경우 .as("컬럼명")으로 처리할 수 있음 
// setter를 이용한 매핑 
private FactoryExpressionBase<TourlistDTO> makeProjectiosBean() {
    return Projections.bean(TourlistDTO.class,
            qTourlistEntity.contentid ,
            qTourlistEntity.contenttypeid ,                
            qTourlistEntity.title ,
            qTourlistEntity.zipcode ,
            qTourlistEntity.addr ,
            qTourlistEntity.tel ,
            qTourlistEntity.overview ,
            qTourlistEntity.recommendCount ,
            qTourlistEntity.addCount );
}  
```

> setter 대신 필드를 직접 사용해야 하는 경우 fields를 사용.

```java
private FactoryExpressionBase<TourlistDTO> makeProjectiosBean() {
    return Projections.fields(TourlistDTO.class,
            qTourlistEntity.contentid ,
            qTourlistEntity.contenttypeid ,                
            qTourlistEntity.title ,
            qTourlistEntity.zipcode ,
            qTourlistEntity.addr ,
            qTourlistEntity.tel ,
            qTourlistEntity.overview ,
            qTourlistEntity.recommendCount ,
            qTourlistEntity.addCount );
}
```

## **3. 생성자 - @QueryProjection 사용** <a href="#id-3-queryprojection" id="id-3-queryprojection"></a>

Projections.constructor을 사용해서 생성자 기반으로 행변환울 할 수 있습니다.

```java
private ConstructorExpression<TourlistDTO> makeProjectiosConstructor() {
    return Projections.constructor(TourlistDTO.class,
            qTourlistEntity.contentid ,
            qTourlistEntity.contenttypeid ,                
            qTourlistEntity.title ,
            qTourlistEntity.zipcode ,
            qTourlistEntity.addr ,
            qTourlistEntity.tel ,
            qTourlistEntity.overview ,
            qTourlistEntity.recommendCount ,
            qTourlistEntity.addCounty);
}
```

다른 방법으로 @QueryProjection 어노테이션을 사용합니다. @QueryProjection은 생성자를 통해 DTO를 조회하는 방법 빌드시 Q파일이 생성 됩니다.

```java
class UserInfo {

    private String firstName, lastName;

    @QueryProjection
    public UserInfo(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    // getters and setters
}  

QUser user = QUser.user;   
List <UserInfo> result = jpaQueryFactory
        .select(new QUserInfo(user.firstName, user.lastName)).
        .from(user)
        .where(user.valid.eq(true))     
        .fetch();

```

## **4. 결과집계** <a href="#id-4" id="id-4"></a>

클래스 com.querydsl.core.group.GroupBy 는 쿼리 결과를 메모리에 집계하는 데 사용할 수 있는 집계 기능을 제공합니다. 다음은 몇 가지 사용 예입니다.

> 상위 하위 관계 집계로 관련 댓글에 대한 게시물 ID 맵이 반환됩니다.

```java
import  static com.querydsl.core.group.GroupBy.*;

Map<Integer, List<Comment>> 결과 = query.from(게시물, 댓글)
    .where(comment.post.id.eq(post.id))
    .transform(groupBy(post.id).as(list(comment)));
```

> 여러 결과 열 : 게시물 이름과 댓글 ID에 액세스할 수 있는 그룹 인스턴스에 게시물 ID 맵이 반환됩니다.

```java
Map<Integer, Group> 결과 = query.from(게시물, 댓글)
    .where(comment.post.id.eq(post.id))
    .transform(groupBy(post.id).as(post.name, set(comment.id)));
```

Group은 Tuple 인터페이스와 동등한 GroupBy입니다.
