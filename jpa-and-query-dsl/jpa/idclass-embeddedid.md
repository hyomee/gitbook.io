# 복합키-@IdClass, @EmbeddedId

JPA에서 복합키는 @IdClass, @EmbeddedId 두 가지 방식을 제공 합니다.

| @IdClass                                                                                                                                                                          | @EmbeddedId                                                                                                                                                                                                           |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <ul><li>식별자, Entity 속성명 같아야 함</li><li>식별자 Class는 Serializable 구현되어 있어야 함.</li><li>equals, hasCode가 구현되어 있어야함.</li><li>생성자가 있어야 함 </li><li>식별자 클래스는 public 이어야 함<br><br></li></ul> | <ul><li>식별자 클래스는 식별자 클래스에기본키를 직접 매핑 </li><li>식별자 Class는 @Embeddedable 어노테이션 사용</li><li>식별자 Class는 Serializable 구현 되어야 함</li><li>equals, hasCode 구현되어야 함</li><li>기본 생성자가 있어야 함</li><li>식별자 클래스는 public 이어야 함</li></ul> |

<mark style="color:purple;">@IdClass가 db에 맞추어진 방법이고 @EmbeddedId는 객체지향적인 방법으로 복합 키에는 @GenerateValue를 사용 할 수 없습니다. 복합 키를 구성하는 여러 컬럼 중 하나에도사용 할 수 없습니다.</mark>

다음은 @IdClass, @EmbeddedId에 예제에 사용할 모델 입니다.

| DB                                           | 객체 관계                                        |
| -------------------------------------------- | -------------------------------------------- |
| ![](<../../.gitbook/assets/image (138).png>) | ![](<../../.gitbook/assets/image (139).png>) |

## &#x20;1. @IdClass

가입별상품key 객체를 Serializable 를 상속받아 만들고 가입별상품엔티티 객체에 @IdClass로 복합키 설정을 합니다. 상품부가엔티티는 oneToone 관계로 설정 합니다.

```java
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
@AllArgsConstructor
@NoArgsConstructor
Public class 가입별상품key implements Serializable {
    @id
    @EqualsAndHashCode.Include
    private int 가입별상품누적번호;
    
    @id
    @EqualsAndHashCode.Include
    private int 가입번호
}

@Entity
@IdClass(가입별상품key.class)
Public class 가입별상품{
    private int 가입별상품누적번호;
    private int 가입번호
    private String 서비스코드;
    priavte String 상품코드;
}

@Entity
Public class 상품부가{
    @Id
    private int 상품부가;
    
    @ManyToOne
    @JoinColumns( { @JoinColumn(name=“가입상품누적번호”, 
                                refrencedColumn=“가입상품누적번호”),
    @JoinColumn( name=“가입번호”, 
                 refrencedColumn=“가입번호”) })
    private 가입별상품 가입별상품
}
```

## 2. @EmbeddedId

가입별상품Key 엔티티에 @Embeddable를 적용하고 가입별상품 엔티티에서 @EmbeddedId로 참조하게 합니다.

```java
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
@AllArgsConstructor
@NoArgsConstructor
@Embeddable
Public class 가입별상품key implements Serializable {
    @id
    @EqualsAndHashCode.Include
    private int 가입별상품누적번호;
    
    @id
    @EqualsAndHashCode.Include
    private int 가입번호
}

@Setter
@Getter
@Entity
Public class 가입별상품{
    @EmbeddedId
    가입별상품key 가입별상품key;
    
    private String 서비스코드;
    priavte String 상품코드;
}
```

## 3.  복합키 연관

다음과 같은 모델을 생각해 봅니다.

<figure><img src="../../.gitbook/assets/image (140).png" alt=""><figcaption><p>가입별 상픔</p></figcaption></figure>

가입자는 상품을 여러 개 가지고 있으면 각 상품은 고유의 부가 정보를 가지고 있는 것을 모델로 정의 하면 위와 같은 모델이 나옵니다. 몰론 방법은 다양하지만 복합키 연관을 위해 작성한 예 입니다.

```java
@Entity
Public class 가입{
    @id
    @GeneratedValue
    @Column(name=“ENTR_NO”)
    private int 가입번호
    String 서비스코드
}

@Entity
Public class 가입별상품{
    @id
    @GeneratedValue
    @Column(name=“가입상품누적번호”)
    private int 가입상품누적번호
    
    @id
    @ManyToOne
    @JoinColumn(name=“가입번호”)
    private int 가입번호
    private String 서비스코드
    
    @OneToOne(mappedBy = “가입별상품”)
    private 가입별상품부가 가입별상품부가
}

@Entity
Public class 가입별상품부가{
    @Id
    private int 가입상품누적번호
    
    @MapsId
    @OneToOne
    @JoinColumn(name= “가입상품누적번호”)
    private 가입별상품 가입별상품;
    
    private String 대리점코드
}
```

```
@MapsId : OneToOne 관계에서 PK 이면서 FK인 경우 사용 
```
