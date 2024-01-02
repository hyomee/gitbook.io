---
description: 영속성 컨텍스트
---

# Persistence Context

JPA는 Application과 Database사이에 객체(Entity)를 보관하는 공간을 만들어 엔티티 매니저(EntityManager)를 통해 객체(Entity)저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 객체(Entity)를 보관하고 관리하게 된다

<figure><img src="../../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

* 논리적인 개념으로 눈에 보이지 않으며, “Entity를 영구 저장 하는 환경” 이라는 뜻으로 persist() 시점에 Entity를 Persistence Context에 저장한다.
* 버퍼링, 캐싱등으로 동일성(Identity), 쓰지 지연(Transactional Write-Behind), 변경감지(Dirty Checking)등 이점이 있다

## **1. EntityManagerFactory** <a href="#id-1-entitymanagerfactory" id="id-1-entitymanagerfactory"></a>

* EntityManager 인스턴스 관리
* EntityManagerFactory 생성시 커넥션 풀 생성
* 같은 EntityManagerFactory를 통해 EntityManager는 같은 Database에 접속
* 한 번 생성 후 Application 전체에 공유

## **2. EntityManager** <a href="#id-2-entitymanager" id="id-2-entitymanager"></a>

* 엔티티를 저장, 수정, 삭제, 조회 등 엔티티와 관련된 작업 수행
  * em.find() : 조회
  * em.persist() : 저장
  * em.remove() : 삭제
  * em.flush() : 연속성 컨텍스트 내용을 db에 저장
  * em.detach() : 준연속성 객체로 전환
  * em.merge() : 연속 상태로 전환
  * em.clear() : 영속성 초기화
  * em.close() : 연속성 종료

## **3. Entity Life Cycle** <a href="#id-3-entity-life-cycle" id="id-3-entity-life-cycle"></a>

<figure><img src="../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

1. New (비영속객체 ) : Entity 객체가 DB에 반영 되지 않음, new 키워드를 사용해 생성한 Entity 객체
2. Managed (영속객체) : persist 메소드를 이용해 저장 한 경우와 db에서 조회한 경우로 Persistence Context가 관리하는 상태로 해당 객체를 수정 했는지(자동 변경 감지)알아 낸다.
3. Removed (삭제 객체) : Managed 상태인 객체를 remove method를 이용해 삭제 한 경우 removed상태로 작업 단위가 종료되는 시점에 실제로 DB삭제
4. Detached( 준 영속 객체 ) : 트랜잭션이 commit되었거나, clear, flush Method가 실행된 경우 Managed상태의 객체는 모두 Detached 상태가 됨, 이 상태에서는 DB동기화를 보장 못하므로 merge() Method로 Managed 상태로 전환 필요하다.

### **3-1. New (비영속)** <a href="#id-3-1-new" id="id-3-1-new"></a>

```java
MemberEntity memberEntity = new MemberEntity();
memberEntity.setId("member1");
memberEntity.setUserName("김길동");
```

### **3-2. Managed (영속객체)** <a href="#id-3-2-managed" id="id-3-2-managed"></a>

```java
MemberEntity memberEntity = new MemberEntity();
memberEntity.setId("member1");
memberEntity.setUserName("김길동");

EntityManager entityManager = entityManagerFactory,createEntityManager();
entityManager.getTransaction().begin();
// 객체를 저장한 상태 ( 영속 ) : db에 저장 하지 않음
entityManager.persist(member)

```

실제 저장을 위해서는 transaction.commit() 이 필요하다.

### **3-3. Detached (준영속)** <a href="#id-3-3-detached" id="id-3-3-detached"></a>

```java
MemberEntity memberEntity = new MemberEntity();
memberEntity.setId("member1");
memberEntity.setUserName("김길동");

EntityManager entityManager = entityManagerFactory,createEntityManager();
entityManager.getTransaction().begin();
entityManager.persist(member)

// 준영속 상테로 분리
entityManager.detach(member)

```

### **3-4. Removed (삭제)** <a href="#id-3-4-removed" id="id-3-4-removed"></a>

```java
 MemberEntity memberEntity = new MemberEntity();
```
