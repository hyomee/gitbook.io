---
description: DB 입장에서는 존재 할 수 없음
---

# 다대다 \[ N : N ]

DB 설계시에도연 N:N 관계는 잘못된이  설계로 만들지 말아야 합니다. 이런 경우 매핑 테이블을 만들어서 설계를 해야 합니다.&#x20;

<figure><img src="../../../.gitbook/assets/image (88).png" alt=""><figcaption><p>DB 매핑 관계</p></figcaption></figure>

위의 DB 매핑을 객체 관계로는 N:N 단방향과 N:N 양방향으로 풀 수 있습니다.

<figure><img src="../../../.gitbook/assets/image (90).png" alt=""><figcaption><p>객체 관계</p></figcaption></figure>

## 1.  N:N 단방향

* @ManyToMany, @JoinTable
* 사용자 전화번호 테이블 설정 되고 제약 조건 설정 됨

<figure><img src="../../../.gitbook/assets/image (91).png" alt=""><figcaption><p>N:N 단방항</p></figcaption></figure>

```java
@Entity
public class 사용자{
    private String 사용자ID;
    private String 사용자이름;
    
    @ManyToMany
    @JoinTable(name=“사용자전화번호”)
    private List<전화번호> 전화번호s 
}
```

## 2. N:N 양방향

* @ManyToMany(mappedBy=“대상객체”) 설정
* 사용자 전화번호 테이블 설정 되고 제약 조건 설정 됨

<figure><img src="../../../.gitbook/assets/image (92).png" alt=""><figcaption><p>N:N 양방향</p></figcaption></figure>

```java
@Entity
public class 전화번호{
    private String 사용자ID;
    private String 사용자이름;
    
    @ManyToMany(mappedBy=“전화번호”)
    private List< 사용자 > 사용자s
}
```

## 3. 한계

* 중간 테이블이 생성 됨
* 중간 테이블에 다른 정보 삽입 불가
* 잘못된 쿼리 수행
* <mark style="color:red;">**사용 하면 안됨**</mark>

## 4. 해결방안

* <mark style="color:purple;">**중간 테이블을 엔티티로 승격**</mark>
* <mark style="color:purple;">**다대일 관계, 일대다 관계로 해결**</mark>



