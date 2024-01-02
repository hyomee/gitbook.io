# Spring Cache

Cache는 Application의 성능을 높이기 위해 사용하는 것으로 버퍼(buffer)와는 차이가 있다. Buffer과 Cache차이점은 [위키피아에서  ](https://en.wikipedia.org/wiki/Cache\_\(computing\)#The\_difference\_between\_buffer\_and\_cache)확인할 수 있다.

Spring Cache는  JDK 기반 캐시, Gemfire 캐시, Caffeine 및 JSR-107 호환 캐시(예: Ehcache 3.x)와 같은 추상화의 몇 가지 구현을 제공하고,  다른 캐시 저장소 및 공급자 연결을.위해서는  CacheManager및 Cache구현하여야 한다. \
(CacheManager   :: org.springframework.cache.supportAbstractCacheManager)&#x20;

Cache 적용시 주의할 사항은 멀티 프로세스환경에서 사용하는 경우 Cache의 데이터가 변경된 경우 전파하는 메커니즘을 확인해야 한다.



참고 : [Cache 일반 개념](https://hyomee.gitbook.io/develop/undefined-1/cache)

참고 : [Spring-Data-Cache](https://hyomee.gitbook.io/develop/spring-data/spring-data-redis)

참고 : [Redis](https://hyomee.gitbook.io/develop/db/redis)

