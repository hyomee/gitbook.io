# 단방향 매핑

객체의 관계를 한 쪽 방향으로만 연관관계를 설정하는 것이다.

<figure><img src="../../../.gitbook/assets/image (102).png" alt=""><figcaption><p>DB, 객체 연관</p></figcaption></figure>

* DB : 테이블과 데이블을 JOIN을 통해서 양방향 관계 설정
  * 사용자  입장 : 사용자는 조직을 가지고 있음&#x20;
  * 조직  입장:  조직은 여러 사용자를 가지고 있음
* 객체 : 객체와 객체의 관계
  * 사용자는 조직에 속해 있지 않거나 하나의 조직을 가지고 있음
  * 조직은 사용자를 가지고 있지 않음 ( 맴버 변수 없음 )

## 1. 연관  관계 없는 코드

사용자 엔티티, 조직 엔티티는 어떤 연관 관계 없다.

<pre class="language-java"><code class="lang-java">@Entity
public class 사용자 {
    private String 사용자ID;
    private String 사용자이름;
    private String 조직ID;
}

@Entity
public class 조직 {
<strong>    private String 조직ID;
</strong>    private String 조직이름;
}
</code></pre>

사용자 조직 서비스로 사용자 생성시 다음과 같은 순서로 코드를 작성한다**.**

1. 조직 정보를 생성
2. 사용자 객체에 조직 설정
3. 사용자 정보 생성&#x20;

```java
@Service
public class 사용자조직 {
    public void 사용자조직_생성(사용자 user) {
        사용자 user = new 사용자();
        조직 team = new 조직();
        // 조직 설정 
        team.set 조직이름(“A팀”);
        
        // 조직 생성
        em.persist(team);      
        
        // 사용자에 조직 설정          
        user.set조직ID(team.get조직ID());   
        
        // 사용자 저장 
        em.persist(user);
    } 
}
```

사용자 정보를 조회 하는 메서드로 사용자와 사용자가 속한 조직을 조회 하기 위해서는 다음 순서로 코드를 작성해야 한다.

1. 사용자 정보 조회
2. 조회한사용자 정보에서 조직ID로 조직 조회
3. 사용자와 조직 정보를 모두 가지고 있는 사용자\_조직객체에 설정&#x20;

```java
public 사용자_조직  조회사용자조직(사용자 user_i) {
    // 사용자 조회 
    사용자 user = em.find(user_i자);  
    
    // 사용자로 조직 조회 
    조직 team = em.find(조직, user .get조직ID); 
    
    // 이하 사용자_조직 정보 설정     
} 
```

## 2. 연관  관계 있는 코드

사용자 엔티티에 조직 엔티티를 참조하고 N:1 매핑,  Join 컬럼으로 조직의 조직ID 설정한다.

```java
@Entity
public class 사용자 {
    private String 사용자ID;
    private String 사용자이름;
    
    @ManyToOne
    @JoinColumn(name=“조직ID”)
    private 조직 조직;
}

@Entity
public class 조직 {
    @Id
    @Column(name=“조직ID”)
    private String 조직ID;
    
    private String 조직이름;
}
```

사용자 조직 서비스로 사용자 생성시 다음과 같은 순서로 코드를 작성한다**.**

1. 조직 정보 설정
2. 사용자 정보 설정
3. 사용자 저장&#x20;

```java
@Service
public class 사용자조직 {
    public void 생성사용자조직(사용자 사용자_I) {
        // 사용자 Entity 설정 
        사용자 user = new 사용자();
        
        // 조직  Entity 설정             
        조직 team = new 조직();
        team.set 조직이름(“A팀”); 
        
        // 사용자 Entity 설정
        user.set("사용자이름", 사용자_I.get사용자());
        user.set 조직(team)
        
        // 사용자 정보 생성 
        em.persist(user);  
    } 
    
    public 사용자 조회사용자조직(사용자 사용자_I) {
    
        // 사용자 정보 조회 
        // 조직 정보를 같이 조회 하지 않기 위해서는 Lazy 사용 해야 함 
        사용자 user = em.find(사용자_I));  
        
        // 조직을 사용하는 경우 쿼리 수행 없이 조직 정보 사용 
        조직 team = 사용자.get조직();
    } 
}
```

사용자 정보를 조회 하는 메서드로 사용자와 사용자가 속한 조직을 조회 하기 위해서는 다음 순서로 코드를 작성해야 한다.

1. 사용자 정보 조회
2. 필요에 따라 조직 정보를 사용자 객체에서 꺼내서 사용&#x20;
   1. 조직 정보 조회 하지 않음&#x20;
   2. 단. Lazy 옵션 사용시 조회 됨 ( 성능 고려  )

```java
@Service
public class 사용자조직 {
    public 사용자 조회사용자조직(사용자 사용자_I) {
    
        // 사용자 정보 조회 
        // 조직 정보를 같이 조회 하지 않기 위해서는 Lazy 사용 해야 함 
        사용자 user = em.find(사용자_I));  
        
        // 조직을 사용하는 경우 쿼리 수행 없이 조직 정보 사용 
        조직 team = 사용자.get조직();
    } 
}
```

## 3. 차이점

연관 관계 없는 코드와 연관 관계 있는 코드의 수행 코드 작성 순서나 수행 순서는 크게 차이는 없지만 쿼리 수행 코드에 차이가 있다.

연관 관계 없는 코드에서는 개발자가 엔티티에 대해서 업무 흐름에 따라서 직접 조회, 저장을 해야 하지만 연관 관계 있는 코드는 주인 엔티티에 대해서만 조회, 저장을 관리하며 된다. 즉 개발자는 DB 처리 보다는 업무처리를 중싱므로 개발할 수 있다는 것이다.&#x20;

그러나 <mark style="color:red;">모든것은 잘 사용 할 때로 JPA의 연관을 잘못 사용하면 성능 또는 업무처리에 문제가 발생 할 수 있다는 것을 생각해야 한다.</mark>

### 참고.1   @JoinColumn

* 조인컬럼은 외래키를 매핑
* name 속성에는 매핑할 외래키 이름을 지정
  * 속성 :&#x20;
    * name \
      : 매핑할 외래 키 이름\
      : 필드명 + \_ + 참조하는 테이블의 기본키 컬럼명 :&#x20;
    * referecedColumnName \
      : 외래키가 참조하는 대상 테이블의 컬럼명 \
      : 참조하는 테이블의 기본키 컬럼명&#x20;
    * foreignKey \
      : 외래키 제약조건을 직접 지정 가능 \
      : DDL 생성 때만 사용&#x20;
    * : unique, nullable, insertable, updatable, columnDefinition, table \
      : @Column의 속성과 같다.

### 참고.2  .@ManyToOne

* 다대일()관계임을 명시.
* name 속성에는 매핑할 외래키 이름을 지정
  * 속성 :&#x20;
    * optional \
      : false로 설정하면 연관된 엔티티가 항상 있어야 함 \
      : 기본값 : true&#x20;
    * fetch \
      : 글로벌 페치 전략 설정 \
      : FetchType.EAGER, FetchType.LAZY&#x20;
    * cascade \
      : 영속성 전이 기능 사용&#x20;
    * targetEntity \
      : 연관된 엔티티의 타입 정보를 설정. 거의 사용하지 않음
