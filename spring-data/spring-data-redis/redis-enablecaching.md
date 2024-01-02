---
description: '@EnableCaching를 사용하여 Spring 에서 Cache 관리'
---

# Redis EnableCaching

적용 Spring Boot Version

* spring-boot-starter-data-jpa:3.0.4
* spring-boot-starter-data-redis:3.1.3

## 1. 의존성 추가 (gradle 기준)

```gradle
runtimeOnly 'org.mariadb.jdbc:mariadb-java-client:2.7.6'

implementation 'org.springframework.boot:spring-boot-starter-data-redis:3.1.3'
implementation 'org.springframework.boot:spring-boot-starter-data-jpa:3.0.4'

implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'

annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"
annotationProcessor 'jakarta.annotation:jakarta.annotation-api:2.1.1'
annotationProcessor 'jakarta.persistence:jakarta.persistence-api:3.1.0'
```

## 2. Application.yml 설정

아래 설정과 같이 Redis 관련 설정을 한다.

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password: "비밀번호"
      lettuce:
        pool:
          enabled: true
```

## 3.  RedisCacheManager 속성 설정

Spring Boot에서 Redis Cache 기본 속성을 사용 할 수 있지만 재정의 하여 사용하고자 한다. 다음은 RedisCacheManager의 속성 입니다.

* **spring.cache.type**: 적용할  Cache 설정, ex) redis &#x20;
* **spring.cache.cache-names**: Cache 이름  설정
* **Redis 관련 설정 ( spring.cache.redis.\* )**
  * **spring.cache.redis.cache-null-values**: null 허용 여부, true null 허용
  * **spring.cache.redis.time-to-live**: 만료  시간.
    * 600000 이면 만료 시간 : 10분&#x20;
  * **spring.cache.redis.use-key-prefix**: true인 경우 Redis에 쓰는 동안 키 접두사가 사용
  * **spring.cache.redis.key-prefix:** 기본적으로 두 개의 개별 캐시가 동일한 키를 사용할 때 키가 겹치는 것을 방지하기 위해 키 접두사가 추가.

## 4.  RedisCacheManager 재정의

Redis구성을 Application에서 제어하기 위해 RedisStandaloneConfiguration, LettuceConnectionFactory을 사용하여 Redis연동을 위한 Bean 등록한다.

```java
@Configuration
public class RedisConfig {


    @Value("${spring.data.redis.host}")
    private String host;

    @Value("${spring.data.redis.port}")
    private int port;

    @Value("${spring.data.redis.password}")
    private String password;


    /**
     * Redis 연결
     * @return
     */
    // LettuceConnectionFactory Bean 등록 
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration redisConfiguration = new RedisStandaloneConfiguration();
        redisConfiguration.setHostName(host);
        redisConfiguration.setPort(port);
        redisConfiguration.setPassword(password);
        LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory(redisConfiguration);

        return lettuceConnectionFactory;
    }

    // RedisTemplate Bean 등록 
    @Primary
    @Bean
    public RedisTemplate<String,?> redisTemplate(){
        RedisTemplate<String, ?> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return redisTemplate;
    }
    
    // RedisCacheManager Bean 등록 
    @Primary
    @Bean
    public CacheManager cacheManager(){
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
                .entryTtl(Duration.ofSeconds(3600));
        RedisCacheManager cacheManager = RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory(redisConnectionFactory())
                .cacheDefaults(redisCacheConfiguration).build();

        return cacheManager;
    }
}
```

## 5. Caching 활성화@ -  EnableCaching

추상화된 Cache를 사용하기 위해서는 @EnableCaching를 활성화 하여야.하는데 위치는@Configuration 또는 @SpringBootApplication에 추가 한다.

```java
@EnableJpaAuditing
@EnableCaching
@SpringBootApplication()
public class JpaServiceApplication  {
    public static void main(String[] args) {
        SpringApplication.run(JpaServiceApplication.class, args);
    }
}
```

## 6. Cache Annotation

### 6-1. @Cacheable

Cache에서 조회 결과가 있으면 메서드 호출을 하지 않는다, 즉 Cache된 결과만 반환&#x20;

* cacheNames: Cache 이름
* value: Cache 이름의별칭
* condition: 조건부 캐싱을 수행하는 Spring SpEL 표현식
* key: 키를 동적으로 계산하는 SpEL.
* keyGenerator: 사용자 정의를 위한 Bean 이름
* unless : SpEL을 사용하여 메서드 캐싱을 거부&#x20;
* sync: 여러 스레드가 동일한 키에 대한 값을 로드하려고 할 때 메소드 호출을 동기화하는 데 사용
* key, condition, unless 에서 사용할 수 있는 SpEL
  * \#result: 결과.반환 참조
  * \#root.method: 메서드 참조 ...&#x20;

```java
@Cacheable(value= "allCodeCache", unless= "#result.size() == 0")	
public List<Code> getAllCode(){
  ------
} 
```

* 결과가 O인 것은 반환하지 않는다 .

<pre class="language-java"><code class="lang-java"><strong>@Cacheable(value= "CodeCache", key= "#codeId")	
</strong>public List&#x3C;Code> getCodeByCodeId(String codeId){
  ------
} 
</code></pre>

* CodeCache에서 codeId가 같은 것을 반환 한다.

### 6-2. @CachePut

Cache에 쓰기 작업을 한다.

```java
@CachePut(value= "CodeCache", key= "#code.codeId")	
public Code  addCodeByCodeId(Code code){
  ------
} 
```

### 6-3. @CacheEvict

Cache에 삭제 작업을 한다.

```java
@CacheEvict(value= "CodeCache")	
public void deleteCodeCache(String codeId){
  ------
} 
```

### 6-4. @Caching

Cache 작업을 그룹으로 실행 한다.

```java
@Caching(
   put= { @CachePut(value= "CodeCache", key= "#code.codeId") },
   evict= { @CacheEvict(value= "CodeCache", allEntries= true ) }
)
public Article updateCode(Code code) {
   ------
} 


@Caching(
   evict= { 
	@CacheEvict(value= "CodeCache", key= "#codeId"),
	@CacheEvict(value= "allCodeCache", allEntries= true)
   }
)
public void deleteCode(String codeId) {
   ------
} 
```

