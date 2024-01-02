# Open Session In View

속성 컨텍스트를 뷰 렌더링이 끝나는 시점까지 개방한 상태를 유지 하는 것

## 1. 영속성 컨텍스트

* 도메인 레이어 객체들이 하부의 데이터 저장소와 영속성 매커니즘에 대해 알지 않아도 되는 투명한 영속성을 제공하는 프레임워크로 Hibernate사용
* Hibernate의 session객체가 영속성 컨텍스트를 관리함
* Transaction는 쪼갤 수 없는 업무처리의 단위로 영속성 컨텍스는 1:1:로 연결된하나의 Transaction동안 수정된 객체의 모든 상태는 영속성 컨텍스트 내에 저장 되고 종료 될 때 데이터 저장소와 동기 된다.

<figure><img src="../../.gitbook/assets/image (123).png" alt=""><figcaption></figcaption></figure>

## 2. Open Session In View를 이해하기 전에 알아여 할 개념&#x20;

* Transaction 범위 : Spring에서 @Transactional을 사용 하면 자동으로 Session이 열리고 객체의 변경 추적이 되기 시작되고 종료 시점에 쿼리 실행을 하여 데이터 저장소와 동기화 된다.
* 영속성 객체 상태 : Persistence, Detached, Transient, Removed
* 패치전략 : 쿼리 수행과 관련된 객체와 연관 관계를 맺고 있는 객체나 컬렉션을 어느 시점에 가져 올지에 대한 전략으로 EAGER(즉시로딩), LAZY(지연로딩)가 있음&#x20;
  * EAGER : 데이터를 즉시 가져 오는 전략&#x20;
  * LAZY : 데이터를 처음 액세스 될 떄 가져오는 전략
* 프록시 : 지연 패치 전략에 따라 연관 관계를 맺고 있는 객체와 컬렉션에 있은 실제 객체 처럼 위장된 프록시 객체 생성 후 실제 엔티티에 접근 할 때 영속성 컨텍스트에 생성 요청 하는 것을 프록시 초기화라고하며 한번만 초기화 된다.

## 3. Open Session In View

Service Layer는 Application의 Transaction경계를 정의하는 역할을 하게 되고, 이로인해 발생하는 문제가 Open Session In View으로  View Layer에서 연관 객체를 사용하려 할 때 발생하는 LazyInitialzationException 이 발생하는 것을 의미한다.

### <mark style="color:red;">3-1. 해결방안</mark>&#x20;

1. **뷰 렌더링에 필요한 객체 그래프 모두 로**딩 : \
   EAGER Fetch -> View와 강한 결합 -> 관심사의 분리 원칙 위배
2. **POJO FAÇ ADE Pattern** : \
   새로운 객체를 통해 프록시를 초기화한 후 사용자 인터페이스로 반환 하 는 방법 -> View에 사용하는 객체를 모두 로드 하는 방식
3. **Open Session In View** : \
   뷰 렌더링 시점에 영속성 컨텍스트가 존재 하지 않기 때문에 Detached 객체의 프록시를 초기화 할 수 없다면 영속성 컨텍스트를 오픈 된 채로 뷰 렌더링 시점까지 유지 하 는 것으로 서블릿 필터 시작 시에 Hibernate Session을 열고 Transaction 시작하고 종료 시점에 커밋하므로JDBC Connection 보유 시간 증가
4. **Spring Open Session In View** : \
   FlushMode 와 ConnectionReleaseMode의 조정을 통해 전통적인 서블릿 필터의 단점을 보안 하는 OpenSessionInViewFilter 와 OpenSessionInViewInterceptor 를 제공하여OpenSessionInViewFilter 는 필터 내에서 Session을 오픈하지만 트랜잭션은 시작하지 않음

<figure><img src="../../.gitbook/assets/image (125).png" alt=""><figcaption></figcaption></figure>

JSP 렌더링  오류발생  : org.hibernate.LazyInitializationException: proxy – no Session

<figure><img src="../../.gitbook/assets/image (126).png" alt=""><figcaption><p>내부 동작</p></figcaption></figure>

영속성 컨텍스트가 Detached 상태에서 사용하므로 LazyInitializationException 발생&#x20;

<mark style="color:purple;">**해결방안 : ThreadLocal Session 패턴 => Session per request**</mark>

1. **지연 로딩을 하지 않는다. (QueryDSL 사용 ) :** \
   \- Repository 재활용성 감소 \
   \- Repository 복잡도 증가 \
   \- View와 영속성 관심사 강한 결합
2. **POJO FAÇADE 패턴**\
   \- 모든 프록시를 초기화 \
   \- getHibernateTemplate().initialize() 사용 \
   \- 개발 난이도 증가
3. **OPEN SESSION IN VIEW 패턴**\
   \- 작업 단위를 요청 시작 부터 뷰 렌더링 시점까지 확장 \
   \- Filter 사용 -> Transaction 관리 : Spring 의 OpenSessionInViewFilter 설정

## 4.   영속성 컨텍스트 범위

### 4-1. 일반적인 영속성 컨텍스트, 트랜젹션, 커넥션 범위

<figure><img src="../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

### 4-2. Filter를 사용 한 영속성 컨텍스트, 트랜젹션, 커넥션 범위

<figure><img src="../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

### 4-3. Spring의 OpenSessionInViewFilter 영속성 컨텍스트, 트랜젹션, 커넥션 범위

<figure><img src="../../.gitbook/assets/image (129).png" alt=""><figcaption></figcaption></figure>

* 영속성 컨텍스트와 트랜잭션의 경제가 틀림
* 트랜잭션에 대한 일관성 있는 뷰를 제공 하지만 영속성 컨텍스 트에 대한 일관성 있는 뷰는 제공 하지 않음
  * Controller에서 영속성 컨텍스트가 활성화됨&#x20;
  * Transaction 경계 외부에서 영속성 컨텍스트 내에 저장
* OpenSessionInViewFilter=false 설정 :&#x20;
  * Transaction별로 Session 생성&#x20;
  * 서블릿필터 시작시에 Session 을오픈하지않는다.&#x20;
  * 대신 SessionFactory자체를 ThreadLocal에 저장한후 컨트롤러로 요청 처리를 위임한다.
* 기본적으로 HibernateTransactionManager는@Transactional 시점에 ThreaLocal에 저장 되어 있는 SessionFactory를 사용하여 Session 오픈&#x20;
  * singleSession=true -> 서블릿 핕터에서 Session 공유&#x20;
  * singleSession=false -> Transactional 시점에 Session 공유
    * JDBC Connection 증가됨

#### **4-3-1. singleSession=false 영속성 컨텍스트, 트랜젹션, 커넥션 범위**

<figure><img src="../../.gitbook/assets/image (130).png" alt=""><figcaption><p>singleSession=false 영속성 컨텍스트, 트랜젹션, 커넥션 범위</p></figcaption></figure>

4-3-2. Controller에서 두번 호출&#x20;

<figure><img src="../../.gitbook/assets/image (131).png" alt=""><figcaption></figcaption></figure>

* 1\), 2) 에서 조회한 객체는 동일함
* 만약 2)번에서 변경이 있었으면 Transaction은 두 번 실행됨

<mark style="color:purple;">OpenSessionInViewFilter를 사용하지 않으면 다른 객체에 대한 작업</mark>

<mark style="color:purple;">OpenSessionInViewFilter를 사용하면 동일객체에 대한 작업</mark>
