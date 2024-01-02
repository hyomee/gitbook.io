# Auditing 사용자 정의

테이블에 CRUD 작업을 해야 할 떄 생성일자, 변경일자, 등록자, 수정자 필드에 작업은 공통으로 적용이 되는 작업 입니다. JPA를 적용 할 때 @EntityListeners 에노테이션을 사용 하면 쉽게 적용 할 수 있습니다. @EntityListeners 주석은&#x20;

```
Specifies the callback listener classes to be used for an entity or mapped superclass. 
This annotation may be applied to an entity class or mapped superclass
```

과 같이 작성되어 있습니다.  Spring-Data-Jpa에서 제공 하는 기본 에노테이션을 사용하여 생성 일자와 수정일자를 코드 개발 없이 사용 할 수 있습니다. 다음은 코드 개발 없이 적용 할 수 있는 코드 입니다

## 1.요구 사항 :&#x20;

TB\_EVENT\_DEMO 테이블에 신규 등록시 자동으로 생성일자에 날짜를 적용하고 테이블의 내용이 변경이 되면 자동으로 수정 일자를 적용해야 한다.&#x20;

```java
// Entity에 적용할 공통 기능 
@EntityListeners(AuditingEntityListener.class)
public class AuditVO {

// create_Date_Time 컬럼, insert 적용, update 불가 
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createDateTime; 

// update_date_time  컬럼, insert 적용, update 적용 
    @LastModifiedDate
    private LocalDateTime updateDataTime;
    
}

// TB_EVENT_DEMO 테이블 
// AuditVO를 확장 하여 DemoEventEntity에 정의된 컬럼 외 
// create_Date_Time, update_date_time 컬럼 추가 
@Table(name = "TB_EVENT_DEMO")
@Entity
public class DemoEventEntity extends AuditVO {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private int id;
    private String title;
    private String content;
    private String name;
}

// Spring-Data-Jpa에서 제공하는 Auditing기능 활성화 
@EnableJpaAuditing
@SpringBootApplication()
public class JpaServiceApplication  {
    public static void main(String[] args) {
        SpringApplication.run(JpaServiceApplication.class, args);
    }
}
```

@EnableJpaAuditing 선언으로 Auditing 기능이 활성화(: JpaAuditingRegistrar )되고 Entity에 적용된 @EntityListeners에 지정한 클래스  AuditingEntityListener.class에 의해서 Entity에 작업이 발생 하면 리스너기 작동을 히여 지정한 형식으로 적용이 되어 요구 사항에 만족 하는 코드가 됩니다.

AuditingEntityListener 클래스는 AuditingHandler에 등록된 값을 이용해서 @PrePersist, @PreUpdate가 작동 될 떄 해당 값을 설정 하도록 하는 기능 입니다.

```java
@Configurable
public class AuditingEntityListener {

	private @Nullable ObjectFactory<AuditingHandler> handler;

	/**
	 * Configures the {@link AuditingHandler} to be used to set the current auditor on the domain types touched.
	 *
	 * @param auditingHandler must not be {@literal null}.
	 */
	public void setAuditingHandler(ObjectFactory<AuditingHandler> auditingHandler) {

		Assert.notNull(auditingHandler, "AuditingHandler must not be null");
		this.handler = auditingHandler;
	}

	/**
	 * Sets modification and creation date and auditor on the target object in case it implements {@link Auditable} on
	 * persist events.
	 *
	 * @param target
	 */
	@PrePersist
	public void touchForCreate(Object target) {

		Assert.notNull(target, "Entity must not be null");

		if (handler != null) {

			AuditingHandler object = handler.getObject();
			if (object != null) {
				object.markCreated(target);
			}
		}
	}

	/**
	 * Sets modification and creation date and auditor on the target object in case it implements {@link Auditable} on
	 * update events.
	 *
	 * @param target
	 */
	@PreUpdate
	public void touchForUpdate(Object target) {

		Assert.notNull(target, "Entity must not be null");

		if (handler != null) {

			AuditingHandler object = handler.getObject();
			if (object != null) {
				object.markModified(target);
			}
		}
	}
}
```

만약 생성자와 수정자 등을 추가 해서 로그인 사용자 또는 작업자를 관리 해야 하는 요구 사항이 추가 되면 AuditingHandler에 등록 해서 사용 해야 한다는 것을 알 수 있습니다. 아래는 AuditingHandler의 일부 코드 입니다. AuditorAware\<?> 라는 맴버 변수를 가지고 있고 해당 값애 추가 하도록 되어 있습니다.&#x20;

```java
package org.springframework.data.auditing;

public class AuditingHandler extends AuditingHandlerSupport implements InitializingBean {

	private static final Log logger = LogFactory.getLog(AuditingHandler.class);

	private Optional<AuditorAware<?>> auditorAware;
    ....
    
    Auditor<?> getAuditor() {

		return auditorAware.map(AuditorAware::getCurrentAuditor).map(Auditor::ofOptional) //
				.orElse(Auditor.none());
	}
    ....
}
```

즉, AuditorAware\<?>의 구현체를 만들고 Bean 등록을 하면 Spring가 알아서 처리 하겠다는 이야기 입니다. 다음과 같은 작업을 통해서 추가 할 수 있습니다.

1. &#x20;AuditorAware\<String>의 구현체 작성
2. 작성한 구현체를 Bean 등록&#x20;

\*\*\*\* 등록자와 수정자가 추가된 클레스&#x20;

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class AuditVO {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createDateTime;

    @CreatedBy
    @Column(updatable = false)
    private String createUserid;

    @LastModifiedDate
    private LocalDateTime updateDataTime;

    @LastModifiedBy
    private String updateUserId;
}
```

### **1.  AuditorAware\<String>의 구현체 작성**

AuditorAware\<String>의 getCurrentAuditor()에서 필요한 코드로 권한 사용자 또는 세션에 있는 사용자 또는 요구사항에 맞게 코드를 작성 합니다.\


```java
public class AuditorAwareImpl implements AuditorAware<String> {
  @Override
  public Optional<String> getCurrentAuditor() {
    String userid = "Test";
    // 이곳에 권한 사용자 : - Authentication authentication =
    //            SecurityContextHokder.getContext().getAuthentication()
    // Session 정보 사용 : - RequestScopeUtil.getAttribute().getUserId()
     return Optional.of(userid);
  }
}
```

### **2. 작성한 구현체를 Bean 등록**&#x20;

Bean 등록 코드에 @EnableJpaAuditing를 작성 합니다.&#x20;

```java
@EnableJpaAuditing
@Configuration
public class AuditConfig {

  @Bean
  public AuditorAware<String> auditorAware() {
      return new AuditorAwareImpl();
  }
}
```

3\. 결과&#x20;

* 신규 저장 : PrePersist  로그를 확인 하면 등록일자, 생성자, 수정일자, 수정자가 할당 되어 있는 것을 확인 할 수 있습니다.

```bash
curl --location 'localhost:8080/demoevent' \
--header 'Content-Type: application/json' \
--data '{
    "name": "신규저장",
    "title": "24동안 하는일12",
    "content": "24시간 동안 장을 잔다다"
}'
===================================================================================
[hyomee] [2023-09-03 23:44:23.161] [http-nio-8080-exec-8] INFO  
- com.hyomee.service.tour.demo.entity.DemoEventEntity 
> PrePersist : DemoEventEntity(super=AuditVO(
createDateTime=2023-09-03T23:44:23.157782400, 
createUserid=Test, 
updateDataTime=2023-09-03T23:44:23.157782400, 
updateUserId=Test), 
id=0, title=24동안 하는일12, content=24시간 동안 장을 잔다다, name=신규저장)
```

***

* 수정 저장 : PreUpdate로그를 확인 하면 등록일자, 생성자는 null 적용 하지 않고 수정일자, 수정자 만 자동 할당 되는 것을 확인 할 수 있습니다.

```bash
curl --location 'localhost:8080/demoevent' \
--header 'Content-Type: application/json' \
--data '{
    "id": "3",
    "name": "김김",
    "title": "24동안  ",
    "content": "24시간 동안 "
}'
=====================================================================
[hyomee] [2023-09-03 23:48:31.896] [http-nio-8080-exec-2] INFO  c.h.j.event.EntityEventCallListener 
> PreUpdate : DemoEventEntity(super=AuditVO(c
reateDateTime=null, 
createUserid=null,
updateDataTime=2023-09-03T23:48:31.896108600, 
updateUserId=Test), 
id=3, title=24동안  , content=24시간 동안 , name=김김)
```

***

* 결과 조회 : 생성 일자와 수정일자가 틀린 것을 확인 할 수 있습니다.

```bash
curl --location 'localhost:8080/demoevent/김김' \
--header 'Content-Type: application/json'

최종 결과 
{
    "createDateTime": "2023-09-03 11:44:23",
    "createUserid": "Test",
    "updateDataTime": "2023-09-03 11:48:31",
    "updateUserId": "Test",
    "id": 3,
    "title": "24동안  ",
    "content": "24시간 동안 ",
    "name": "김김"
}
```

\* 등록자, 수정자가 변경 되지 않는 이유는 AuditorAwareImpl 클래스의 getCurrentAuditor() 코드 에서 "Test"로 작성이 되어 있어서 입니다.
