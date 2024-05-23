# Core J2EE 패턴

* Intercepting Filter 패턴: 요청 타입에 따라 다른 처리를 하기 위한 패턴.
* Front Controller 패턴: 요청 전후에 처리하기 위한 컨트롤러를 지정하는 패턴.
* View Helper 패턴: 프레젠테이션 로직과 상관 없는 비즈니스 로직을 헬퍼로 지정하는 패턴.
* Composite View 패턴: 최소 단위의 하위 컴포넌트를 분리하여 화면을 구성하는 패턴.
* Service to Worker 패턴: Front Controller와 View Helper 사이에 디스패처를 두어 조합하는 패턴
* Dispatcher View 패턴: Front Controller와 View Helper로 디스패치 컴포넌트를 형성한다. 뷰 처리가 종료될 때까지 다른 활동을 지연하는 점이 Service to Worker 패턴과 다르다.
* Business Delegate 패턴: 비즈니스 서비스 접근을 캡슐화하는 패턴
* Service Locator 패턴: 서비스와 컴포넌트 검색을 쉽게 하는 패턴
* Session Facade 패턴: 비즈니스 티어 컴포넌트를 캡슐화하고, 원격 클라이언트에서 접근할 수 있는 서비스를 제공하는 패턴
* Composite Entity 패턴: 로컬 엔티티 빈과 POJO를 이용하여 큰 단위의 엔티티 객체를 구현한다.
* Transfer Object 패턴: 일명 Value Object 패턴이라고 많이 알려져 있다. 데이터를 전송하기 위한 객체에 대한 패턴이다.
* Transfer Object Assembler 패턴: 하나의 Transfer Object로 모든 데이터를 처리 할 수 없으므로, 여러 Transfer Object를 조합하거나 변형한 객체를 생성하여 사용하는 패턴이다.
* Value List Handler 패턴: 데이터 조회를 처리하고, 결과를 임시 저장하며, 결과 집합을 검색하여 필요한 항목을 선택하는 역할을 수행한다.
* Data Access Object 패턴: 일명 DAO라고 많이 알려져 있다. DB에 접근을 전달하는 클래스를 추상화하고 캡슐화한다.
* Service Activator 패턴: 비동기적 호출을 처리하기 위한 패턴이다.\


<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption><p><strong>Core J2EE Pattern Catalog</strong></p></figcaption></figure>

* **출처**: [http://www.corej2eepatterns.com/](http://www.corej2eepatterns.com/)
* **출처**: 자바성능튜닝이야기 -출판사:인사이트

