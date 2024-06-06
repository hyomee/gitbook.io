# Spring Boot RabbitMQ

RabbitMQ는 AMQP(Advanced Message Queuing Protocol)를 구현한 오픈소스 메시지 브로커로, 서로 다른 서비스 간의 통신을 중개하여 분리된 방식으로 용이하게 합니다. Spring Boot와 결합될 때, JAVA 기반의 마이크로서비스나 메시징 시스템을 구축하는 데에 강력한 솔루션을 제공합니다. 비동기 메시징을 사용하는 시스템 간 통신은 구성 요소들이 작업 완료를 기다리지 않고도 작업을 지속할 수 있게 해줍니다. 이러한 구성 요소의 분리는 시스템의 복원력, 확장성 및 응답성을 향상시키는 데 기여합니다.

**비동기 메시징의 이점**은 다음과 같습니다.

1. 디커플링(Decoupling): 구성 요소가 독립적으로 작동하여 다른 구성 요소에 영향을 주지 않고 한 구성 요소를 변경할 수 있습니다.
2. 확장성(Scalability): 비동기 통신을 사용하면 워크로드에 따라 개별 구성 요소의 크기를 조정할 수 있습니다.
3. 복원력(Resilience): 시스템은 구성 요소의 오류 또는 일시적인 사용 불가를 더 잘 처리할 수 있습니다.
4. 응답성 향상(Increased Responsiveness): 구성 요소는 실제 처리에 시간이 걸리더라도 사용자 요청에 즉시 응답할 수 있습니다.

## 1. Spring Boot와 RabbitMQ 통합

Spring Boot와 RabbitMQ를 통합하기 위해서는 종족성 설정,  RabbitMQ 속성 구성, 메시지를 보낼 메시지 생성자 생성, 메시지 수신 및 처리를 위한 메시지 소비자 개발을 비롯한 여러 단계를 따라야 합니다.

### 1-1. 의존성 추가

Maven을 사용하는 경우) 또는 (Gradle을 사용하는 경우) 파일을 열고 다음 종속성울 추가  합니다.

{% code title="pom.xml" lineNumbers="true" %}
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.3</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<!-- 1. Web Application  -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.2.3</version>
</dependency>

<!-- 2. RabbitMQ   -->          
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>3.2.3</version>
</dependency>

<!-- 3. RabbitMQ - Stream  --> 
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit-stream</artifactId>
    <version>3.1.2</version>
</dependency>
```
{% endcode %}

### 1-2. RabbitMQ 속성 구성

application.propertie(or sapplication.yml) 파일에 RabbitMQ 연결 속성을 추가합니다.

{% code title="sapplication.yml" lineNumbers="true" %}
```yaml
spring:
  application:
    name: acube-rabbitmq
  rabbitmq:
    dynamic: true
    host: localhost
    port: 5673
    username: guest
    password: guest
```
{% endcode %}

* **rabbitmq.host**: RabbitMQ 호스트를 지정합니다. 이는 RabbitMQ 서버의 주소입니다.
* **rabbitmq.virtualhost**: 연결할 브로커의 가상 호스트를 설정합니다. RabbitMQ에서 가상 호스트는 논리적인 분리를 위해 사용됩니다.
* **rabbitmq.port**: RabbitMQ 서버와 통신할 때 사용할 포트 번호입니다. 기본값은 5672입니다.
* **rabbitmq.username**: RabbitMQ 브로커에 인증하기 위한 로그인 사용자 이름입니다.
* **rabbitmq.password**: RabbitMQ 브로커에 인증하기 위한 비밀번호입니다.
* **rabbitmq.exchange**: 메시지 전송 작업에 사용할 교환기(exchange)의 이름입니다. 교환기는 메시지를 큐로 라우팅하는 역할을 합니다.
* **rabbitmq.queue**: 메시지가 저장되는 메시지 큐의 이름입니다.
* **rabbitmq.routingkey**: 라우팅 키의 이름입니다. 라우팅 키는 메시지를 어떤 큐로 보낼지 결정하는 역할을 합니다.
* **rabbitmq.reply.timeout**: 소비자(delivery)의 확인을 강제하는 타임아웃입니다. 이는 소비자가 메시지를 확인하지 않는 버그를 감지하는 데 도움이 됩니다.
* **rabbitmq.concurrent.consumers**: 여러 프로듀서와 컨슈머가 동일한 큐에서 읽고 쓸 때 중요한 필드입니다.
* **rabbitmq.max.concurrent.consumers**: 동시 컨슈머의 수를 나타냅니다. 예제에서는 단일 컨슈머만 사용하므로 해당 필드는 중요하지 않습니다.

### 1-3. RabbitMQ 구성 환경 설정

RabbitMQ에 이미 Exchange와 Queue가 생성되어 있다면, 1-3 단계의 코드 작성은 필요하지 않습니다. RabbitMQ의 Exchange, Queue, Binding을 API를 통해 생성할 경우 이미 존재한다면 이는 무시됩니다.

Spring Boot에서 Exchange, Queue, Binding을 API로 생성하는 두 가지 방법은 Bean 주입과 AmqpAdmin 사용입니다.

#### 1-3-1. Bean 주입

1. **Exchange 주입 :** Exchange 객체를 생성 하여 주입합니다.
   *   **new 직접 생성:**  DirectExchange, FanoutExchange, TopicExchange, HeadersExchange 를 사용해서 객체 생성한 것을 Bean으로 주입합니다.

       <pre class="language-java"><code class="lang-java"><strong>@Bean
       </strong>public Exchange directExchange() {
           // durable=true, autoDelete=false
           return new DirectExchange(DIRECT_EXCHANGE, true, false);
       } 
       </code></pre>
   *   **ExchangeBuilder 통한 생성**: ExchangeBuilder 를 사용하여 객체를 생성하여 Bean으로 주입합니다.

       ```java
       @Bean
       TopicExchange myTopicExchange() {
           return ExchangeBuilder.topicExchange(TOPIC_EXCHANGE)
                   .durable(true).build();
       }

       @Bean
       public FanoutExchange myFanoutExchange() {
           return ExchangeBuilder.fanoutExchange(FANOUT_EXCHANGE)
                   .durable(true).build();
       }
       ```
2.  **Queue 주입: Queue 객체를 생성하여 주입**&#x20;

    ```java
    @Bean
    public Queue createDirectEmailQueue() {
        //For learning purpose - durable=false,
        // in a real project you may need to set this as true.
        return new Queue(DIRECT_QUEUE_EMAIL, true);
    }
    ```
3.  **Binding 설정:**&#x20;

    ```java
    @Bean
    public Binding queueDirectEmailBinding() {
        return new Binding(DIRECT_QUEUE_EMAIL, Binding.DestinationType.QUEUE, DIRECT_EXCHANGE, "email", null);
    }
    ```
4. Jons
5. **ㅇㅁㄴㅇ**

