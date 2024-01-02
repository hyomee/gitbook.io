# Redis

Redis는 대표적인 Key/Value DB 중에 하나 이다.

설치 참고 : [Redis 설치](https://hyomee.gitbook.io/solution/undefined-1/redis)

학습 참고 :  [Redis 사이트](https://redis.io/)

개발자   참고. : [개발자 사이트](https://developer.redis.com/develop/java)

## 1. Key/Value DB의 논리적 구조

* Table : 하나의 DB에서 데이블을 저장하는 논리적 구조 ( DB : TABLE 개념 )
* Data Sets : 테이블을 구성하는 논리적 구성 ( DB : row 개념 )
  * 하나의 Key 와 한 개 이상의 Field/Element로 구성&#x20;
* Key : 키가 되는 값 ( DB : Primary Key )
* Values : 키에 대한 구체적인 데이터 값으로 한 개 이상의 Field/Element구성.

## 2. Redis 특징

1. 성능 : 모든 Redis 데이터는 메모리에 저장되어 대기 시간을 낮추고 처리량을 높인다.
2. 유연한 데이터 구조 : Redis의 데이터는 String, List, Set, Hash, Sorted Set, Bitmap, JSON 등 다양한 데이터 타입을 지원한다.
3. 개발 용이성 : 단순한 명령 구조로 데이터의 저장, 조회 등이 가능하며.   Java, Python, C, C++, C#, JavaScript, PHP, Node.js, Ruby 등을 비롯한 다수의 언어를 지원한다.
4. 영속성 : Redis는 영속성을 보장하기 위해 데이터를 디스크에 저장할 수 있다.&#x20;
5. &#x20;싱글 스레드 방식 : Redis는 싱글 스레드 방식을 사용하여 한 번에 하나의 명령어만을 처리한다. 따라서 연산을 원자적으로 처리하여 Race Condition(경쟁 상태)가 거의 발생하지 않는다.

## 3. Redis 데이터 구조

<figure><img src="../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

* Binary-safe 문자열 : 문자, Binary 데이터를 저장
  * ![](<../.gitbook/assets/image (74).png>)
* List : 하나의 key에 여러개의 배열 저장
  * ![](<../.gitbook/assets/image (75).png>)
* Set : 순서가 보장되지 않으며 중복이 허용되지 않는 타입
  * ![](<../.gitbook/assets/image (76).png>)
* Sorted set : 셋 데이터와 동일한 특징을 가지나 저장된 요소에 가중치를 부여하여 작은 값부터 큰 값(오름차순) 정렬, 순서가 변경될 수 있
  * ![](<../.gitbook/assets/image (77).png>)
* Hash : 하나의 Key에 여러개의 Fields와 Value 구성&#x20;
  * ![](<../.gitbook/assets/image (78).png>)

## 4. Redis Cache Strategy

### 4-1. Read Cache Strategy

Cache 데이터를 읽기 위한 전략으로 다음과 같은 전략이 있다

#### 4-1-1. **Look Aside Strategy**

Cache 전략 중  [Cache-Aside](https://hyomee.gitbook.io/develop/undefined-1/cache)이며 데이터를 찾을때 Cache에 있는 정보를 먼저 찾고 없으면 원본 데이터 (  DB  or RestAPI ) 를 조회하는 패턴이다,

* Cache와 DB가 분리 되어 있어서 필요한 Data만 데이터만 Cache로 구성 할 수 있음
* Redis 장애에 따른 원본 조회 기능 필요
* Redis 장애 또는 Redis에 데이터가 없을 경우 원본 조회에 따른 부하 발생&#x20;
* Redis를 먼저 읽으므로 초기 적재 하는 작업이 때에 따라서 필요함 (Cache Warming)&#x20;

#### 4-1-2. **Read Through Strategy**

Cache 전략 중[ Read-Through Cache](https://hyomee.gitbook.io/develop/undefined-1/cache)전략으로 데이터 동기화를 Cache에 위임 하는 전략이다.

* 조회를 Cache 만 의존하는 전략으로 Cache에 장애가 생겼을 때 문제가 발생 할 수 있음
  * Replication 또는 Cluster로 구성을 하여 가용성을 높여야 함&#x20;
* Cache에서 DB 동기화를 하여 동기화에 대한 문제를 Application에서 자유로움&#x20;

### 4-2. Write Cache Strategy

Cache 데이터를 쓰기 위한 전략으로 다음과 같은 전략이 있다

**4-2-1. Write Back Strategy**

Cache 전략 중 [Write-Back(Write Behind)](https://hyomee.gitbook.io/develop/undefined-1/cache)전략으로 Cache에 있는 정보가 배치 또는 스케줄에 의해서 동기화 처리되는 전략이다.

* 캐시에 모았다가 DB에 저장 하므로 DB쓰기에 대한 부하를 줄일 수 있지만 캐시 장애시 데이터 유실이 될 수 있지만 DB 장애가 있어도 Cache에서 읽어서 서비스를 제공 할 수 있다.

**4-2-2. Write-Through Strategy**

Cache 전략 중 [Write-Through Cache](https://hyomee.gitbook.io/develop/undefined-1/cache)전략으로 Cache에 적재 하고 DB에 바로 저장하는 전략이다.

* Cache + DB로 가장 최신 상태를 유지 할 수 있으며 일관성을 유지 할 수 있다,
* 매 요청마다 생성  및  수정 시 Cache와 DB가 같이 변경이 발생 하므로 서비스   성능 이슈가 발생 할 수 있다.

**4-2-3. Write-Around Strategy**

Cache 전략 중[ Write-Around](https://hyomee.gitbook.io/develop/undefined-1/cache)[ Cache](#user-content-fn-1)[^1]전략으로 DB에 저장하는 전략

* &#x20;모든 데이터는 DB에 저장 하고 Cache miss가 발생하는 경우에만 DB와 캐시에 데이터를 저장하므로 불일치 발생 할 수 있으므로 Cache의 expire를 짧게 조정하여 극복 할 불일치를 조정 할 수 있다,



Cache애 대한 [자세한 사항은 여기를 참고](../design/cache.md)하세요.







####

[^1]: 
