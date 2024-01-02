# Redis CacheManager

SpringBoot : v3.1.2 &#x20;

참고 : [Spring-Data-Redis](https://docs.spring.io/spring-data-redis/docs/current/reference/html/)

## 1. 종속성 추가

Spring Data Redis를 사용하기 위해서는 spring-boot-starter-data-redis를 추가해주어야 합니다.

```
implementation 'org.springframework.boot:spring-boot-starter-data-redis:3.1.3'
```

## 2. Application.yml 추가&#x20;

Redis에 대한 환경 설정입니다.&#x20;

```yaml
spring:  
  data:
    redis:
      host: localhost
      port: 6379
      password: "password" 
```

## 3. Redis 환경 구성

Redis OM Spring을 사용하면 Spring Data Redis(SDR) 프레임워크를 기반으로 구축된  리포지토리 및 사용자 지정 개체 매핑 추상화를 제공하여 코드를 작성할 수 있습니다.

본 문서에서는 @Configuration을 사용해서 자동 구성 하는 방법에 대해서 설명을 합니다.

### 3-1. 자동 구성 파일 작성

Application.yml에 작성한 환경 정보를 읽어와 변수에 설정 합니다.

```java
@Configuration
public class RedisConfig {

    @Value("${spring.data.redis.host}")
    private String host;

    @Value("${spring.data.redis.port}")
    private int port;

    @Value("${spring.data.redis.password}")
    private String password;
}
```

### 3-2. **Lettuce 커넥터 구성**

Lettuce는 Spring Data Redis에서 패키지를 통해 지원하는 Netty 기반 오픈 소스 커넥터로  LettuceConnectionFactory를 작성 합니다.

단일 노드 Redis 설치에 연결하기 위해 RedisStandaloneConfiguration을 사용해 접속 정보를 설정하고 Lettuce연결 포인트를 작성 합니다.

```java
@Bean
public RedisConnectionFactory redisConnectionFactory() {
    RedisStandaloneConfiguration redisConfiguration 
            = new RedisStandaloneConfiguration();
    redisConfiguration.setHostName(host);
    redisConfiguration.setPort(port);
    redisConfiguration.setPassword(password);
    
    LettuceConnectionFactory lettuceConnectionFactory 
            = new LettuceConnectionFactory(redisConfiguration);

    return lettuceConnectionFactory;
}
```

### 3-3 RedisTemplate 재정의&#x20;

&#x20;Redis의 데이터가 JSON 형식으로 데이터를 저장하도록 하기 위해서  Key, HashValue. Value에 대한 직렬화를 재정의 합니다.

```java
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
```

### 3-4. CacheManager 재정의

Redis CacheManager는 스프링 프레임워크에서 제공하는 캐시 추상화 계층 중 하나로 Redis CacheManager는 Redis를 사용하여 캐시를 구현합니다. &#x20;

key/value에 대해서 직렬화 방법을 설정 하고 RedisCacheManager를 생성 합니다다. @Primary CacheManager가 이름 중복으로 우선 설정하라는 의미입니다. 또는 @Bean(name = "dataCacheManager")으로 선언하고 사용해야 합니다.

```java
@Primary
@Bean
public CacheManager cacheManager(){
        RedisCacheConfiguration redisCacheConfiguration 
            = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
                .entryTtl(Duration.ofSeconds(3600));
                
        RedisCacheManager cacheManager 
            = RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory(redisConnectionFactory())
                .cacheDefaults(redisCacheConfiguration).build();

return cacheManager;
}
```

## 4. 공통 컴포넌트

비지니스 개발시 Redis에 put,get, clear등을 하기 위해서 개별적으로 코드를 작성하기 보다는 공통 컴포넌트를 만들어서 재사용하는 것이 효율적입니다.

### 4-1. CacheManager 컴포넌트

<pre class="language-java"><code class="lang-java"><strong>@RequiredArgsConstructor
</strong>@Component
public class RedisCacheComponent&#x3C;T> {

  private final CacheManager cacheManager;
  
  public T get(String cacheName, String key){
    return (T) getCacheName(cacheName).get(key);
  }

  public T get(String cacheName, String key, Class&#x3C;T> clszz) {
    return  getCacheName(cacheName).get(key, clszz);
  }

  public void put(String cacheName, String key, Object value){
    getCacheName(cacheName).put(key, value);
  }

  public void putIfAbsent(String cacheName, String key, Object value){
    getCacheName(cacheName).put(key, value);
  }

  public void removeValues(String cacheName, String key){
    getCacheName(cacheName).evictIfPresent(key);
  }

  public void clear(String cacheName){
    getCacheName(cacheName).invalidate();
  }

  private Cache getCacheName(String cacheName) {
    return cacheManager.getCache(cacheName);
  }

}

</code></pre>

### 4-2. RedisTemplate 컴포넌트

```java
@Component
@RequiredArgsConstructor
public class RedisTemplateComponet {
 
    private final RedisTemplate redisTemplate;

    public String getValues(String key){
        //opsForValue : Strings를 쉽게 Serialize / Deserialize 해주는 Interface
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        return values.get(key);
    }

    public void setValues(String key, String value){
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set(key,value);
    }

    public void setSets(String key,String... values){
        redisTemplate.opsForSet().add(key,values);
    }

    public Set getSets(String key){
        return redisTemplate.opsForSet().members(key);
    }
}

```

#### 4-2-1. RedisTemplate 사용 시 메서드별 반환 타입 및 Redis 테이터 타입&#x20;

|      메소드명     |     반환 오퍼레이션    | 관련 Redis 자료구조 |
| :-----------: | :-------------: | :-----------: |
| opsForValue() | ValueOperations |     String    |
|  opsForList() |  ListOperations |      List     |
|  opsForSet()  |  ListOperations |      List     |
|  opsForList() |  SetOperations  |      Set      |
|  opsForZSet() |  ZSetOperations |   Sorted Set  |
|  opsForHash() |  HashOperations |      Hash     |

#### 4-2-2. 직접 주입을 통해서 코드  작성 방법&#x20;

```java
// 특정 오퍼레이션 직접 주입
@Resource(name ="redisTemplate")
private ValueOperations<String, Object> valueOps
```

## 5. 예제

```java
@Data
public class RedisDTO {
  private String prodCd;
  private String prodNm;
}

@RequiredArgsConstructor
@Service
public class RedisTestService {

  private final RedisCacheComponent redisCacheComponent;

  public RedisDTO put(String cacheName, RedisDTO redisDTO) {
    redisCacheComponent.put(cacheName, redisDTO.getProdCd(), redisDTO);

    return (RedisDTO) redisCacheComponent.get(cacheName, redisDTO.getProdCd(), RedisDTO.class  );
  }
}

@RequiredArgsConstructor
@RestController
@RequestMapping("/redis")
public class RedisTestController {

  private final RedisTestService redisTestService;

  @PutMapping ("/put/{cacheName}")
  public ResponseEntity put( @PathVariable String cacheName,
                             @RequestBody RedisDTO redisDTO)   {
    return ResponseUtils.completed(redisTestService.put(cacheName, redisDTO));
  }

}

```

Curl&#x20;

````
```powershell
curl --location --request PUT 'http://localhost:8080/redis/put/PROD_CD' \
--header 'Content-Type: application/json; charset=UTF-8' \
--data '{
    "prodCd": "LPZ000004",
    "prodNm": "일반요금제4"
}'
```
````

Redis에서 확인이 가능하다.

```
127.0.0.1:6379> keys *
1) "PROD_CD::LPZ000004"
2) "PROD_CD::LPZ000003"
3) "CUST_CH::LZC000001"
4) "PROD_CD::LPZ000001"
5) "PROD_CD::LPZ000002"
127.0.0.1:6379>

```
