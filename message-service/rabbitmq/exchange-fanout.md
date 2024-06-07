# Exchange-Fanout

라우팅 키와 관계없이 연결된 모든 Queue에 동일한 메시지를 전달합니다.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

* 모든 Queue가 Publisher이 발행한 메시지를 Consumer에서 받습니다.
* Routing Key를 지정해도 발행한 메시지를 Consumer에서 받습니다.

## 1. 관리자 UI

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* **Exchange 이름:** My-Fanout-Exchange:&#x20;
* Binding Queue
  *   My-Fanout-Email-Q: Routing Key (email)로 지정\


      <figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>
  *   My-Fanout-Sms-Q: Routing Key 지정 하지 않음&#x20;

      <figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>
  *   My-Fanout-Sns-Q: Routing Key 지정 하지 않음

      <figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

## 2. 소스 코드

```java
public class FanoutExchange {

    private static Logger logger =  LoggerFactory.getLogger(FanoutExchange.class);

    public static final String FANOUT_EXCHANGE = "My-Fanout-Exchange";
    public static final String FANOUT_QUEUE_EMAIL = "My-Fanout-Email-Q";
    public static final String FANOUT_QUEUE_SMS = "My-Fanout-Sms-Q";
    public static final String FANOUT_QUEUE_SNS = "My-Fanout-Sns-Q";

    public static void run() throws IOException, TimeoutException {
        // Exchang FANOUT로 생성 
        exchangeDeclare();
        //  QUEUE 생성 
        queueDeclare();
        // Exchang + QUEUE Binding
        queueBind();

        Thread consumer = new Thread(() -> {
            try {
                consumMessage();
            } catch (IOException | TimeoutException | InterruptedException e) {
                e.printStackTrace();
            }
        });


        Thread publisher = new Thread(() -> {
            try {
                publishMessage();
            } catch (IOException | TimeoutException | InterruptedException e) {
                e.printStackTrace();
            }
        });

        // 메세지 발송
        publisher.start();
        
        // 메세지 수신
        consumer.start();
    }


    // 1. Exchang FANOUT로 생성 
    private static void exchangeDeclare() throws IOException, TimeoutException {
        Channel channel =  RabbitMqQConnectionManager.getConnection().createChannel();
        channel.exchangeDeclare(FANOUT_EXCHANGE, BuiltinExchangeType.FANOUT, true);
        channel.close();
    }

    // 2. QUEUE 생성 
    private static void queueDeclare() throws IOException, TimeoutException {
        Channel channel =  RabbitMqQConnectionManager.getConnection().createChannel();
        channel.queueDeclare(FANOUT_QUEUE_EMAIL, true, false, false, null);
        channel.queueDeclare(FANOUT_QUEUE_SMS, true, false, false, null);
        channel.queueDeclare(FANOUT_QUEUE_SNS, true, false, false, null);
        channel.close();
    }

    // 3. Exchang에 QUEUE Binding
    private static void queueBind() throws IOException, TimeoutException {
        Channel channel = RabbitMqQConnectionManager.getConnection().createChannel();

        channel.queueBind(FANOUT_QUEUE_EMAIL, FANOUT_EXCHANGE, "email");
        channel.queueBind(FANOUT_QUEUE_SMS, FANOUT_EXCHANGE, "");
        channel.queueBind(FANOUT_QUEUE_SNS, FANOUT_EXCHANGE, "");

        channel.close();
    }

    // 4. 메시지 발송 publish
    private static void publishMessage() throws IOException, TimeoutException, InterruptedException {
        Channel channel = RabbitMqQConnectionManager.getConnection().createChannel();

        // 3. 전송 메세지
        String emai = "이메일 Fanout 전송 ......";
        String sms = "sms Fanout 전송 ......";
        String sns = "sns Fanout 전송 ......";

        // 4. publish  (exchange, routingKey, properties, messageBody)
        channel.basicPublish(FANOUT_EXCHANGE, "email", null, emai.getBytes());
        Thread.sleep(1000);
        channel.basicPublish(FANOUT_EXCHANGE, "", null, sms.getBytes());
        Thread.sleep(1000);
        channel.basicPublish(FANOUT_EXCHANGE, "", null, sns.getBytes());

        channel.close();
    }

    // 5. 메시지 수신 
    private static void consumMessage() throws IOException, TimeoutException, InterruptedException {
        Channel channel = RabbitMqQConnectionManager.getConnection().createChannel();

        channel.basicConsume(FANOUT_QUEUE_EMAIL,
                true,
                ((consumerTag, message) -> {
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(FANOUT_QUEUE_EMAIL + " : " + new String(message.getBody(), "UTF-8"));
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });

        channel.basicConsume(FANOUT_QUEUE_SMS,
                true,
                ((consumerTag, message) -> {
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(FANOUT_QUEUE_SMS + " : " + new String(message.getBody(), "UTF-8"));
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });

        channel.basicConsume(FANOUT_QUEUE_SNS,
                true,
                ((consumerTag, message) -> {
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(FANOUT_QUEUE_SNS + " : " + new String(message.getBody(),"UTF-8"));
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });
    }
}
```

## 3. 결과

<figure><img src="../../.gitbook/assets/image (332).png" alt=""><figcaption></figcaption></figure>
