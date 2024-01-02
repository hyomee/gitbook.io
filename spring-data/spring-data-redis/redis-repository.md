# Redis Repository

Redis 연동 방법 중 org.springframework.data.repository를 사용하면  findAll, findById, save 등을 사용할 수 있는데  Redis의 Hash 자료구조 한정으로 데이터 객체에 @RedisHash()을 사용해야 한다.

**Redis를 사용하기 위한 환경 구성은** [**Redis CacheManager**](redis-cachemanager.md) **에서 작성한 예제를 참조하여 작성을 해야 한다, ( @Configuration )**

Spring-Data에서 제공하는 repository사용하기 위해서는 다음과 같은 기준이 있습니다.

1. **데이터를 저장할 객체 생성**&#x20;
   * @RedisHash  어노테이션(Annotation)을 사용하는 경우
     * value 사용 :  Redis keyspace 는 작성한.명으로 생성된다.\
       :  @RedisHash(value = "REDIS\_HASH\_DOC") = "REDIS\_HASH\_DOC"
     * value 미사용 : 객체의 Full Package  전체명으로 생성 된다.\
       : "com.hyomee.service.solution.redis.doc.RedisHashDOC"
2. **Redis Key를 생성하기 위해서 @Id 지정 해야 한다.**&#x20;
   * @RedisHash("명") + @Id로 지정한 변수의 값\
     : @RedisHash(value = "REDIS\_HASH\_DOC") 지정 = "REDIS\_HASH\_DOC:LZC000002"
   * @RedisHash () + @Id \
     : com.hyomee.service.solution.redis.doc.RedisHashDOC:LZC000002"&#x20;
3. **secondary index  추가지정을 할려면 @Indexed 어노테이션 사용한다.**&#x20;

## 1. Redis와 연동할 객체 생성

@Id는 org.springframework.data.annotation.Id을 사용하고 있고, @RedisHas, @Indexeds는 org.springframework.data.redis.core을 사용하고 있는 것을 확인할 수 있다.&#x20;

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.redis.core.RedisHash;
import org.springframework.data.redis.core.index.Indexed;

@EqualsAndHashCode(onlyExplicitlyIncluded = true)
@ToString(onlyExplicitlyIncluded = true)
@Data
@RedisHash(value = "REDIS_HASH_DOC")
public class RedisHashDOC {

    @Id
    private String custChnlCd;

    @Indexed
    private String custChnlNm;
    
    private List<RedisDTO> redisDTOList;

}
```

1. @RedisHash("REDIS\_HASH\_DOC") : CacheName이 지정된다.
2. @Id : Redis Key 설정, @RedisHash("REDIS\_HASH\_DOC")으로 지정이 되어 있어서 "REDIS\_HASH\_DOC::id의 값"으로 Key가 만들어 진다.
3. @Indexed : 보조키 설정으로 key가 생성됩니다.\
   : @Indexed로  선언된 변수  \
   &#x20;  :: "REDIS\_HASH\_DOC:custChnlNm:\xea\xb3\xa0\xea\xb0\x9d\xec\xb1\x84\xeb\x84\x902"\
   : @id로 선언된 변수 \
   &#x20;  :: REDIS\_HASH\_DOC:LZC000002:idx"

## 2. @Repository 생성

Repository는 Spring에서 제공하는 어노테이션을 사용하여 Redis와 연동은 Spring에서 자동으로 구성하는 것을 확인 할 수 있다. 즉 Spring Data Redis도 Spring Data 프로젝트의 일부로 Spring Data 하위 프로젝트에서 외부 서버와의 연동은 Spring 에서 자동 구성되는 것을 알 수 있다.

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface RedisHashRepository extends JpaRepository<RedisHashDOC, String> {
     
}
```

## 3. 서비스 코드&#x20;

Spring을 사용한 일반적인 코드로 작성을 한다.

```java
@RequiredArgsConstructor
@Service
public class RedisHashService {
    
    // redisHashRepository 주입 
    private final RedisHashRepository redisHashRepository;


    // findById
    public RedisHashDOC getRedisHashDoc(String custChnlCd) {
        RedisHashDOC redisHashDOC = new RedisHashDOC();
        Optional<RedisHashDOC> redisHashDOCOptional = redisHashRepository.findById(custChnlCd);
        if (redisHashDOCOptional.isPresent()) {
            return redisHashDOCOptional.get();
        }
        return redisHashDOC ;

    }

    // save
    public RedisHashDOC saveRedisHashDoc(RedisHashDOC redisHashDOC) {
            return redisHashRepository.save(redisHashDOC) ;
    }

    // cache 삭제
    public void deleteAll() {
        redisHashRepository.deleteAll();
    }
}
```

## 4. 결과   확인v

### 4-1.  저장 하기 전 Redis&#x20;

```
127.0.0.1:6379> keys *
(empty array)
127.0.0.1:6379>
```

### 4-2. 저장&#x20;

```powershell
curl --location --request PUT 'http://localhost:8080/redis/hash' \
--header 'Content-Type: application/json; charset=UTF-8' \
--data '{
    "custChnlCd": "LZC000002",
    "custChnlNm": "고객채널2",
    "redisDTOList": [
        {
            "prodCd": "LZP000001",
            "prodNm": "요금상품1"
        },
        {
            "prodCd": "LZP000002",
            "prodNm": "요금상품2"
        }
    ]
}'
```

### 4-3 저장 처리 이후 Redis

```
127.0.0.1:6379> keys *
1) "REDIS_HASH_DOC:custChnlNm:\xea\xb3\xa0\xea\xb0\x9d\xec\xb1\x84\xeb\x84\x902"
2) "REDIS_HASH_DOC"
3) "REDIS_HASH_DOC:LZC000002"
4) "REDIS_HASH_DOC:LZC000002:idx"
127.0.0.1:6379>
```

### 4-4. findbyId

````powershell
curl --location --request GET 'http://localhost:8080/redis/hash/LZC000002' \
--header 'Content-Type: application/json; charset=UTF-8' \
--data '{
    "prodCd": "LZC000001",
    "prodNm": "고객채널널"
}'
===============================================
```json 결과
{
    "custChnlCd": "LZC000002",
    "custChnlNm": "고객채널2",
    "redisDTOList": [
        {
            "prodCd": "LZP000001",
            "prodNm": "요금상품1"
        },
        {
            "prodCd": "LZP000002",
            "prodNm": "요금상품2"
        }
    ]
}
```

````

### 4-5. 삭제 요청

```java
curl --location --request DELETE 'http://localhost:8080/redis/hash' \
--header 'Content-Type: application/json; charset=UTF-8' \
--data '{
    "prodCd": "LZC000001",
    "prodNm": "고객채널널"
}'

```

### 4-6. 삭제 요청 후 Redis

key 모두 삭제가 되는 것을 확인 할 수 있습니다.

```
127.0.0.1:6379> keys *
(empty array)
127.0.0.1:6379>
```

Spring-Data
