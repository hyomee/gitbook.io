# Spring Boot

RabbitMQ는 AMQP(Advanced Message Queuing Protocol)를 구현한 오픈소스 메시지 브로커로, 서로 다른 서비스 간의 통신을 중개하여 분리된 방식으로 용이하게 합니다. Spring Boot와 결합될 때, JAVA 기반의 마이크로서비스나 메시징 시스템을 구축하는 데에 강력한 솔루션을 제공합니다. 비동기 메시징을 사용하는 시스템 간 통신은 구성 요소들이 작업 완료를 기다리지 않고도 작업을 지속할 수 있게 해줍니다. 이러한 구성 요소의 분리는 시스템의 복원력, 확장성 및 응답성을 향상시키는 데 기여합니다.

참고: [Baeldung RabbitMQ 검색](https://www.baeldung.com/?s=RabbitMQ)

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
* **spring.rabbitmq.dynamic**: AmqpAdmin 빈을 생성할지 여부를 설정합니다. 기본값은 true입니다.

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

#### 1-3-2. AmqpAdmin&#x20;

RabbitMQ 연결을 Spring 없이 작성하는 방법과 동일하게 작성 하기 위해서는 Spring Boot RabbitMQ 환경 구성을 해야 합니다.

*   RabbitAdmin을 구성하려면 AmqpAdmin Bean을 선언하거나 ApplicationContext에 정의해야 합니다. 이것은 큐를 자동으로 선언하고 바인딩하는 데 도움이 됩니다. 여기서 connectionFactory는 RabbitMQ에 연결하기 위한 메서드입니다.

    ```java
    @Bean
    public AmqpAdmin amqpAdmin() {
        return new RabbitAdmin(connectionFactory());
    }
    ```

<details>

<summary>AmqpAdmin을 사용한 선언 </summary>

```java
import org.springframework.amqp.rabbit.annotation.EnableRabbit;
import org.springframework.context.annotation.Configuration;
import org.springframework.amqp.core.AmqpAdmin;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;

@Configuration
@EnableRabbit
public class RabbitMqConfiguration {

    private final AmqpAdmin rabbitAdmin;

    public RabbitMqConfiguration(AmqpAdmin rabbitAdmin) {
        this.rabbitAdmin = rabbitAdmin;
    }

    public void declareQueue() {
        String name = "my-queue";
        String routingKey = "my-key";
        boolean durable = true;
        boolean exclusive = false;
        boolean autoDelete = false;


        String queueName = rabbitAdmin.declareQueue(new Queue(name, durable, exclusive, autoDelete));

        String exchangeName = "my-exchange";
        DirectExchange exchange = new DirectExchange(exchangeName, durable, autoDelete);
        rabbitAdmin.declareExchange(exchange);

        DestinationType destinationType = DestinationType.QUEUE;
        Map<String, Object> arguments = null;
        Binding binding = new Binding(queueName, destinationType, exchangeName, routingKey,
            arguments);
        rabbitAdmin.declareBinding(binding);
    }
}
```

</details>

* RabbitAdmin으로 하는 다른 방법으로 AmqpAdmin에 Exchange, Queue, Binding를 설정하는 예제 입니다. ( fanoutExchange )

{% tabs %}
{% tab title="Config 파일" %}
```java
@Configuration
public class RabbitFantOutConfig {
    public static final String FANOUT_EXCHANGE = "My-Fanout-Exchange";
    public static final String FANOUT_QUEUE_EMAIL = "My-Fanout-Email-Q";
    public static final String FANOUT_QUEUE_SMS = "My-Fanout-Sms-Q";
    public static final String FANOUT_QUEUE_SNS = "My-Fanout-Sns-Q";
    
    public FanoutExchange myFanoutExchange() {
        return ExchangeBuilder.fanoutExchange(FANOUT_EXCHANGE)
                .durable(true).build();
    }


    public List<Queue> fanoutQueues() {
        return List.of(new Queue(FANOUT_QUEUE_EMAIL, true),
                new Queue(FANOUT_QUEUE_SMS, true),
                new Queue(FANOUT_QUEUE_SNS, true));
    }


    public List<Binding> fanoutBindings() {
        return List.of(new Binding(FANOUT_QUEUE_EMAIL, Binding.DestinationType.QUEUE, FANOUT_EXCHANGE, "email", null),
                new Binding(FANOUT_QUEUE_SMS, Binding.DestinationType.QUEUE, FANOUT_EXCHANGE, "", null),
                new Binding(FANOUT_QUEUE_SNS, Binding.DestinationType.QUEUE, FANOUT_EXCHANGE, "", null));
    }


    @Bean
    public RabbitMqCreationConfig fanoutCreationConfig(AmqpAdmin rabbitAdmin  ) {
        return new RabbitMqCreationConfig( rabbitAdmin,
                 myFanoutExchange() ,
                 fanoutQueues(),
                 fanoutBindings());
    }

}
```
{% endtab %}

{% tab title="AmqpAdmin 을 통한 선언 " %}
```java
public class RabbitMqCreationConfig {

    public  RabbitMqCreationConfig(AmqpAdmin rabbitAdmin,
                                   FanoutExchange myFanoutExchange,
                                   List<Queue> fanoutQueues,
                                   List<Binding> fanoutBindings) {

        rabbitAdmin.declareExchange(myFanoutExchange);

        // Queue 생성 (RabbitMQ 에 없는 경우에만, 이미 있으면 ignore)
        for (Queue queue : fanoutQueues) {
            rabbitAdmin.declareQueue(queue);
        }

        // Binding 생성 (RabbitMQ 에 없는 경우에만, 이미 있으면 ignore)
        for (Binding binding : fanoutBindings) {
            rabbitAdmin.declareBinding(binding);
        }
    }
}
```
{% endtab %}
{% endtabs %}

#### 1-3-3. Jons To Bean을 하기위한 Convert

메시지가 JSON 형식으로 전달되도록 하려면 _**MessageConverter**_에 대한 Bean을 작성합니다.

```java
public static final String AMQP_OBJECT_MAPPER_CONVERT   = "AmqpObjectMapperConvert";


@Bean(name = AMQP_OBJECT_MAPPER_CONVERT)
public MessageConverter amqpObjectMapperConvert() {

    ObjectMapper configure
            = new ObjectMapper()
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);

    return new Jackson2JsonMessageConverter(configure);
}
```

#### 1-3-4. _**ConnectionFactory**_ Bean

RabbitMQ에 연결하기 위해 _**ConnectionFactory**_ Bean을 등록하면 다양한 속성을 제어 할 수 있습니다.

```java
@Bean
public ConnectionFactory connectionFactory() {
    CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
    connectionFactory.setVirtualHost(virtualHost);
    connectionFactory.setHost(host);
    connectionFactory.setUsername(username);
    connectionFactory.setPassword(password);
    return connectionFactory;
}
```

#### 1-3-5. rabbitTemplate Bean

_**rabbitTemplate**_의 Bean은 대기열에 메시지를 보내는 목적으로 사용되며 두 가지 방법으로 사용합니다.

*   **new 로 생성:** RabbitAutoConfiguration에서 제공하는 default rabbitTemplate 를 사용하지 않고 재정의 합니다.

    ```java
    public AmqpTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        final RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setDefaultReceiveQueue(queueName);
        rabbitTemplate.setMessageConverter(amqpObjectMapperConvert());
        rabbitTemplate.setReplyAddress(queue().getName());
        rabbitTemplate.setReplyTimeout(replyTimeout);
        rabbitTemplate.setUseDirectReplyToContainer(false);
        return rabbitTemplate;
    }
    ```
*   **추가기능 설정**:   RabbitAutoConfiguration에서 제공하는 default rabbitTemplate 에 추가 기능 설정

    ```java
    @Bean
    public RabbitTemplateCustomizer rabbitTemplateCustomizer() {
        return rabbitTemplate -> rabbitTemplate.setMessageConverter(amqpObjectMapperConvert());
    }
    ```

#### 1-3-6. _**SimpleRabbitListenerContainerFactory**_

Spring Boot에서 RabbitMQ 메시지 리스너를 구성하는 데 사용되는 클래스입니다. 이 클래스는 기본적인 설정을 제공하며, 필요한 경우 사용자 정의 설정을 추가할 수 있습니다. 즉 RabbitMQ 메시지를 처리하는 리스너 컨테이너를 생성하는 데 사용됩니다. 이를 통해 메시지를 수신하고 처리할 수 있습니다.

```java
@Bean
public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory() {
    final SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory());
    factory.setMessageConverter(jsonMessageConverter());
    factory.setConcurrentConsumers(concurrentConsumers);
    factory.setMaxConcurrentConsumers(maxConcurrentConsumers);
    factory.setErrorHandler(errorHandler());
    return factory;
}
```

#### 1-3-7. _**errorHandler**_

_**errorHandler**_는 리스너에서 예외가 발생할 때 사용자에게 친숙한 Object를 반환하는 데 사용됩니다

```java
@Bean
public ErrorHandler errorHandler() {
    return new ConditionalRejectingErrorHandler(new MyFatalExceptionStrategy());
}

public static class MyFatalExceptionStrategy extends ConditionalRejectingErrorHandler.DefaultExceptionStrategy {
    private final Logger logger = LogManager.getLogger(getClass());
    @Override
    public boolean isFatal(Throwable t) {
        if (t instanceof ListenerExecutionFailedException) {
            ListenerExecutionFailedException lefe = (ListenerExecutionFailedException) t;
            logger.error("Failed to process inbound message from queue "
                    + lefe.getFailedMessage().getMessageProperties().getConsumerQueue()
                    + "; failed message: " + lefe.getFailedMessage(), t);
        }
        return super.isFatal(t);
    }
}
```

<details>

<summary>RabbitMQ 설정<br><br>참고: <a href="https://www.appsdeveloperblog.com/messaging-using-rabbitmq-in-spring-boot-application/">https://www.appsdeveloperblog.com/messaging-using-rabbitmq-in-spring-boot-application/</a></summary>

```java
#RabbitMQ settings
rabbitmq.host=localhost
rabbitmq.virtualhost=/
rabbitmq.port=15672
rabbitmq.username=guest
rabbitmq.password=guest
rabbitmq.exchange=rabbitmq.exchange
rabbitmq.queue=rabbitmq.queue
rabbitmq.routingkey=rabbitmq.routingkey
rabbitmq.reply.timeout=60000
rabbitmq.concurrent.consumers=1
rabbitmq.max.concurrent.consumers=1


@EnableRabbit
@Configuration
public class RabbitMQConfig {
    @Value("${rabbitmq.queue}")
    private String queueName;
    @Value("${rabbitmq.exchange}")
    private String exchange;
    @Value("${rabbitmq.routingkey}")
    private String routingkey;
    @Value("${rabbitmq.username}")
    private String username;
    @Value("${rabbitmq.password}")
    private String password;
    @Value("${rabbitmq.host}")
    private String host;
    @Value("${rabbitmq.virtualhost}")
    private String virtualHost;
    @Value("${rabbitmq.reply.timeout}")
    private Integer replyTimeout;
    @Value("${rabbitmq.concurrent.consumers}")
    private Integer concurrentConsumers;
    @Value("${rabbitmq.max.concurrent.consumers}")
    private Integer maxConcurrentConsumers;
    @Bean
    public Queue queue() {
        return new Queue(queueName, false);
    }
    @Bean
    public DirectExchange exchange() {
        return new DirectExchange(exchange);
    }
    @Bean
    public Binding binding(Queue queue, DirectExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(routingkey);
    }
    @Bean
    public MessageConverter jsonMessageConverter() {
        ObjectMapper objectMapper = new ObjectMapper();
        return new Jackson2JsonMessageConverter(objectMapper);
    }
    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setVirtualHost(virtualHost);
        connectionFactory.setHost(host);
        connectionFactory.setUsername(username);
        connectionFactory.setPassword(password);
        return connectionFactory;
    }
    @Bean
    public AmqpTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        final RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setDefaultReceiveQueue(queueName);
        rabbitTemplate.setMessageConverter(jsonMessageConverter());
        rabbitTemplate.setReplyAddress(queue().getName());
        rabbitTemplate.setReplyTimeout(replyTimeout);
        rabbitTemplate.setUseDirectReplyToContainer(false);
        return rabbitTemplate;
    }
    @Bean
    public AmqpAdmin amqpAdmin() {
        return new RabbitAdmin(connectionFactory());
    }
    @Bean
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory() {
        final SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        factory.setMessageConverter(jsonMessageConverter());
        factory.setConcurrentConsumers(concurrentConsumers);
        factory.setMaxConcurrentConsumers(maxConcurrentConsumers);
        factory.setErrorHandler(errorHandler());
        return factory;
    }
    @Bean
    public ErrorHandler errorHandler() {
        return new ConditionalRejectingErrorHandler(new MyFatalExceptionStrategy());
    }
    public static class MyFatalExceptionStrategy extends ConditionalRejectingErrorHandler.DefaultExceptionStrategy {
        private final Logger logger = LogManager.getLogger(getClass());
        @Override
        public boolean isFatal(Throwable t) {
            if (t instanceof ListenerExecutionFailedException) {
                ListenerExecutionFailedException lefe = (ListenerExecutionFailedException) t;
                logger.error("Failed to process inbound message from queue "
                        + lefe.getFailedMessage().getMessageProperties().getConsumerQueue()
                        + "; failed message: " + lefe.getFailedMessage(), t);
            }
            return super.isFatal(t);
        }
    }
}
```

</details>

## 2. Consumer

_**@RabbitListener**_ 는 들어오는 메시지에 대해 RabbitMQ 대기열을 수신할 수 있습니다.

```java
@Service
public class MessageListener {

    @RabbitListener(queues = FANOUT_QUEUE_SNS)
    public void receiveFanoutSnsMessage(String message) {
        System.out.println("My-Fanout-Sns-Q Received Message:" + message);
        System.out.println();
    }
    
    @RabbitListener(queues = DIRECT_QUEUE_OBJECT,
            messageConverter = AMQP_OBJECT_MAPPER_CONVERT)
    public void secondQueueListener(CustomerRecord customerRecord) {
        System.out.printf("Message From " + DIRECT_QUEUE_OBJECT + " => %s%n", customerRecord.getInfo());
    }
    
}
```

* secondQueueListener 메소드를 @RabbitHandler사용으로 변경한 코드 입니다.\
  \
  페이로드 유형에 따라 읽기 데이터를 분류할 때는 _**@RabbitHandler 사용하면**_ 서로 다른 메서드를 정의하고, 각각에 @RabbitHandler 주석을 달아 동일한 대기열에서 여러 데이터 유형 페이로드를 사용합니다.

```java
@Component
@RabbitListener(queues = DIRECT_QUEUE_OBJECT,
        messageConverter = AMQP_OBJECT_MAPPER_CONVERT)
@Slf4j
public class RabbitMQConsumer {

    @RabbitHandler
    public void consumer(CustomerRecord customerRecord) {
        log.info(DIRECT_QUEUE_OBJECT + " :: Consuming Message : " + customerRecord.getInfo());
    }
}
```

## 3. Publisher

rabbitTemplate.convertAndSend 메서드를 사용하여 발행 할 수 있습니다.

```java
@RestController
@RequiredArgsConstructor
public class MessageController {

    private final AmqpTemplate amqpTemplate;

    @GetMapping("/directMsg/{routingkey}")
    public String sendMessage(@PathVariable String routingkey,
                              @RequestParam("message") String message) {
        amqpTemplate.convertAndSend(RabbitMQDirectConfig.DIRECT_EXCHANGE, routingkey, message);
        return "Message Sent";
    }

    @PostMapping("/directObjectMsg/{routingkey}")
    public String sendDirestMessage(@PathVariable String routingkey,
                              @RequestBody CustomerRecord customerRecord) {

        amqpTemplate.convertAndSend(RabbitMQDirectConfig.DIRECT_OBJECT_EXCHANGE, "", customerRecord);
        return "Message Sent";
    }

    @GetMapping("/topicMsg/{routingkey}")
    public String sendTopicMessage(@PathVariable String routingkey,
                              @RequestParam("message") String message) {
        amqpTemplate.convertAndSend(RabbitMQTopicConfig.TOPIC_EXCHANGE, routingkey, message);
        return "Message TOPIC Sent";
    }

    @GetMapping("/fanoutMsg/{routingkey}")
    public String sendfanoutMessage(@PathVariable String routingkey,
                                   @RequestParam("message") String message) {
        amqpTemplate.convertAndSend(RabbitFantOutConfig.FANOUT_EXCHANGE, routingkey, message);
        return "Message Fanout Sent";
    }
}
```



## 4. 데이터 수/발신용 record&#x20;

```java
public record CustomerRecord(@JsonProperty("name") String name,
                             @JsonProperty("age") Integer age) {

    public CustomerRecord {
        if (name == null || age == null  ) {
            throw new IllegalArgumentException();
        }
    }

    @JsonIgnore
    public String getInfo() {
        return this.name + " " + this.age;
    }
}
```

## 5. 전체소스

{% tabs %}
{% tab title="application.yml" %}
```yaml
spring:
  application:
    name: acube-rabbitmq
  rabbitmq:
    dynamic: true
    host: x.x.x.x
    port: 21003
    username: xxxx
    password: xxxx!
```
{% endtab %}

{% tab title="RabbitMQDirectConfig" %}
```java
@Configuration
public class RabbitMQDirectConfig {
    public static final String DIRECT_EXCHANGE = "My-Direct-Exchange";

    public static final String DIRECT_QUEUE_EMAIL = "My-Direct-Email-Q";
    public static final String DIRECT_QUEUE_SMS = "My-Direct-Sms-Q";
    public static final String DIRECT_QUEUE_SNS = "My-Direct-Sns-Q";

    public static final String DIRECT_OBJECT_EXCHANGE = "My-Direct-Object-Exchange";
    public static final String DIRECT_QUEUE_OBJECT = "My-Direct-Object-Q";


    @Bean
    public Exchange directExchange() {
        // durable=true, autoDelete=false
        
        return new DirectExchange(DIRECT_EXCHANGE, true, false);
    }



    @Bean
    public Queue createDirectEmailQueue() {
        //For learning purpose - durable=false,
        // in a real project you may need to set this as true.
        return new Queue(DIRECT_QUEUE_EMAIL, true);
    }




    @Bean
    public Binding queueDirectEmailBinding() {
        return new Binding(DIRECT_QUEUE_EMAIL, Binding.DestinationType.QUEUE, DIRECT_EXCHANGE, "email", null);
    }

    @Bean
    public Queue createDirectSmsQueue() {
        //For learning purpose - durable=false,
        // in a real project you may need to set this as true.
        return new Queue(DIRECT_QUEUE_SMS, true);
    }

    @Bean
    public Binding queueDirectSmsBinding() {
        return new Binding(DIRECT_QUEUE_SMS, Binding.DestinationType.QUEUE, DIRECT_EXCHANGE, "sms", null);
    }

    @Bean
    public Queue createDirectSnsQueue() {
        //For learning purpose - durable=false,
        // in a real project you may need to set this as true.
        return new Queue(DIRECT_QUEUE_SMS, true);
    }

    @Bean
    public Binding queueDirectSndBinding() {
        return new Binding(DIRECT_QUEUE_SNS, Binding.DestinationType.QUEUE, DIRECT_EXCHANGE, "sns", null);
    }

    @Bean
    public Exchange directObjectExchange() {
        // durable=true, autoDelete=false
        return new DirectExchange(DIRECT_OBJECT_EXCHANGE, true, false);
    }

    @Bean
    public Queue createDirectObjectQueue() {
        //For learning purpose - durable=false,
        // in a real project you may need to set this as true.
        return new Queue(DIRECT_QUEUE_OBJECT, true);
    }

    @Bean
    public Binding queueDirectObjectBinding() {
        return new Binding(DIRECT_QUEUE_OBJECT, Binding.DestinationType.QUEUE, DIRECT_OBJECT_EXCHANGE, "", null);
    }

}
```
{% endtab %}

{% tab title="RabbitMQTopicConfig " %}
```java
@Configuration
public class RabbitMQTopicConfig {
    public static final String TOPIC_EXCHANGE = "My-Topic-Exchange";
    public static final String TOPIC_QUEUE_EMAIL = "My-Topic-Email-Q";
    public static final String TOPIC_QUEUE_SMS = "My-Topic-Sms-Q";
    public static final String TOPIC_QUEUE_SNS = "My-Topic-Sns-Q";
    public static final String TOPIC_QUEUE_EMPTY = "My-Topic-Empty-Q";


    @Bean
    TopicExchange myTopicExchange() {
        return ExchangeBuilder.topicExchange(TOPIC_EXCHANGE)
                .durable(true).build();
    }

    @Bean
    Queue topicQueueEamil() {
        return QueueBuilder.durable(TOPIC_QUEUE_EMAIL).build();
    }

    @Bean
    Queue topicQueueSMS() {
        return QueueBuilder.durable(TOPIC_QUEUE_SMS).build();
    }

    @Bean
    Queue topicQueueSNS() {
        return QueueBuilder.durable(TOPIC_QUEUE_SNS).build();
    }

    @Bean
    Queue topicQueueEmpty() {
        return QueueBuilder.durable(TOPIC_QUEUE_EMPTY).build();
    }

    @Bean
    Binding topicBindingEmail(Queue topicQueueEamil, TopicExchange myTopicExchange) {
        return BindingBuilder.bind(topicQueueEamil).to(myTopicExchange).with("email.*");
    }

    @Bean
    Binding topicBindingSMS(Queue topicQueueSMS, TopicExchange myTopicExchange) {
        return BindingBuilder.bind(topicQueueSMS).to(myTopicExchange).with("#.sms.*");
    }

    @Bean
    Binding topicBindingSNS(Queue topicQueueSNS, TopicExchange myTopicExchange) {
        return BindingBuilder.bind(topicQueueSNS).to(myTopicExchange).with("#.sns");
    }

    @Bean
    Binding topicBindingEmpty(Queue topicQueueEmpty, TopicExchange myTopicExchange) {
        return BindingBuilder.bind(topicQueueEmpty).to(myTopicExchange).with("*");
    }
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="RabbitFantOutConfig " %}
```java
blic class RabbitMqCreationConfig {

    public  RabbitMqCreationConfig(AmqpAdmin rabbitAdmin,
                                       FanoutExchange myFanoutExchange,
                                       List<Queue> fanoutQueues,
                                       List<Binding> fanoutBindings) {

        rabbitAdmin.declareExchange(myFanoutExchange);

        // Queue 생성 (RabbitMQ 에 없는 경우에만, 이미 있으면 ignore)
        for (Queue queue : fanoutQueues) {
            rabbitAdmin.declareQueue(queue);
        }

        // Binding 생성 (RabbitMQ 에 없는 경우에만, 이미 있으면 ignore)
        for (Binding binding : fanoutBindings) {
            rabbitAdmin.declareBinding(binding);
        }
    }
}


@Configuration
public class RabbitFantOutConfig {
    public static final String FANOUT_EXCHANGE = "My-Fanout-Exchange";
    public static final String FANOUT_QUEUE_EMAIL = "My-Fanout-Email-Q";
    public static final String FANOUT_QUEUE_SMS = "My-Fanout-Sms-Q";
    public static final String FANOUT_QUEUE_SNS = "My-Fanout-Sns-Q";



    @Bean
    public FanoutExchange myFanoutExchange() {
        return ExchangeBuilder.fanoutExchange(FANOUT_EXCHANGE)
                .durable(true).build();
    }


    public List<Queue> fanoutQueues() {
        return List.of(new Queue(FANOUT_QUEUE_EMAIL, true),
                new Queue(FANOUT_QUEUE_SMS, true),
                new Queue(FANOUT_QUEUE_SNS, true));
    }


    public List<Binding> fanoutBindings() {
        return List.of(new Binding(FANOUT_QUEUE_EMAIL, Binding.DestinationType.QUEUE, FANOUT_EXCHANGE, "email", null),
                new Binding(FANOUT_QUEUE_SMS, Binding.DestinationType.QUEUE, FANOUT_EXCHANGE, "", null),
                new Binding(FANOUT_QUEUE_SNS, Binding.DestinationType.QUEUE, FANOUT_EXCHANGE, "", null));
    }


    @Bean
    public RabbitMqCreationConfig fanoutCreationConfig(AmqpAdmin rabbitAdmin,
                                                       FanoutExchange myFanoutExchange ) {
        return new RabbitMqCreationConfig( rabbitAdmin,
                 myFanoutExchange,
                 fanoutQueues(),
                 fanoutBindings());
    }

}

```
{% endtab %}

{% tab title="AmqpObjectMapperConvertConfig " %}
```java
@Configuration
public class AmqpObjectMapperConvertConfig {

    public static final String AMQP_OBJECT_MAPPER_CONVERT   = "AmqpObjectMapperConvert";


    @Bean(name = AMQP_OBJECT_MAPPER_CONVERT)
    public MessageConverter amqpObjectMapperConvert() {

        ObjectMapper configure
                = new ObjectMapper()
                .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);

        return new Jackson2JsonMessageConverter(configure);
    }

    // RabbitAutoConfiguration 에서 제공하는 default rabbitTemplate 에 추가 기능 설정
    @Bean
    public RabbitTemplateCustomizer rabbitTemplateCustomizer() {
        return rabbitTemplate -> rabbitTemplate.setMessageConverter(amqpObjectMapperConvert());
    }
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="MessageController " %}
```java
@RestController
@RequiredArgsConstructor
public class MessageController {

    private final AmqpTemplate amqpTemplate;

    @GetMapping("/directMsg/{routingkey}")
    public String sendMessage(@PathVariable String routingkey,
                              @RequestParam("message") String message) {
        amqpTemplate.convertAndSend(RabbitMQDirectConfig.DIRECT_EXCHANGE, routingkey, message);
        return "Message Sent";
    }

    @PostMapping("/directObjectMsg/{routingkey}")
    public String sendDirestMessage(@PathVariable String routingkey,
                              @RequestBody CustomerRecord customerRecord) {

        amqpTemplate.convertAndSend(RabbitMQDirectConfig.DIRECT_OBJECT_EXCHANGE, "", customerRecord);
        return "Message Sent";
    }

    @GetMapping("/topicMsg/{routingkey}")
    public String sendTopicMessage(@PathVariable String routingkey,
                              @RequestParam("message") String message) {
        amqpTemplate.convertAndSend(RabbitMQTopicConfig.TOPIC_EXCHANGE, routingkey, message);
        return "Message TOPIC Sent";
    }

    @GetMapping("/fanoutMsg/{routingkey}")
    public String sendfanoutMessage(@PathVariable String routingkey,
                                   @RequestParam("message") String message) {
        amqpTemplate.convertAndSend(RabbitFantOutConfig.FANOUT_EXCHANGE, routingkey, message);
        return "Message Fanout Sent";
    }
}
```
{% endtab %}

{% tab title="RabbitMQConsumer " %}
```java
import ko.co.abacus.acube.rabbitmq.dto.CustomerRecord;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import static ko.co.abacus.acube.rabbitmq.config.AmqpObjectMapperConvertConfig.AMQP_OBJECT_MAPPER_CONVERT;
import static ko.co.abacus.acube.rabbitmq.config.RabbitMQDirectConfig.DIRECT_QUEUE_OBJECT;

@Component
@RabbitListener(queues = DIRECT_QUEUE_OBJECT,
        messageConverter = AMQP_OBJECT_MAPPER_CONVERT)
@Slf4j
public class RabbitMQConsumer {

    @RabbitHandler
    public void consumer(CustomerRecord customerRecord) {
        log.info(DIRECT_QUEUE_OBJECT + " :: Consuming Message : " + customerRecord.getInfo());
    }
}
```
{% endtab %}

{% tab title="MessageListener " %}
```java
@Service
public class MessageListener {

    @RabbitListener(queues = DIRECT_QUEUE_EMAIL)
    public void receiveDirectEmailMessage(String message) {
        System.out.println("My-Direct-Email-Q Received Message:" + message);
        System.out.println();
    }

    @RabbitListener(queues = DIRECT_QUEUE_SMS)
    public void receiveDiretSmsMessage(String message) {
        System.out.println("Received Message:" + message);
        System.out.println();
    }

    @RabbitListener(queues = DIRECT_QUEUE_SNS)
    public void receiveDiretSnsMessage(String message) {
        System.out.println("y-Direct-Sns-Q Received Message:" + message);
        System.out.println();
    }

    @RabbitListener(queues = TOPIC_QUEUE_EMAIL)
    public void receiveTopicEmailMessage(CustomerRecord customerRecord) {
        System.out.println("My-Topic-Email-Q Received Message:" + customerRecord.getInfo());
        System.out.println();
    }

    @RabbitListener(queues = TOPIC_QUEUE_SMS)
    public void receiveTopicSmsMessage(String message) {
        System.out.println("My-Topic-Sms-Q Received Message:" + message);
        System.out.println();
    }

    @RabbitListener(queues = TOPIC_QUEUE_SNS)
    public void receiveTopicSnsMessage(String message) {
        System.out.println("My-Topic-Sns-Q Received Message:" + message);
        System.out.println();
    }

    @RabbitListener(queues = TOPIC_QUEUE_EMPTY)
    public void receiveTopiceMPTYMessage(String message) {
        System.out.println("My-Topic-Empty-Q Received Message:" + message);
        System.out.println();
    }

    @RabbitListener(queues = FANOUT_QUEUE_EMAIL)
    public void receiveFanoutEmailMessage(String message) {
        System.out.println("My-Fanout-Email-Q Received Message:" + message);
        System.out.println();
    }

    @RabbitListener(queues = FANOUT_QUEUE_SMS)
    public void receiveFanoutSmsMessage(String message) {
        System.out.println("My-Fanout-Sms-Q Received Message:" + message);
        System.out.println();
    }

    @RabbitListener(queues = FANOUT_QUEUE_SNS)
    public void receiveFanoutSnsMessage(String message) {
        System.out.println("My-Fanout-Sns-Q Received Message:" + message);
        System.out.println();
    }

    @RabbitListener(queues = DIRECT_QUEUE_OBJECT,
            messageConverter = AMQP_OBJECT_MAPPER_CONVERT)
    public void secondQueueListener(CustomerRecord customerRecord) {
        System.out.printf("Message From " + DIRECT_QUEUE_OBJECT + " => %s%n", customerRecord.getInfo());
    }
}
```
{% endtab %}
{% endtabs %}

## 6. 사례

메시징 대기열에는 다음과 같은 많은 사용 사례가 있습니다.

* **분리(Decoupling)**: 서로 다른 서비스 또는 응용 프로그램 간의 통신을 분리하는 방법입니다.
* **높은 응답 시간(High Response Time)**: 계산, 검색 또는 PDF 작성과 같이 요청의 응답 시간이 너무 긴 경우입니다.
* **백그라운드 작업(Background Jobs)**: 많은 사용자에게 백그라운드 메시지, 이메일 또는 알림을 보냅니다.
* **비동기 메시징(Asynchronous Messaging)**: 메시징 큐는 비동기 프로그래밍을 구현하는 가장 좋은 방법입니다."
