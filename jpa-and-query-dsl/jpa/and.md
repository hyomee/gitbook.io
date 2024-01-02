# 프록시 & 지연로딩

## 1. 프록시

JPA는 find(), getReference() 제공하며 getReference()가 가상 객체 조회하는데  실제 DB 조회 하지 않고 사용 시점에 DB 조회 합니다.

준 영속성 상태에서 getReference는 예외 발생 하고,  트랜잭션 범위 밖에서 프록시 객체 조회 하는 경우 open-session-in-view 패턴을 사용해야 합니다.

<table><thead><tr><th width="271">위임</th><th>동작 구조</th></tr></thead><tbody><tr><td><img src="../../.gitbook/assets/image (79).png" alt=""></td><td><img src="../../.gitbook/assets/image (81).png" alt="" data-size="original"></td></tr></tbody></table>

## 2. 즉시 로딩/지연 로딩

사용 시점에 쿼리 수행하는 것으로&#x20;

* @ManyToOne(fetch = FetchType.EAGER) : 기본은 즉시 로딩&#x20;
  * @ManyToOne, @OneToOne : optional = false : 내부 조인 : optional = true : 외부 조인
  * @OneToMany, @ManyToMany : optional = false : 외부 조인 : optional = true : 내부 조인
* @ManyToOne(fetch = FetchType.LAZY)  :  지연로딩&#x20;

<figure><img src="../../.gitbook/assets/image (82).png" alt=""><figcaption><p>지연로딩</p></figcaption></figure>

## 3. 영속성 전이

부모 객체 저장시 자식 객체 동시 저장&#x20;

* @OneToMany(mapprdBy=“p” cascade = CasCadeType.PERSIST):

## 4. 고아 객체

* orphanRemoval = true&#x20;
  * @OneToMany, @ManyToOne 만 사용
* 객체가 제거 되면 남아 있는 객체를 의미 하며 true 설정 하면 자동 삭제 FetchType.EAGER&#x20;
  * db 삭제 됨
