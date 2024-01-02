# Spring Data JPA

Spring Data JPA는 실제로 Hibernate, Jboss, EclipseLink와 같은 JPA 제공자가 아니고 라이브러리 또는 프레임 워크로 Spring Data의 프로젝트

## 1. Spring Data JPA

* Spring Data Repository 확장하여 Repository를 구
* CRUDRepository, JPARepository에서 제공하는 내장 CRUD 메소드를 사용
* 메소드 이름 규칙을 사용하여 메소드를 작성하거나 @Query 어노테이션이있는 조회를 제공하는 메소드를 작성
* 페이지 매김, 슬라이싱, 정렬 및 감사 지원
* XML 기반 엔터티 매핑 지원
* Specification and QueryDsl 제공

## 2. Spring Data JPA에서 지원하는 쿼리

* JPQL : 문자열 기반의 쿼리 언어
* Criteria Query : CriteriaBulider을 사용한 쿼리 생성로 동적 쿼리를 사용하기 위한 JPA 라이브러리 제공
* Native Query : JPA에서 SQL 쿼리 실행
* RDBMS 저장 프로시저 지원

## 3. Spring Data JPA Repository 계층 구조

<figure><img src="../../.gitbook/assets/image (99).png" alt=""><figcaption><p>계층 구조</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (100).png" alt=""><figcaption><p>Spring Data</p></figcaption></figure>

