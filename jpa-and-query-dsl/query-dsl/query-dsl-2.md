# Query-DSL 동적 쿼리

동적 쿼리는 SQL Query에서 Where절의 조건을 프로그램을 작성 하여 조건에 맞는 조건식을 구성하는 것을 의미하는 것으로 아래 QueryDSL의 where 시그니처를 보면 Predicate을 인수로 받아 참/거짓을 판단 합니다.

1. Complex predicates (BooleanBuilder)
2. Dynamic expressions (Expressions)
3. BooleanExpression
4. BooleanBuilder, BooleanExpression 혼합
5. case문
6. Select 문에 문자 표시

```java
public Q where(Predicate o) {
    return queryMixin.where(o);
}
```

QueryDSL의 where절에 Predicate넘기는 방법은 다음과 같습니다.

1. Where 절에 피라미터로 Predicate 를 직접 작성하는 방법
   * 일반적인 방법으로 “where(qDemoEventEntity.id.eq(id))” 직접 코드 작성
2. BooleanBuilder 를 작성하는 방법
3. Where 절과 피라미터로 Predicate 를 상속한 BooleanExpression 을 사용하는 방법

여기는 BooleanBuilder, BooleanExpression 사용법에 대해서 설명을 합니다.

## **1. Complex predicates (BooleanBuilder)** <a href="#id-1-complex-predicates-booleanbuilder" id="id-1-complex-predicates-booleanbuilder"></a>

BooleanBuilder에 조건을 추가하고 where절에 builder를 전달하는 방법으로 StringBuffer과 비슷하게 조건식을 추가하는 형태로 처음에는 null로 시작하고 and, or , not 등으로 연결 하여 조건식을 계단식 형식으로 만들어 작업 결과를 호출합니다.

```java
package com.querydsl.core.BooleanBuilder;

public final class BooleanBuilder implements Predicate, Cloneable  {
    ...
}
```

> 관광지 제목이 부산으로 시작되고 추천수가 100 보다 큰 관광지 조회를 다음과 같이 작성할 수 있습니다.

```java
BooleanBuilder builder = new BooleanBuilder();
if (condition1) {
    builder.and(QTourlistEntity.title.startsWith("부산"));
}
if (condition2) {
    builder.and(QTourlistEntity.recommendCount.gt(100));
}
List<TourlistEntity> users = queryFactory.selectFrom(QTourlistEntity)
    .where(builder)
    .fetch();

```

## **2. Dynamic expressions (Expressions)** <a href="#id-2-dynamic-expressions-expressions" id="id-2-dynamic-expressions-expressions"></a>

com.querydsl.core.types.dsl.Expressions는 동적 표현식 구성을 위한 정적 팩토리 클래스로 동적 경로, 사용자 정의 구문을 작성하기 위해 사용합니다. 즉 Q 유형을 사용할 수 없는 경우 사용됩니다.

```java
// Expression Path 설정 ( Q 유형이 아닌 객체 ) : 변수 
Path<TourlistDTO> tourlistDTO = Expressions.path(TourlistDTO.class, "tourlistDTO");

// 조건식의 Key Path 설정 : 속성
Path<String> title = Expressions.path(String.class, tourlistDTO, "title");

// 값 설정 : 상수 
Constant<String> constant = (Constant<String>) Expressions.constant("부산");

// 조건식 생성 : Operations  
BooleanOperation titleBooleanOperation = Expressions.predicate(Ops.STARTS_WITH, personFirstName, constant);
```

> Ops는 조건 연산자로 QueryDSL에서 제공하는 연산자를 가지고 있는 enum 객체 입니다.

```java
package com.querydsl.core.types;

public enum Ops implements Operator {
    // general
    EQ(Boolean.class),
    NE(Boolean.class),
    IS_NULL(Boolean.class),
    IS_NOT_NULL(Boolean.class),
    ...
}
```

## **3. BooleanExpression** <a href="#id-3-booleanexpression" id="id-3-booleanexpression"></a>

QueryDsl BooleanExpression은 BooleanBuilder와 다르게 메서드를 작성해서 조건을 판단합니다.

```java
public abstract class BooleanExpression extends LiteralExpression<Boolean> implements Predicate {
}
```

> 관광지 제목이 부산으로 시작되고 추천수가 100 보다 큰 관광지 조회를 다음과 같이 작성할 수 있습니다.

```java
public List<TourlistEntity> findTourlistPage(TourlistConditionReqDTO tourCondition) {
    List<TourlistEntity> users = queryFactory.selectFrom(QTourlistEntity)
        .where(
            titleContains("부산"),
            recommendCountgt(100),
        )
        .fetch();
        ...
}

private BooleanExpression titleContains(String value) {
    return StringUtils.isNotEmpty(value) ?
            qTourlistEntity.title.startsWith(value) :
            null;
}

private BooleanExpression recommendCountgt(int value) {
    return StringUtils.isNotEmpty(value) ?
            qTourlistEntity.recommendCount.gt(value) :
            null;
}

```

## **4. BooleanBuilder, BooleanExpression 혼합** <a href="#id-4-booleanbuilder-booleanexpression" id="id-4-booleanbuilder-booleanexpression"></a>

BooleanBuilder, BooleanExpression 혼합하여 사용 하여 코드를 깔끔하게 작성할 수 있습니다.

```java
public List<TourlistEntity> findTourlistPage(TourlistConditionReqDTO tourCondition) {
    List<TourlistEntity> users = queryFactory.selectFrom(QTourlistEntity)
        .where(condition(tourCondition))
        .fetch();
        ...
}

private BooleanBuilder condition(TourlistConditionReqDTO tourCondition) {
        return new BooleanBuilder(ExpressionUtils.allOf(
                    titleContains(tourCondition.getTitle()),
                    recommendCountgt(tourCondition.getZipcode()), 
                ));

    }

private BooleanExpression titleContains(String value) {
    return StringUtils.isNotEmpty(value) ?
            qTourlistEntity.title.startsWith(value) :
            null;
}

private BooleanExpression recommendCountgt(int value) {
    return StringUtils.isNotEmpty(value) ?
            qTourlistEntity.recommendCount.gt(value) :
            null;
}


```

\*\*주의사항\*\*

where(null, condition(tourCondition))과 같이 작성이 되면 조건이 \[condition(tourCondition)]null 이면 무시 되어 전체가 조회됩니다.

```java
public List<TourlistEntity> findTourlistPage(TourlistConditionReqDTO tourCondition) {
    List<TourlistEntity> users = queryFactory.selectFrom(QTourlistEntity)
        .where(null, condition(tourCondition))
        .fetch();
        ...
}
```

## **5. case문** <a href="#id-5-case" id="id-5-case"></a>

CaseBuilder class를 사용해서 case-when-then-else 표현식을 작성합니다.

> 관광 정보의 추천수에 따라서 최고/상급/중급/일반으로 표시하는 예제 입니다.

```java
Expression<String> cases = new CaseBuilder()
        .when(qTourlistEntity.recommendCount.gt(10000)).then("최고")
        .when(qTourlistEntity.recommendCount.gt(5000)).then("상급")
        .when(qTourlistEntity.recommendCount.gt(2000)).then("중급")
        .otherwise("일반");
```

## **6. Select 문에 문자 표시** <a href="#id-6-select" id="id-6-select"></a>

SQL Query에서 SELECT ‘A’ as GUBUN.. 처럼 상수를 표현하기 위한 QueryDSL 구문입니다. 보통 SubQuery에 사용합니다.

```java
query.select(Expressions.constant(1), Expressions.constant("A"));
```

\
