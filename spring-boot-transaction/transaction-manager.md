# Transaction Manager



## 1. 트랜잭션 격리 수준(Transaction Isolation Level

* **트랜잭션**은  여러 데이터베이스 작업을 하나의 작업 단위로 묶어, 모든 작업이 성공적으로 완료되거나 실패할 경우 원래 상태로 복구하여 데이터의 정합성을 보장한 것으로 데이터베이스의 ACID(원자성, 일관성, 격리성, 지속성) 속성을 만족해야 한다.&#x20;
* **트랜잭션 격리 수준** 이란 테이터 베이스 시스템에서 동시에 여러 트랜잭션이 실행될 때, 트랜잭션 간에 데이터를 어떻게 고립시킬지를 결정하는 설정 으로 동시에 여러 트랜잭션이 처리될 때 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있도록 허용할지 말지를 결정하는 것이다.

### 1-1.  종류

* **READ UNCOMMITTED : Propagation.REQUIRED** \
  **-** 다른 트랜잭션이 커밋하지 않은 데이터도 읽을 수 있어, '더티 리드(Dirty Read)'가 발생 할 수 있다.&#x20;
* **READ COMMITTED : Propagation.SUPPORTS** \
  \- 커밋된 데이터만 읽을 수 있어, '논리적인 읽기 오류(Non-repeatable Read)'를 방지한다.
* **REPEATABLE READ** \
  \- 트랜잭션이 시작된 후에 읽은 데이터를 트랜잭션이 종료될 때까지 반복해서 읽을 수 있다.
* **SERIALIZABLE** \
  \- 가장 엄격한 격리 수준으로, 트랜잭션들이 순차적으로 실행되는 것처럼 처리하여 데이터의 일관성을 최대한 보장한다.



<details>

<summary>duyd</summary>

* 원자성(Atomicity) : 트랜잭션이 어느 시점에서든 실패하면 데이터베이스에 적용된 모든 변경 사항이 롤백되고 데이터베이스는 원래 상태로 돌아간다&#x20;

일관성(Consistency) : 데이터베이스는 트랜잭션이 실행되기 전과 후에 모두 유효한 상태이어야 한다. (트랜잭션이 기본 키 또는 외래 키 제약 조건과 같은 데이터베이스 제약 조건을 위반하는 경우 트랜잭션은 롤백) 격리성(Isolation) : 다중 사용자 환경에서는 트랜잭션이 동시에 실행되므로 각 트랜잭션이 격리되어 다른 트랜잭션의 결과에 영향을 미치지 않아야 한다. 영속성(Durability) : 트랜잭션이 커밋되면 데이터베이스에 대한 변경 내용이 영구적으로 유지되어야 하며 시스템 충돌이나 정전과 같은 후속 장애에도 영향을 받지 않아야 한다.

</details>

### 1-2.  격리 수준 레벨에 따른 이슈

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

### 1-3.  격리 수준 레벨에 따른 고립 정도 및 성능

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>



참고사이트&#x20;

[PostgreSQL triggers and isolation levels - Vlad ](https://vladmihalcea.com/postgresql-triggers-isolation-levels/)[Mihalcea](https://vladmihalcea.com/postgresql-triggers-isolation-levels/)

[\[Database\] ](https://velog.io/@sukyeongs/Database-Transaction-Isolation-Level)[트랜잭션 격리 수준](https://velog.io/@sukyeongs/Database-Transaction-Isolation-Level)[(Transaction Isolation Level) (velog.io)](https://velog.io/@sukyeongs/Database-Transaction-Isolation-Level)
