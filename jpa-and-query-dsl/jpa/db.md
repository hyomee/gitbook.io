---
description: DB의 슈퍼타입 서브타입 논리 모델
---

# DB 슈퍼타입

<figure><img src="../../.gitbook/assets/image (83).png" alt=""><figcaption><p>수퍼 타입/서브타입</p></figcaption></figure>

**논리 모델을 물리 모델로 변경시 다음과 같은 방법으로 변환합니다**.

* 조인 테이블 : 조회 시 조인을 많이 사용 하므로 성능 이슈가 있으며 조회 쿼리가 복잡 하고 insert가 두 번 발생 합니다.
* 단일 테이블 : 조인이 발생 하지 않아서 성능이 빠르고 조회 쿼리가 단순 합니다.
* 복수 테이블 : Union 조인 필요합니다.

**객체 모델**

<figure><img src="../../.gitbook/assets/image (85).png" alt=""><figcaption><p>객체 모델</p></figcaption></figure>

* **전략 : @Inheritance(strategy= InheritanceType)**
  * default 전략은 SINGLE\_TABLE.
  * InheritanceType 종류&#x20;
    * JOINED&#x20;
    * SINGLE\_TABLE&#x20;
    * &#x20;TABLE\_PER\_CLASS
* **하위 구분 : @DiscriminatorColumn(name="DTYPE")**
  * 구분값에 들어갈 값 @DiscriminatorValue("XXX")

## 1. 조인 테이블

<figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption><p> 조인 테이블</p></figcaption></figure>

**strategy= InheritanceType.JOINED**

```java
@Entity
@Inheritance(strategy= InheritanceType.JOINED)
@DiscriminatorColumn(name="DTYPE")
public abstract class 상품 {
    @Id
    @GeneratedValue
    private int id
}

@Entity
@DiscriminatorValue(“A") {
public class 앨범 extends 상품 {
    private String 작곡가
}

@Entity
@DiscriminatorValue(“B") {
public class 영하 extends 상품 {
    private String 감독;
    private String 배우;
}
```

## 2. 단일 테이블

<figure><img src="../../.gitbook/assets/image (86).png" alt=""><figcaption><p>단일 테이블</p></figcaption></figure>

**strategy= InheritanceType.SINGLE\_TABLE**

```java
@Entity
@Inheritance(strategy= InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name="DTYPE")
public abstract class 상품 {
    @Id
    @GeneratedValue
    private int id
}

@Entity
@DiscriminatorValue(“A") {
public class 앨범 extends 상품 {
    private String 작곡가
}

@Entity
@DiscriminatorValue(“B") {
public class 영하 extends 상품 {
    private String 감독;
    private String 배우
}
```

## 3. 복수 테이블

<figure><img src="../../.gitbook/assets/image (87).png" alt=""><figcaption><p>복수 테이블</p></figcaption></figure>

**strategy= InheritanceType.TABLE\_PER\_CLASS**

```java
@Entity
@Inheritance(strategy= InheritanceType.TABLE_PER_CLASS)
public abstract class 상품 {
    @Id
    @GeneratedValue
    private int id
}

@Entity
public class 앨범 extends 상품 {
    private String 작곡가
}

@Entity
public class 영하 extends 상품 {
    private String 감독;
    private String 배우
}
```
