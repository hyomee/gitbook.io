# @JoinTable

테이블의 연관 관계를 정규화를 진행하여 별도의 테이블을 통해서 하는 방식을 JPA에서는 객체 관계를 JOIN TABLE로 해결 할 떄 사용 합니다.

#### @JoinTable&#x20;

* name : 매핑할 조인 테이블 이름
* joinColumn : 현재 엔티티를 참조하는 외래키&#x20;
* inverseoinColumn : 반대방향 엔티티를 참조하는 외래키

## 1. 일대일 JoinTable

| 모델                                                                                      | 부모- 자식 매핑 테이블                                |
| --------------------------------------------------------------------------------------- | -------------------------------------------- |
| <p><img src="../../.gitbook/assets/image (148).png" alt=""><br><br><br><br><br><br></p> | ![](<../../.gitbook/assets/image (149).png>) |



부모 - 자식 ( 1 : 1 ) 관계에 중간에 부모ID와 자삭 ID의 PK만으로 구성되는 부모-자식 테이블을 생성 하여 관리 하는 것으로 기본키의 경우 유니크 제약 조건이 걸려있기 때문에 기본키가 아닌 외래 키에 대한 유니크 제약 조건만 설정해 주면 됩니다.

<pre class="language-java"><code class="lang-java">@Entity
Public class 부모 {
    @id 
    @GeneratedValue
    @Column(name=“부모ID”)
    private long 부모ID;
    private String 이름;
    
    @OneToOne
    @JoinTable(name=“부모_자식”
                JoinColumns={ @JoinColumn(name=“부모ID”,
                                    referencedColumnName = "부모ID") },
                InverseJoinColumns= { @JoinColumn=“자식ID”, 
                                    referencedColumnName = "자식ID") } )
    private 자식 자식;
}

@Entity
Public class 자식 {
<strong>    @id @GeneratedValue
</strong>    @Column(name=“자식ID”)
    private long 자식ID;
    
    private String 이름;
}
</code></pre>

## 2. 일대다 JoinTable

| 모델                                                                                      | 부모-자식 테이블 매핑                                 |
| --------------------------------------------------------------------------------------- | -------------------------------------------- |
| <p><img src="../../.gitbook/assets/image (150).png" alt=""><br><br><br><br><br><br></p> | ![](<../../.gitbook/assets/image (154).png>) |

기존의 부모-자식의 1 : N 연관을 부모-자식 테이블을 생성 하여 부모 : 부모-자식 테이블은 1:N 관계를 만들고 부모-자식 테이블 과 자식 테이블은 1 : 1관계를 만들어서 매핑하는 것으로 부모-자식 테이블에서 자식 ID를 PK로 부모ID를 FK로 정의합니다.

```java
@Entity
Public class 부모 {
    @id @GeneratedValue
    @Column(name=“부모ID”)
    private long 부모ID;
    private String 이름;
    
    @OneToMany
    @JoinTable(name=“부모_자식”
            JoinColumns=@JoinColumn(name=“부모ID”),
            InverseJoinColumns=@JoinColumn=“자식ID”))
    private LIst<자식> 자식 = new ArrayList<자식>();
} 

@Entity
Public class 자식 {
    @id @GeneratedValue
    @Column(name=“자식ID”)
    private long 자식ID;
    private String 이름;
}
```

## 3. 다대일 JoinTable

| 모델                                                                                          | 부모-자식                                        |
| ------------------------------------------------------------------------------------------- | -------------------------------------------- |
| <p><img src="../../.gitbook/assets/image (152).png" alt=""><br><br><br><br><br><br><br></p> | ![](<../../.gitbook/assets/image (155).png>) |

1:N 연관 관계에서는 1쪽에 @OneToMany 어노테이션과 함께 @JoinTable 관련 설정을 한 반면, N:1 연관 관계에서는 N 쪽에 @ManyToOne 어노테이션과 함께 @JoinTable 관련 설정을 하게 됩니다.&#x20;

@ManyToOne 어노테이션의 optional 속성의 경우 값이 ture일 때, 해당 객체에 null이 들어갈 수 있는데요.여기서는 false로 설정함으로써 연관된 Entity_(=_ 부모_)_가 항상 있어야 한다는 것이 보장됩니다.

_(optional = false 속성으로 인해 연관된 Entity가 항상 있기 때문에 inner join을 사용할 수 있습니다.)_

```java
@Entity
Public class 자식{
    @id 
    @GeneratedValue
    @Column(name=“자식ID”)
    private long 자식ID;
    
    @ManyToOne(optional=false)
    @JoinTable(name=“부모_자식”
                JoinColumns=@JoinColumn(name=“자식ID”),
                InverseJoinColumns=@JoinColumn=“부모ID”))
    private 부모 부모;
}

@Entity
Public class 부모{
    @id 
    @GeneratedValue
    @Column(name=“부모ID”)
    private long 부모ID;
    private String 이름;
    
    @OneToMany(mapperdBy=“부모”)
    private LIst<자식> 자식= new ArrayList<자식>();
}
```

## 4. 다대다 JoinTable

![](<../../.gitbook/assets/image (156).png>)

\
조인 테이블의 두 컬럼을 하나로 복합 유니크 제약 조건을 걸어야 합니다. &#x20;

```java
@Entity
Public class 부모 {
    @id 
    @GeneratedValue
    @Column(name=“부모ID”)
    private long 부모ID;
    private String 이름;
    
    @ManyToMany
    @JoinTable(name=“부모자식”
        JoinColumns=@JoinColumn(name=“부모ID”),
        InverseJoinColumns=@JoinColumn=“자식ID”));
    private LIst<자식> 자식 = new ArrayList<자식>();
} 

@Entity
Public class 자식 {
    @id 
    @GeneratedValue
    @Column(name=“자식ID”)
    private long 자식ID;
    private String 이름;
}
```
