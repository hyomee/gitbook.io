# Exchange-Exchange

Exchanges-Queues 바인딩을 사용하듯이 Exchange 타입이 Direct인 경우는 바인딩 key의해서 Direct 타입의 다른 Exchange로 메시지를 전달 할 수 있습니다.

* 라우팅 키에 의해서&#x20;

**Exchange To Exchange 흐름**\


<figure><img src="../../.gitbook/assets/image (340).png" alt=""><figcaption></figcaption></figure>

* Publisher에서 발송한 메시지를 받는 Exchange(My-First-Exchange)에서 바인딩으로 큐를 지정(queuebind) 하듯이 Exchange를 exchangeBind를 사용하여 Exchange 바인딩(My-Second-Exchange)을 합니다.
* Publisher에서 발송한 메시지의 라우트 키가 바인딩 키와 동일 하면 동일한 My-First-Exchange에 바인딩 된 Queue와 Exchange로 메시지를 전달합니다.
* **라우팅 키가 동일하며 Queue와 My-Second-Exchange 가 동시에 받습니다.**

## 1. 관리자 UI

<figure><img src="../../.gitbook/assets/image (341).png" alt=""><figcaption></figcaption></figure>

* My-First-Exchange에 3개의 Queue와 1개의 Exchanger가 라우팅 키에 의해 바인딩 되어 있습니다.
* My-Second-Exchange는 My-First-Exchange에서 라우팅키가 email, messaheq인 경우 메세지를 전달 받습니다.

## 2. 소스

```java
public class ExchangeExchange {
    private static Logger logger =  LoggerFactory.getLogger(ExchangeExchange.class);
    public static final String FIRST_EXCHANGE = "My-First-Exchange";
    public static final String SECOND_EXCHANGE = "My-Second-Exchange";
    
    public static final String FIRST_QUEUE_EMAIL = "My-Fiest-Email-Q";
    public static final String FIRST_QUEUE_SMS = "My-Fiest-Sms-Q";
    public static final String FIRST_QUEUE_SNS = "My-Fiest-Sns-Q";

    public static final String SECOND_QUEUE_EMAIL = "My-Second-Email-Q";
    public static final String SECOND_QUEUE_MQ = "My-Second-Message-Q";

    public static void run() throws IOException, TimeoutException {
        // 1. Exchange 생성
        exchangeDeclare();

        // 2. Queue 생성
        queueDeclare();

        // 3. Exchange - Queue 바인딩 
        queueBind();

        // 4. Exchange - Exchange 바인딩 
        exchangeBind();

        Thread consumer = new Thread(() -> {
            try {
                // 5. Consumer 소비
                consumMessage();
            } catch (IOException | TimeoutException | InterruptedException e) {
                e.printStackTrace();
            }
        });


        Thread publisher = new Thread(() -> {
            try {
                // 6. Publisher 발행
                publishMessage();
            } catch (IOException | TimeoutException | InterruptedException e) {
                e.printStackTrace();
            }
        });

        publisher.start();
        consumer.start();
    }

    // Exchange 생성
    private static void exchangeDeclare() throws IOException, TimeoutException {
        Channel channel =  RabbitMQConnectionManager.getConnection().createChannel();
        channel.exchangeDeclare(FIRST_EXCHANGE, BuiltinExchangeType.DIRECT, true);
        channel.exchangeDeclare(SECOND_EXCHANGE, BuiltinExchangeType.DIRECT, true);
        channel.close();
    }

    // Queue 생성
    private static void queueDeclare() throws IOException, TimeoutException {
        Channel channel =  RabbitMQConnectionManager.getConnection().createChannel();
        channel.queueDeclare(FIRST_QUEUE_EMAIL, true, false, false, null);
        channel.queueDeclare(FIRST_QUEUE_SMS, true, false, false, null);
        channel.queueDeclare(FIRST_QUEUE_SNS, true, false, false, null);

        channel.queueDeclare(SECOND_QUEUE_EMAIL, true, false, false, null);
        channel.queueDeclare(SECOND_QUEUE_MQ, true, false, false, null);
        channel.close();
    }

    // Exchange - Queue 바인딩
    private static void queueBind() throws IOException, TimeoutException {
        Channel channel = RabbitMQConnectionManager.getConnection().createChannel();

        channel.queueBind(FIRST_QUEUE_EMAIL, FIRST_EXCHANGE, "email");
        channel.queueBind(FIRST_QUEUE_SMS, FIRST_EXCHANGE, "sms");
        channel.queueBind(FIRST_QUEUE_SNS, FIRST_EXCHANGE, "sns");

        channel.queueBind(SECOND_QUEUE_EMAIL, SECOND_EXCHANGE, "email");
        channel.queueBind(SECOND_QUEUE_MQ, SECOND_EXCHANGE, "messageq");

        channel.close();
    }

    // Exchange - Exchange 바인딩
    private static void exchangeBind() throws IOException, TimeoutException {
        Channel channel = RabbitMQConnectionManager.getConnection().createChannel();

        // (destination-exchange, source-exchange , routingKey
        channel.exchangeBind(SECOND_EXCHANGE, FIRST_EXCHANGE, "email");
        channel.exchangeBind(SECOND_EXCHANGE, FIRST_EXCHANGE, "messageq");

        channel.close();
    }

    private static void publishMessage() throws IOException, TimeoutException, InterruptedException {
        Channel channel = RabbitMQConnectionManager.getConnection().createChannel();

        // 3. 전송 메세지
        String emai = "이메일 전송 ......";
        String sms = "sms 전송 ......";
        String sns = "sns 전송 ......";
        String mq = "message 전송 ......";

        // 4. publish  (exchange, routingKey, properties, messageBody)
        channel.basicPublish(FIRST_EXCHANGE, "email", null, emai.getBytes());
        Thread.sleep(1000);
        channel.basicPublish(FIRST_EXCHANGE, "sms", null, sms.getBytes());
        Thread.sleep(1000);
        channel.basicPublish(FIRST_EXCHANGE, "sns", null, sns.getBytes());

        Thread.sleep(1000);
        channel.basicPublish(FIRST_EXCHANGE, "messageq", null, mq.getBytes());

        channel.close();
    }

    private static void consumMessage() throws IOException, TimeoutException, InterruptedException {
        Channel channel = RabbitMQConnectionManager.getConnection().createChannel();

        channel.basicConsume(FIRST_QUEUE_EMAIL,
                true,
                ((consumerTag, message) -> {
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(FIRST_QUEUE_EMAIL + " : " + new String(message.getBody(), "UTF-8"));
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });

        channel.basicConsume(FIRST_QUEUE_SMS,
                true,
                ((consumerTag, message) -> {
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(FIRST_QUEUE_SMS + " : " + new String(message.getBody(), "UTF-8"));
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });

        channel.basicConsume(FIRST_QUEUE_SNS,
                true,
                ((consumerTag, message) -> {
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(FIRST_QUEUE_SNS + " : " + new String(message.getBody(),"UTF-8"));
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });


        channel.basicConsume(SECOND_QUEUE_EMAIL,
                true,
                ((consumerTag, message) -> {
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(SECOND_QUEUE_EMAIL + " : " + new String(message.getBody(), "UTF-8"));
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });

        channel.basicConsume(SECOND_QUEUE_MQ,
                true,
                ((consumerTag, message) -> {
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(SECOND_QUEUE_MQ + " : " + new String(message.getBody(), "UTF-8"));
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });
    }
}
```

## 3. 결과

<figure><img src="../../.gitbook/assets/image (342).png" alt=""><figcaption></figcaption></figure>

* email 라우팅키에 대해서 My-First-Email-Q(My-First-Exchange)와 My-Second-Email-Q(My-Second-Exchange)에서 메시지를 수신 받습니다.
* messageq라우팅키에 대해서 My-Second-Message-Q(My-Second-Exchange)에서 메시지를 수신 받습니다.
