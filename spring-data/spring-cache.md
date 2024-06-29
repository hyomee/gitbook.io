# Spring Cache

## 1. 캐싱이란?

캐싱은 자주 사용되는 데이터를 임시 저장소에 저장하여 빠르게 접근할 수 있도록 하는 기술입니다. 이를 통해 데이터 처리 속도를 향상시키고 서버의 부하를 줄일 수 있습니다. 예를 들어, 피보나치 계산과 같은 CPU 집약적인 작업이나 디스크에서 파일을 읽는 IO 비용이 많이 드는 작업의 비용을 줄이는 데 도움이 됩니다

## 2. Spring Framework의 캐시 구현

Spring Framework는 캐싱을 쉽게 구현할 수 있도록 다양한 기능을 제공합니다. Spring에서는 @Cacheable, @CachePut, @CacheEvict와 같은 어노테이션을 사용하여 캐시된 데이터를 쉽게 조작할 수 있습니다.

### 2-1. 선언적 주석 기반

선언적 어노테이션 기반은 어노테이션을 사용하여 캐싱을 관리할 수 있게 해줍니다. 이는 단순히 어노테이션을 메서드에 추가하기만 하면 됩다.

#### **2-1-1.  @Cacheable**:&#x20;

Cacheable은 호출 메서드의 결과가 나중에 사용하기 위해 캐시에 저장되어야 함을 나타냅니다. 기본적으로 Spring Cache 추상화는 캐시 키 생성을 위해 SimpleKeyGenerator라는 기본 키 생성기를 사용합니다.

```java
```

* **value**:  cacheNames에 대한 별칭&#x20;
* **cacheNames**:  메서드 호출 결과가 저장되는 캐시의 이름&#x20;
* **key**: 키를 동적으로 계산하기 위한 SpEL(Spring Expression Language) 식으로 Empty()가 기본값입니다.
* **keyGenerator**:  사용할 커스텀의 Bean 이름입니다.
* **cacheManager**:  아직 설정되지 않은 경우 기본값을 만드는 데 사용할 사용자 정의의 Bean 이름입니다. (기본 org.springframework.cache.interceptor.CacheResolver를 생성하는 데 사용할 사용자 정의 org.springframework.cache.CacheManager의 빈 이름).
* **cacheResolver**: 사용할 커스텀의 Bean 이름입니다.(org.springframework.cache.interceptor.CacheResolver)
* **condition**:  메서드 캐싱을 조건부로 만드는 데 사용되는 SpEL(Spring Expression Language) 표현식입니다. 기본값은 ""입니다. 이는 메서드 결과가 항상 캐시된다는 의미입니다.
* **unless:** 메소드 캐싱을 거부하는 데 사용되는 SpEL(Spring Expression Language) 표현식입니다. 조건과 달리 이 표현식은 메서드가 호출된 후에 평가되므로 결과를 참조할 수 있습니다. 기본값은 ""입니다. 이는 캐싱이 거부되지 않음을 의미합니다.
* **sync**:  여러 스레드가 동일한 키에 대한 값을 로드하려고 시도하는 경우 기본 메서드 호출을 동기화합니다. 동기화에는 몇 가지 제한 사항이 있습니다.

#### **2-1-2. @CachePut**:&#x20;

캐싱 시스템에서는 캐시 값이 수정하는 기능입니다.

```java
// Some code
```

#### **2-1-3. @CacheEvict**:&#x20;

캐시에서 특정 항목을 제거합니다.

#### **2-1-4. @**CacheConfig:&#x20;

클래스 수준에서 일반적인 캐시 관련 설정을 공유하기 위한 메커니즘을 제공합니다. 이 주석이 특정 클래스에 있으면 해당 클래스에 정의된 모든 캐시 작업에 대한 기본 설정 집합을 제공합니다.

프로그래밍에서는 항상 코드를 재사용하고 상용구 코드를 제거하려고 하므로 CacheConfig를 적용하는 것이 도움이 됩니다.

#### **2-1-5. @**EnableCaching:

@EnableCaching 어노테이션을 사용하면 캐싱 기능을 활성화할 수 있습니다. Spring은 모든 빈을 스캔하여 공용 메서드에 캐싱 어노테이션이 있는지 확인합니다. 어노테이션이 발견되면, 메서드 호출을 가로채고 캐싱 동작을 처리하기 위해 자동으로 프록시를 생성합니다.

<mark style="color:orange;">**@EnableCaching 주석을 지정하지 않으면 @Cachable, @CachePut, @CacheEvict가 작동하지 않습니다.**</mark>

#### 2-1-6. 사용자 주석(Customizing)

Spring Cache Abstraction을 사용하면 자체 주석을 만들 수 있습니다

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
@Cacheable(value=“farms”, key="#farmcode)
public @interface FarmsCacheable {
}
```



### 2-2. XML-based configuration

주석 기반 접근 방식 외에도 Spring Cache Abstraction은 XML 기반 주석도 지원합니다. 이 접근 방식을 사용하면 소스 코드를 수정할 수 없는 기존 응용 프로그램에 캐시를 추가할 수 있습니다.

참고: [https://docs.spring.io/spring-framework/docs/5.2.10.RELEASE/spring-framework-reference/integration.html#cache-declarative-xml](https://docs.spring.io/spring-framework/docs/5.2.10.RELEASE/spring-framework-reference/integration.html#cache-declarative-xml)

## 3. 제공되는 프로바이더

* [JCache (JSR-107)](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-jcache)
* [EhCache 2.x](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-ehcache2)
* [Hazelcast](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-hazelcast)
* [Infinispan](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-infinispan)
* [Couchbase](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-couchbase)
* [Redis](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-redis)
* [Caffeine](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-caffeine)

\
