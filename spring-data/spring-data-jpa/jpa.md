# JPA 이름 규칙

JPA로 개발시 쿼리 자동 생성에 따른 Repository Method 명명 규칙 입니다.

## **1. repositories 요소**

| 이름                           | 설명                                                                                                                                                                                                                        |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| base-package                 | Repository 자동 검색할 패키지로 설정한패키지 아래의 모든 패키지도 검사되며,  와일드카드가 허용됩니다.                                                                                                                                                            |
| repository-impl-postfix      | <p>사용자 정의 Repository 구현체의 접미사. <br>기본값은 Impl.</p>                                                                                                                                                                         |
| query-lookup-strategy        | <p>쿼리를 생성하는 데 사용할 전략으로  " <a href="https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-lookup-strategies">쿼리 조회 전략</a> "에서 확인 할 수 있습니다.<br>기본값은 create-if-not-found.</p> |
| named-queries-location       | 외부에서 정의된 쿼리가 포함된 속성 파일을 검색할 위치 .                                                                                                                                                                                          |
| consider-nested-repositories | 중첩된 저장소 인터페이스 정의를 고려해야 하는지 여부. 기본값은 false                                                                                                                                                                                 |

## **2. 메서드 명명 Keyword**

| 명명 규칙                                                                              | 설명                                                                                                                                                                     |
| ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p>find…By, <br>read…By, <br>get…By, <br>query…By, <br>search…By,<br>stream…By</p> | <p>조회시 사용 되는 메서드 명명 규칙으로 , Collection, Streamable하위 유형,  Page,  GeoResults 타입으로 결과를 반환 하는 메서드 입니다.<br>findBy…로 사용 하거나 findMyDomainTypeBy…추가 키워드와 조합하여 사용할 수 있습니다 .</p> |
| exists…By                                                                          | 존재 유무를 조회 하는 쿼리에 대한 메서드 명명 규칙이며  boolean결과를 반환합니다.                                                                                                                     |
| count…By                                                                           | 쿼리에서 count와 같은 역활을 하늠 메서드 명명 규칙                                                                                                                                        |
| <p>delete…By,<br>remove…By</p>                                                     |  void 또는 삭제 횟수를 반환하는 삭제 쿼리 메서드 명명 규칙                                                                                                                                   |
| <p>…First&#x3C;number>…,<br>…Top&#x3C;number>…</p>                                 | Top 쿼리와 같은 의미를 가지는 메서드 명명 규칙으로 number에 지정한 수 만큼 제한 할 때 사용 됩니다.                                                                                                         |
| …Distinct…                                                                         | 쿼리의 Discinct와 같은 기능을 하는 메서드의 명명 규칙                                                                                                                                     |

## **3, 쿼리에 사용 되는 조건식 명명 규칙**

| 키워드                   | 표현식                                     |
| --------------------- | --------------------------------------- |
| AND                   | And                                     |
| OR                    | Or                                      |
| AFTER                 | After,IsAfter                           |
| BEFORE                | Before,IsBefore                         |
| CONTAINING            | Containing, IsContaining,Contains       |
| BETWEEN               | Between,IsBetween                       |
| ENDING\_WITH          | EndingWith, IsEndingWith,EndsWith       |
| EXISTS                | Exists                                  |
| FALSE                 | False,IsFalse                           |
| GREATER\_THAN         | GreaterThan,IsGreaterThan               |
| GREATER\_THAN\_EQUALS | GreaterThanEqual,IsGreaterThanEqual     |
| IN                    | In,IsIn                                 |
| IS                    | Is, Equals, (또는 키워드 없음)                 |
| IS\_EMPTY             | IsEmpty,Empty                           |
| IS\_NOT\_EMPTY        | IsNotEmpty,NotEmpty                     |
| IS\_NOT\_NULL         | NotNull,IsNotNull                       |
| IS\_NULL              | Null,IsNull                             |
| LESS\_THAN            | LessThan,IsLessThan                     |
| LESS\_THAN\_EQUAL     | LessThanEqual,IsLessThanEqual           |
| LIKE                  | Like,IsLike                             |
| NEAR                  | Near,IsNear                             |
| NOT                   | Not,IsNot                               |
| NOT\_IN               | NotIn,IsNotIn                           |
| NOT\_LIKE             | NotLike,IsNotLike                       |
| REGEX                 | Regex, MatchesRegex,Matches             |
| STARTING\_WITH        | StartingWith, IsStartingWith,StartsWith |
| TRUE                  | True,IsTrue                             |
| WITHIN                | Within,IsWithin                         |

### 3-1. 예제

| 키워드                 | Sample                                                               | JPQL snippet                                                  |
| ------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------- |
| And                 | findByLastnameAndFirstname                                           | … where x.lastname = ?1 and x.firstname = ?2                  |
| Or                  | findByLastnameOrFirstname                                            | … where x.lastname = ?1 or x.firstname = ?2                   |
| <p>Is<br>Equals</p> | <p>findByFirstname<br>findByFirstnameIs<br>findByFirstnameEquals</p> | … where x.firstname = ?1                                      |
| Between             | findByStartDateBetween                                               | … where x.startDate between ?1 and ?2                         |
| LessThan            | findByAgeLessThan                                                    | … where x.age < ?1                                            |
| LessThanEqual       | findByAgeLessThanEqual                                               | … where x.age <= ?1                                           |
| GreaterThan         | findByAgeGreaterThan                                                 | … where x.age > ?1                                            |
| GreaterThanEqual    | findByAgeGreaterThanEqual                                            | … where x.age >= ?1                                           |
| After               | findByStartDateAfter                                                 | … where x.startDate > ?1                                      |
| Before              | findByStartDateBefore                                                | … where x.startDate < ?1                                      |
| IsNull              | findByAgeIsNull                                                      | … where x.age is null                                         |
| IsNotNull,NotNull   | findByAge(Is)NotNull                                                 | … where x.age not null                                        |
| Like                | findByFirstnameLike                                                  | … where x.firstname like ?1                                   |
| NotLike             | findByFirstnameNotLike                                               | … where x.firstname not like ?1                               |
| StartingWith        | findByFirstnameStartingWith                                          | … where x.firstname like ?1(parameter bound with appended %)  |
| EndingWith          | findByFirstnameEndingWith                                            | … where x.firstname like ?1(parameter bound with prepended %) |
| Containing          | findByFirstnameContaining                                            | … where x.firstname like ?1(parameter bound wrapped in %)     |
| OrderBy             | findByAgeOrderByLastnameDesc                                         | … where x.age = ?1 order by x.lastname desc                   |
| Not                 | findByLastnameNot                                                    | … where x.lastname <> ?1                                      |
| In                  | findByAgeIn(Collection\<Age> ages)                                   | … where x.age in ?1                                           |
| NotIn               | findByAgeNotIn(Collection\<Age> ages)                                | … where x.age not in ?1                                       |
| True                | findByActiveTrue()                                                   | … where x.active = true                                       |
| False               | findByActiveFalse()                                                  | … where x.active = false                                      |
| IgnoreCase          | findByFirstnameIgnoreCase                                            | … where UPPER(x.firstame) = UPPER(?1)                         |

## **4. 조건식의 수정 키워드**

| 키워드                           | 설명                                          |
| ----------------------------- | ------------------------------------------- |
| IgnoreCase,IgnoringCase       | 대소문자 구별 없이 조회                               |
| AllIgnoreCase,AllIgnoringCase | 대소문자를 무시                                    |
| OrderBy…                      | 정렬 순서 (예: OrderByFirstnameAscLastnameDesc). |

## **5. 예제**

```java
public interface CrudRepository<T, ID> extends Repository<T, ID> {

  <S extends T> S save(S entity);    	// 저장  

  Optional<T> findById(ID primaryKey);  // 조회

  Iterable<T> findAll();  				// 전체 조회             

  long count();							// count                        

  void delete(T entity);     			// 삭제          

  boolean existsById(ID primaryKey);    // 존재 여부

  // … more functionality omitted.
}
```

## 6. 페이징&#x20;

```java
Page<User> findByLastname(String lastname, Pageable pageable);

Slice<User> findByLastname(String lastname, Pageable pageable);

Window<User> findTop10ByLastname(String lastname, ScrollPosition position, Sort sort);

List<User> findByLastname(String lastname, Sort sort);

List<User> findByLastname(String lastname, Pageable pageable);
```

&#x20;

참고 : [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-keywords](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-keywords)
