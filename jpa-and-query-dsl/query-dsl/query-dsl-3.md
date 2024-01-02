# Query-DSL 수정 및 삭제

QueryDSL을 사용해서 수정 및 삭제를 할 수 있으며 영속성을 지원하지 않습니다.(캐시 지원하지 않음)

## 1.삭제 <a href="#id-1" id="id-1"></a>

Querydsl JPA의 삭제 후 삭제된 갯수 반환

```java
QCustomer 고객 = QCustomer.customer;
// 1. 모든 고객 삭제
queryFactory.delete(고객).execute();

// 2. 레벨 3 미만의 모든 고객 삭제 
queryFactory.delete(customer)
    .where(customer.level.lt( 3 ))
    .execute();
```

## 2.수정 <a href="#id-2" id="id-2"></a>

Querydsl JPA의 수정 후 수정한 갯수 반환으로 대량 데이터 삭제시 성능을 높일 수 있습니다. 단, 업데이트 대상들에 대한 Cache Eviction을 필요로 합니다.

> Eviction, Expiration, Passivatio cache는 속도를 위해 대부분 memory를 사용

* Eviction
  * 공간이 필요할 때 어떤 데이터를 지워주는 것
  * memory가 가득 차면 사용하지 않는 데이터를 지워줘야 새로 데이터가 들어올 수 있음
  * 대부분 LRU(Least Recently Used : 가장 오랜 기간 참조되지 않은 데이터를 교체) 알고리즘 방식을 사용
* Expiration
  * 데이터의 유통기한
  * 일반적으로 TTL(Time To Live)이라는 단어를 사용
* Passivation
  * 기능을 사용하면 eviction의 대상이 되는 데이터가 지워지기 전에 우선 디스크등 다른 스토리지에 저장
  * 추후 같은 데이터에 대한 요청이 들어오면 파일에서 찾아 돌려줌

```java
QCustomer 고객 = QCustomer.customer;

// Bob이라는 고객의 이름을 Bobby로 바꿉니다. 
queryFactory.update(customer).where(customer.name.eq( "Bob" ))
    .set(ustomer.name, "바비" )
    .execute();
```

## 3. Cache <a href="#id-3-cache" id="id-3-cache"></a>

JPA는 캐시를 관리하기 위한 javax.persistence.Cache 인터페이스를 제공 (EntityManageFactory에서 객체 반환) - 자바 ORM 표준 JPA 도서 학습 필요

> Cache 관리 객체 조회 예시

```java
Cache cache = emf.get cache.contains(TestEntity.class, testEntity.getId());
System.out.println("contains = ", + contains);

// Cache 인터페이스
public interface Cache {
    //해당 엔티티가 캐시에 있는지 여부 확인
    public boolean contains(Class cls, Object primaryKey);

    //해당 엔티티중 특정 식별자를 가진 엔티티를 캐시에서 제거
    public void evict(Class cls, Object primaryKey);

    //해당 엔티티 전체를 캐시에서 제거
    public void evict(Class cls);

    //모든 캐시 데이터 제거
    public void evictAll();

    //JPA Cache 구현체 조회
    public <T> T unwrap(Class<T> cls);
}
```

참고 : \[자바 ORM 표준 JPA 프로그래밍 16.2 2차 캐시] [https://milenote.tistory.com/157](https://milenote.tistory.com/157)
