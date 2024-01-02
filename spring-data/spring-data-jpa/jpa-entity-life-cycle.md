# JPA Entity Life Cycle

## **1. JPA Entity Life Cycle Event**

JPA 사용할 때 Entity가  어떻게 동작하는지 확인할 수 있습니다. 확인은 JPA는 엔티티의 생명주기 동안 일어나는 일곱 가지 선택적인 이벤트를 통해서 확인합니다. Entity Life Cycle Event는 이벤트가 발생할 때 콜백 메서드를 실행할 수 있도록 애노테이션을 다음과 같이 제공합니다.

* 새 엔터티에 대해 persist가 호출되기 전에 :  @PrePersist
* 새 엔터티에 대해 persist가 호출된 후 :  @PostPersist
* 엔터티가 제거되기 전에 :  @PreRemove
* 엔터티가 삭제된 후 :  @PostRemove
* 업데이트 작업 전 :  @PreUpdate
* 엔터티가 업데이트된 후  :  @PostUpdate
* 엔터티가 로드된 후  :  @PostLoad

이벤트를 처리하기 위해 애노테이션을 사용하는 방법에는&#x20;

1. Entity에 직접 애노테이션 사용 :\
   엔티티 클래스에 콜백 메소드를 정의하고, 각 메소드에 알맞은 애노테이션을 붙여서 이벤트를 처리
2. EntityListener 사용 :\
   엔티티 리스너 클래스를 만들고, 콜백 메소드에 애노테이션을 붙여서 이벤트를 처리

### 1-1. Entity에 직접 애노테이션&#x20;

DemoEntity 클래스에 7가지 애노테이션을 작성하여 Entity Life Cycle에 따른 객체의 값을 출력하도록 작성된 코드입니다.

```java
public class DemoEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private int id;
    private String title;
    private String content;
    private String name;

    @PrePersist
    public void prePersistDemoEntity() {
        log.info("PrePersist: " + this.toString());
    }

    @PostPersist
    public void postPersisttDemoEntity() {
        log.info("PostPersist : " +  this.toString());
    }

    @PreRemove
    public void preRemoveDemoEntity() {
        log.info("PreRemove : " + this.toString());
    }

    @PostRemove
    public void logUserRemoval() {
        log.info("PostRemove : " + this.toString());
    }

    @PreUpdate
    public void logUserUpdateAttempt() {
        log.info("PreUpdate : " + this.toString());
    }

    @PostUpdate
    public void logUserUpdate() {
        log.info("PostUpdate : " + this.toString());
    }

    @PostLoad
    public void logUserLoad() {
        log.info("PostLoad : " + this.toString());
    }
}
```

***

#### 1-1-1. 등록( 저장 )

demoRepository.save(demoEntity) 가 실행이 되면 다음 순서로 처리됩니다.

1. PrePersist  : id = 0 :&#x20;
2. next value : GenerationType.SEQUENCE 전략
3. insert : 쿼리 실행
4. PostPersist : id = 2&#x20;

```bash
curl --location 'localhost:8080/jpademo' \
--header 'Content-Type: application/json' \
--data '{
    "name": "홍퍼플",
    "title": "24동안 하는일",
    "content": "24시간 동안 잠을 잔다"
}'

===========================================================================================
[hyomee] [2023-09-02 22:57:14.632] [http-nio-8080-exec-2] INFO  c.h.s.tour.demo.entity.DemoEntity - 
PrePersist: DemoEntity(id=0, title=24동안 하는일, content=24시간 동안 잠을 잔다, name=홍퍼플)

[Hibernate] 
    select
        next value for tb_demo_seq

[Hibernate] 
    /* insert for
        com.hyomee.service.tour.demo.entity.DemoEntity */insert 
    into
        tb_demo (content,name,title,id) 
    values
        (?,?,?,?)
        
[hyomee] [2023-09-02 22:57:14.820] [http-nio-8080-exec-2] INFO  c.h.s.tour.demo.entity.DemoEntity - 
PostPersist : DemoEntity(id=2, title=24동안 하는일, content=24시간 동안 잠을 잔다, name=홍퍼플)
```

***

#### 1-1-2 조회

demoRepository.findByNameContains(name, pageable) 가 실행이 되면 쿼리 -> postLoad 순서로 실행이 되는 것을 확인할 수 있습니다.

```bash
curl --location 'localhost:8080/jpademo/page/홍?size=10&page=0' \
--header 'Content-Type: application/json'

===========================================================================================
[Hibernate] 
    /* <criteria> */ select
        d1_0.id,
        d1_0.content,
        d1_0.name,
        d1_0.title 
    from
        tb_demo d1_0 
    where
        d1_0.name like ? escape '\' offset ? rows fetch first ? rows only
        
[hyomee] [2023-09-02 23:13:42.481] [http-nio-8080-exec-6] INFO  c.h.s.tour.demo.entity.DemoEntity 
- PostLoad : DemoEntity(id=2, title=24동안 하는일, content=24시간 동안 장을 잔다다, name=홍퍼플플)
```

***

#### 1-1-3 수정&#x20;

demoRepository.save(demoEntity) 가 실행이 되면 다음과 같은 순서로 실행이 됩니다,

1. Select
2. PostLoad : id = 52 , 이전 값 : DemoEntity(id=52, title=24동안 하는일, content=글을 수정 합니다., name=홍퍼)
3. PrePersist : id = 52,  이후 값 : DemoEntity(id=52, title=제목도 수정, content=모두 수정 합니다., name=이름)
4. save :
5. PostUpdate : DemoEntity(id=52, title=제목도 수정정, content=모두 수정 합니다., name=이름)

```bash
curl --location 'localhost:8080/jpademo' \
--header 'Content-Type: application/json' \
--data '{
    "id" : 52,
    "name": "이름",
    "title": "제목도 수정정",
    "content": "모두 수정 합니다."
}'
============================================================================================
[Hibernate] 
    select
        d1_0.id,
        d1_0.content,
        d1_0.name,
        d1_0.title 
    from
        tb_demo d1_0 
    where
        d1_0.id=?
[hyomee] [2023-09-02 23:41:41.916] [http-nio-8080-exec-10] INFO  c.h.s.tour.demo.entity.DemoEntity 
- PostLoad : DemoEntity(id=52, title=24동안 하는일, content=글을 수정 합니다., name=홍퍼)

[hyomee] [2023-09-02 23:41:41.917] [http-nio-8080-exec-10] INFO  c.h.s.tour.demo.entity.DemoEntity 
- PreUpdate : DemoEntity(id=52, title=제목도 수정정, content=모두 수정 합니다., name=이름)

[Hibernate] 
    /* update
        for com.hyomee.service.tour.demo.entity.DemoEntity */update tb_demo 
    set
        content=?,
        name=?,
        title=? 
    where
        id=?
[hyomee] [2023-09-02 23:41:41.919] [http-nio-8080-exec-10] INFO  c.h.s.tour.demo.entity.DemoEntity 
- PostUpdate : DemoEntity(id=52, title=제목도 수정정, content=모두 수정 합니다., name=이름)
```

수정 쿼리가 호출되기 전에 PostLoad를 통해서 Select 쿼리가 발생되는 것을 확인할 수 있습니다.

***

#### 1-1-4. 삭제&#x20;

demoService.deleteId(id)시 다음과 같은 순서로 실행이 됩니다.

1. Select&#x20;
2. PostLoad : DemoEntity(id=52, title=제목도 수정, content=모두 수정 합니다., name=이름)
3. PreRemove : DemoEntity(id=52, title=제목도 수정, content=모두 수정 합니다., name=이름)
4. Delete
5. PostRemove : DemoEntity(id=52, title=제목도 수정, content=모두 수정 합니다., name=이름)

```bash
curl --location --request DELETE 'localhost:8080/jpademo/52' \
--header 'Content-Type: application/json

=============================================================================================
[Hibernate] 
    select
        d1_0.id,
        d1_0.content,
        d1_0.name,
        d1_0.title 
    from
        tb_demo d1_0 
    where
        d1_0.id=?
[hyomee] [2023-09-03 00:00:23.794] [http-nio-8080-exec-3] INFO  c.h.s.tour.demo.entity.DemoEntity 
- PostLoad : DemoEntity(id=52, title=제목도 수정, content=모두 수정 합니다., name=이름)

[hyomee] [2023-09-03 00:00:23.815] [http-nio-8080-exec-3] INFO  c.h.s.tour.demo.entity.DemoEntity 
- PreRemove : DemoEntity(id=52, title=제목도 수정, content=모두 수정 합니다., name=이름)

[Hibernate] 
    /* delete for com.hyomee.service.tour.demo.entity.DemoEntity */delete 
    from
        tb_demo 
    where
        id=?
[hyomee] [2023-09-03 00:00:23.888] [http-nio-8080-exec-3] INFO  c.h.s.tour.demo.entity.DemoEntity 
- PostRemove : DemoEntity(id=52, title=제목도 수정, content=모두 수정 합니다., name=이름)
```

삭제 쿼리가 호출되기 전에 PostLoad를 통해서 Select 쿼리가 발생되는 것을 확인할 수 있습니다.

***

### 1-2. EntityListener 사용

Entity에 직접 애노테이션은 모든 Entity에 필요에 따라서 코드를 작업하는 것보다는 일괄 접근 방법으로 하는 것이 효율적인 방법으로 이때 사용하는 방법이 EntityListener 를 사용하는 방법입니다. 개발 순서는 다음과 같습니다,

1. Entity의 애노테이션 @EntityListeners에 등록할 신규 클래스를 작성 합니다.
2. Entity에 @EntityListeners으로 산규 클래스를 등록 합니다.

#### 1-2-1. @EntityListeners에 등록할 신규 클래스

Entity Life Cycle에 따른 Entity 객체의 값을  출력을 하는 Listener객체로 로그는 다음 형식으로 출력이 됩니다.

```bash
com.hyomee.service.tour.demo.entity.DemoEventEntity > 
PostPersist : DemoEventEntity(super=AuditVO(createDateTime=2023-09-03T21:50:35.736016500, 
createUserid=Test, updateDataTime=2023-09-03T21:50:35.736016500, updateUserId=Test), 
id=2, title=24동안 하는일12, content=24시간 동안 장을 잔다다, name=홍길동동)
```

```java
public class EntityEventCallListener {

  @PrePersist
  public void prePersistEntity(Object obj) {
    printLog("PrePersist", obj);
  }

  @PostPersist
  public void postPersisttEntity(Object obj) {
    printLog("PostPersist", obj);
  }

  @PreRemove
  public void preRemoveEntity(Object obj) {
    printLog("PreRemove", obj);
  }

  @PostRemove
  public void postRemoveEntity(Object obj) {
    printLog("PostRemove", obj);
  }

  @PreUpdate
  public void preUpdateEntity(Object obj) {
    printLog("PreUpdate", obj);
  }

  @PostUpdate
  public void postUpdateEntity(Object obj) {
    printLog("PostUpdate", obj);
  }

  @PostLoad
  public void postLoadEntity(Object obj) {
    printLog("PostLoad", obj);
  }


  private void printLog(String methodName, Object obj) {
    log.info( obj.getClass().getName() + " > "
            + methodName + " : "
            + obj);
  }
```

***

#### 1-2-2. Entity에 @EntityListeners 적용

DemoEventEntity 클래스에 신규 작성한 EntityEventCallListener 클래스를 @EntityListeners 애노테이션에 등록을 합니다.

```java
@Table(name = "TB_EVENT_DEMO")
@EntityListeners(EntityEventCallListener.class)
@Entity 
public class DemoEventEntity extends AuditVO {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private int id;
    private String title;
    private String content;
    private String name;
}
```

***

#### 1-2-3. 결과&#x20;

간단한 조회 RestAPI를 호출하면 아래 로그처럼 > PostLoad : DemoEventEntity.... 를 확인할 수 있습니다.

```bash
curl --location 'localhost:8080/demoevent/홍길동' \
--header 'Content-Type: application/json'
```

```
[Hibernate] 
    /* <criteria> */ select
        d1_0.id,
        d1_0.content,
        d1_0.create_date_time,
        d1_0.create_userid,
        d1_0.name,
        d1_0.title,
        d1_0.update_data_time,
        d1_0.update_user_id 
    from
        tb_event_demo d1_0 
    where
        d1_0.name=?
[hyomee] [2023-09-03 22:11:33.420] [http-nio-8080-exec-6] INFO  c.h.j.event.EntityEventCallListener 
- com.hyomee.service.tour.demo.entity.DemoEventEntity 
> PostLoad : DemoEventEntity(super=AuditVO(createDateTime=2023-09-03T21:50:35.736017, 
createUserid=Test, updateDataTime=2023-09-03T21:50:35.736017, updateUserId=Test), 
id=2,  .... )
```

***

필요에 따라서 Entity Life Cycle Event를 Entity에 직접 애노테이션 방법, EntityListener 방법을 사용하면 됩니다. 어떤 방법을 적용할지는 공통적인 기능을 추가하는 것이라면 EntityListener  방법으로 하고 개별적으로 처리하는 기능이라면 Entity에 직접 애노테이션 방법으로 하는 것이 효율적입니다.
