# JPA 처리방법

JPA는 Application과 JDBC사이에서 JDBC API를 사용하여 SQL을 호출 하여 DB와 통신 하는 인터페이스로 개발자는 Entity 객체에 데이터를 저장하면 나머지는 JPA에서 처리한다

#### **1. JPA 쓰기**

<figure><img src="../../.gitbook/assets/image (103).png" alt=""><figcaption><p>JPA 쓰기</p></figcaption></figure>

1. 개발자 : 데이터 객체 (DAO)의 정보를 Entity 객체에 저장.
2. JPA : Entity 분석
3. JPA : Insert Query 작성
4. JPA : JDBC API를 사용하여 SQL을 DB에서 실행

```
Hibernate: call next value for hibernate_sequence
Hibernate: insert into member (name, id) values (?, ?
```

#### **2. JPA 읽기**

<figure><img src="../../.gitbook/assets/image (104).png" alt=""><figcaption><p>JPA 읽기</p></figcaption></figure>

1. 개발자 : 조회 조건 전달
2. JPA : Entity 매핑 정보를 바탕으로 적절한 SELECT SQL 생성
3. JPA : JDBC API를 사용하여 SQL을 DB에서 실행
4. JPA : DB로 부터 결과를 받음

```
[DEBUG] 2020-06-30 13:43:20.105 [restartedMain] JpaRunner - Member : Member(id=1, name=홍길동)
Hibernate: select member0_.id as id1_0_0_, member0_.name as name2_0_0_ from member member0_ where member0_.id=?
```

## **1. JPA 조회** <a href="#id-1-jpa" id="id-1-jpa"></a>

findById() 즉 테이블의 pk를 사용해서 조회 요청을 하면 JPA는1차 Cache의 Entity에 데이터가 있으면 DB조회 없이 데이터를 반환합니다.

<figure><img src="../../.gitbook/assets/image (105).png" alt=""><figcaption><p>JPA 조회</p></figcaption></figure>

1. findById()로 조회 요청 ( id = 1 )
2. 1차 Cache 검색 후 데이터가 없으면
3. DB 조회를 하여 1차 Cache에 저장
4. 조회 결과 반환
5. findById()를 사용해 동일한 id로 조회 요청 ( id = 1 )
6. 1차 Cache 검색 후 데이터 존재
7. DB 조회 없이 1차 Cache에 있는 데이터 반환

```
// 최초 조회 요청
JpaRunner - Member : Member(id=1)

// DB 조회
Hibernate: select member0_.id as id1_0_0_, member0_.name as name2_0_0_ from member member0_ where member0_.id=?

// 결과 반환 
JpaRunner - Member1 : Member(id=1, name=홍길동)

// 두번째 조회 요청
JpaRunner - Member1 : Member(id=1)

// 결과 반환 
JpaRunner - Member1 : Member(id=1, name=홍길동)

```

## **2. JPA 저장** <a href="#id-2-jpa" id="id-2-jpa"></a>

EntityManagement에 persist()로 저장 요청을 하면 Insert SQL을 생성 하여 쓰기 지연 SQL 저장소에 저장하고 1차 Cache에 정보를 보관 한 후 flush()를 통해 DB와 동기화 한다 . 최종 commit() 요청을 통해 DB에 저장된다.

<figure><img src="../../.gitbook/assets/image (106).png" alt=""><figcaption><p>JPA 저장</p></figcaption></figure>

1. 개발자 : 사용자 A를 영속성 컨텍스트에 저장 요청을 한다. (persist)
2. JPA : 사용자 A를 Insert Query 생성 후 쓰기 지연 SQL 저장소에 저장하고 영속성 컨텍스트에 등록한다.
3. 개발자 : 사용자 B를 영속성 컨텍스트에 저장 요청을 한다. (persist)
4. JPA : 사용자 B를 Insert Query 생성 후 쓰기 지연 SQL 저장소에 저장하고 영속성 컨텍스트에 등록한다.
5. 개발자 : DB 동기화 요청을 한다. (flush)
6. JPA : 사용자 A, 사용자 B를 DB 동기화. ( DB에 반영 되지는 않음, 영속성 컨텍스트는 지워지지 않음)
7. 개발자 : DB 저장 요청 (commit)
8. JPA : DB에 정보를 최종 저장. ( 최종 DB에 반영 됨 )

<figure><img src="../../.gitbook/assets/image (107).png" alt=""><figcaption><p>로그 확인</p></figcaption></figure>

entityManager.persist(memberA), entityManager.persist(memberB) 시 쿼리 생성이 되고 entityManager.flush() 시 동기화 되는 것을 확인할 수 있다.

<mark style="color:purple;">**flush 란**</mark>

영속성 컨텍스트의 변경 내용을 DB 에 반영하는 것을 의미 하며 개발자가 코드에서 적용하지 않으면 일반적으로 Transaction commit 이 일어날 때 flush가 발생된다(쓰기 지연 저장소에 쌓아 놨던 INSERT, UPDATE, DELETE SQL들이 DB에 날라간다) 즉 영속성 컨텍스트의 변경 사항들과 DB의 상태를 맞추는 작업을 하는 것을 의미한다. 실제 DB 반영은 commit에 의해서 적용된다. 동작과정은 다음과 같다

1. Dirty Checking : 영속성 컨텍스트 변경 감지
2. Entity룰 읽어서 쿼리 작성 후 쓰기 지연 SQL 저장소 저장
3. DB와 상태 동기화 : DB에 반영 되지는 않음, 영속성 컨텍스트는 지워지지 않음

## **3. JPA 수정** <a href="#id-3-jpa" id="id-3-jpa"></a>

Entity 객체에 값을 넣으면 Entity 스냅샷과 비교하여 변경된 쿼리로 쓰기 지연 저장소에 정보룰 변경하고 flush(), commit()을 통해서 DB에 저장된다.

<figure><img src="../../.gitbook/assets/image (108).png" alt=""><figcaption><p>JPA 수정 과정</p></figcaption></figure>

1. 개발자 : Entity 객체에 set
2. JPA : Entity 스냅샷과 비교하여 쿼리 생성 후 쓰기 지연 저장소에 저장
3. 개발자 : DB 동기화 요청. (flush)
4. JPA : DB 동기화 한다. ( DB에 반영 되지는 않음, 영속성 컨텍스트는 지워지지 않음)
5. 개발자 : DB 저장 요청 (commit)
6. JPA : DB에 정보를 최종 저장. ( 최종 DB에 반영 됨 )

<figure><img src="../../.gitbook/assets/image (109).png" alt=""><figcaption><p>로그 확인</p></figcaption></figure>

## **4. JPA Detach** <a href="#id-4-jpa-detach" id="id-4-jpa-detach"></a>

영속성 컨텍스트에서 준 영속성 상태로 전환하는 기능이다.

<figure><img src="../../.gitbook/assets/image (110).png" alt=""><figcaption><p>Detach 과정</p></figcaption></figure>

## **5. JPA Remove** <a href="#id-5-jpa-remove" id="id-5-jpa-remove"></a>

삭제 기능으로 영속성 컨텍스트 동작은 동일하다.

<figure><img src="../../.gitbook/assets/image (111).png" alt=""><figcaption><p>삭제 과정</p></figcaption></figure>
