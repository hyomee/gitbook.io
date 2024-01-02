# Cache 일반 개념

Cache는 주 사용하는 데이터나 값을 임시 장소에 복사를 하여 사용하므로 빠른 성능을 제공하기 위해서 사용하는 것이다. 즉 속도가 빠른 장치와 느린 장치에서 속도 차이에 따른 병목 현상을 줄이기 위한 메모리를 말한다.

## 1.    Cache 계층 구조

* Registers : \~ 1KB, 1\~2 Cycle
* L1 : \~ 100KB, 2\~3 Cycle, CPU 내부에 존재
* L2 : \~ 500KB, 3\~5 Cycle, CPU와 RAM 사이에 존재
* L3 : \~ 10\~15MB, 30\~50 Cycle, 보통 메인  보드에 존재&#x20;
* Main Memory : \~10GB, 50\~200 Cycles
* SDD/HDD : \~ 1000GB, 50000 Cycles

## 2. Cache는 언제 사용해야 하는가

* Application 개발 시 동일한 데이터에 대해서 RestAPI 또는 DataBase Query로 시간이 걸리는 경우
* 결과가 같은 것은 반복/중복 하는 경우

### 2-1. Cache Hit&#x20;

원하는 데이터가 Cache에 존재할 경우 해당 데이터를 반환하는 것

### 2-2. Cache Miss

데이터가 Cache에 존재하지 않을 경우 DBMS 또는 서버에 요청을 하여 Cache에 적재 하는 것으로 동기화를 어떻게 할 것 인가에 대한 전략이 필요하다.

1.  **Cold miss**

    해당 메모리 주소를 처음 불러서 나는 미스
2.  **Conflict miss**

    캐시 메모리에 A와 B 데이터를 저장해야 하는데, A와 B가 같은 캐시 메모리 주소에 할당되어 있어서 나는 미스 (direct mapped cache에서 많이 발생)
3.  **Capacity miss**

    캐시 메모리의 공간이 부족해서 나는 미스 (Conflict는 주소 할당 문제, Capacity는 공간 문제)

## 3. Cache 전략

### 3-1. Cache-Aside

Application에서는 Cache를 먼저 조회 하는 전략을 Cache 데이터가 존재(Cache Hit)하면 조회한 데이터를 반환 하고 존재 하지 않으면 (Cache Miss) 원천 데이터( RestAPI 또는 DB Query )를 조회하여 Cache에 적재 한 후 데이터를 반환하는 전략이다.

* 읽기가 많은 곳에 적합
  * 단건 호출 빈도가 높으면 적합하지 않지만 데이터를 미리 적재하는 (Cache Warming) 전략과 병행 하여 사용할 수 있다.
  * Cache 는 사이즈가 한정적이므로 expire(만료 시간)에 대한 정책이 필요하다. → 데이터 일괄성 문제&#x20;
* 원천 데이터( RestAPI 또는 DB Query )를 조회  시 오류에 대한 장애 전파 필요
* Cache Miss 발생 시 Cache에 대한 Update는 Application에서 담당한다.

### 3-2. Read-Through Cache

기본적으로 Cache-Aside전략과 동일하지만 Cache Miss 발생 시 Cache에 대한 Update는 Cache가 담당하는 전략이다.

* Cache 와 DB Table의 모양이 같은 경우 적합하다.
* 미리 Cache에 적재 하는 전략 필요 ( 배치, 스케줄 등 )

### 3-3. Write-Through Cache

Data write를 하는 전략으로  Cache가 주체가 되는 것으로 Write 발생 시 Cache에 데이터를 적재하고 Cache가 DB에 데이터를 적재 합니다.

* Cache 가 DB에 적재를 하므로 쓰기  지연이 발생 할 수 있다.

### 3-4. Write-Around

Data Write 발생 시 DB에 Write와 Catch Wtrie가 분리 되어 있는 전략으로 DB Write가 발생하면 DB에 적재 하고 Application에서 조회 시점에  Cache Miss 발생 하면 DB를 조회 하여 Cache가 Cache를 Update 후 데이터를 반환 해 주는 전략이다.

### 3-5. Write-Back(Write Behind)

Cache에 일관 적재 하는 전략으로 배치나 스케줄을 통해 데이터를 일괄 적재하면 Cache에 반영하고 Cache가 DB에 적재 하는 전략이다.

* 데이터를 모았다가 적재하므로 적재 주기 사이에 Cache가 문제가 발생할 경우 데이터 유실에 따른 문제가 발생할 수 있다.

## 4. Cache 종류

### 4-1. Local Cache

Cache를 사용하는 장비에만 적용되는 Cache로 내부 자원을 사용하여 속도가 빠르나 다른 서버와 데이터 공유가 어렵다.

### 4-2. Global Cache

Cache Server에서 관리하는 Cache로 분산 환경에 Cache를 조회 하므로 Local Cache에 비해서 상대적으로 속도가 느리나 데이터 공유는 Local Cache에 비해서 상대적으로 쉽다

## 5. Cache에 무엇을저장 해야 하나

Cache에 보관되는 정보는 어떤 기준으로 저장을 해야 하는 것 인지에 대한 것은 어찌 보면 단순하다. 자주 변경이 되는 정보를 Cache에서 관리하면 읽기와 쓰기가 자주 발생 할 것이고 동기화에 따른 여러 이슈가 나오게 된다. 즉 자주 사용이 되면 변경되지 않는 데이터를 Cache에 저장 하면 된다. 공통 코드, 정책성 데이터 등으로 Cache는 언제나 휘발성이라는 전제  하에 데이터가 Cache에 데이터가 없어져도 다른 방법으로 쉽게 복구 할 수 있는 데이터를 Cache에 보관한다고 생각하면 된다.

* 중요한 정보, 민감 정보는 보관하지 않는다.
* 자주 변경이 되지 않고 자주 사용하는 정보를 기준으로 Cache 대상을 선정한다.
* Cache 동기화를 위해서 Cache 전략을 혼합해서 사용한다. ( Cache-Aside + Write-Around)
* 실시간으로 데이터를 저장 하는 정보는 피한다. 만약  Cache에  저장을 해야 한다면 수집 주와 저장 주기를 가지도록 설계한다.

## 6. Cache는 언제 삭제 해야 하나

Cache의 삭제는 메모리 제한에 도달할 때 발생되는 것으로 상황에 따라 결정을 해야 한다.&#x20;





[Redis에 대한 Cache는 여기를 참고](https://hyomee.gitbook.io/develop/db/redis)해 주세요.
