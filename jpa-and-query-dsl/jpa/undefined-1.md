---
description: '@MappedSuperclass'
---

# 공통 클래스

* 부모Class는 테이블에 매핑 하지 않고 자식 Class에게 매핑 정보 제공
* 추상 Class와 비슷, 테이블과 매핑 되지 않음
* 엔티티가 아니므로 조회가 불가함
* 공통 요소를 추상 Class 로 만들고 어노테이션 추가

**@Entity 객체는 @Entity, @MappedSuperclass 만 상속 가능**

## 1. 일반 적인 상속

<figure><img src="../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

```java
@Setter
@MappedSuperclass
public abstract class 공통요소 {​
    private String 생성자;​
    private LocalDateTime 생성일자;​
    private String 수정자;​
    private LocalDateTime 수정일자;
}
@Entity
public class 사용자 extends 공통요소 {
...
}
@Entity
public class 전화번호 extends 공통요소 {
...
}

```

## 2. 공통 컬럼에 대한 재정의 하는 방법

* @AttributeOverrides, @AttributeOverride : 매핑 정보 재정의
* @AssociationOverrides, @AssociationOverride : 연관 관계 재정의

```java
@Getter
@Setter
@MappedSuperclass
public abstract class 공통요소 {​
    @Id, 
    @GeneratedValue
    private Long id;
    private String name
}

@Entity
@AttributeOverride( name =id, column=@Column(name=“MEMBER_ID” ))
public class 사용자 extends 공통요소 {
...
}

@Entity
@AttributeOverrides( {
    @AttributeOverride (name =id, column=@Column(name=“MEMBER_ID”) ),
    @AttributeOverride (name =name, column=@Column(name=“MEMBER_NAME” )
public class 사용자 extends 공통요소 {
...
}
```

