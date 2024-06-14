# Exchange-Header

**메시지 헤더를 통해 binding key를 사용하는 것보다 더 다양한 속성을 사용할 수 있으며 헤더 값이 바인딩 시 지정된 값과 같은 경우에만 일치하는 것으로 간주합니다.**

* 헤더 교환은 **일치하는 메시지 헤더**에 따라 메시지를 라우팅하며 라우팅 키는 무시합니다.
* 헤더와 같은 JSON을 사용하여 헤더 교환(my-header-exchange)에 바인딩됨

**Topic  Exchange 흐름**

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 허용 되는 해더(x-match)에는 any, all 두가지 유형이 있습니다.
  * any: Exchange로 보내는 메시지에 Queue가 연결된 헤더 중 하나 이상이 포함되어야 함을 의미
  * all: Exchange로 보내는 메시지에 Queue가 연결된 모든헤더 가 같아야 함을 의미

## 1. 관리자 UI

*   **Exchange 이름:** My-Header-Exchange \


    <figure><img src="../../.gitbook/assets/image (335).png" alt=""><figcaption></figcaption></figure>
* Binding Queue
  *   My-Header-Email-Q: Arguments\
      \- x-match 가 any 이므로 type1, type2 의 값이 둘 중에 하나라도 맞으면 큐에 전당

      <table data-header-hidden><thead><tr><th width="157">키</th><th>값</th></tr></thead><tbody><tr><td>type1:</td><td>email</td></tr><tr><td>type2:</td><td>mail</td></tr><tr><td>x-match:</td><td>any</td></tr></tbody></table>

      <figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
  *   My-Header-Sms-Q: Arguments\
      \- x-match 가 all이므로 type1, type2 의 값이 모두 맞아야 전달&#x20;

      | type1:   | email |
      | -------- | ----- |
      | type2:   | sms   |
      | x-match: | all   |

      <figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>
  *   My-Header-Sns-Q: Arguments\
      \- x-match 가 any 이므로 type1, type2 의 값이 둘 중에 하나라도 맞으면 큐에 전당

      | type1:   | email |
      | -------- | ----- |
      | type2:   | sns   |
      | x-match: | any   |

      <figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 2. 소스

```java
public class HeaderExchange {

    private static Logger logger =  LoggerFactory.getLogger(HeaderExchange.class);

    public static final String HEADER_EXCHANGE = "My-Header-Exchange";
    public static final String HEADER_QUEUE_EMAIL = "My-Header-Email-Q";
    public static final String HEADER_QUEUE_SMS = "My-Header-Sms-Q";
    public static final String HEADER_QUEUE_SNS = "My-Header-Sns-Q";

    public static void run() throws IOException, TimeoutException {
        // Exchang Header로 생성
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
        Channel channel =  RabbitMQConnectionManager.getConnection().createChannel();
        channel.exchangeDeclare(HEADER_EXCHANGE, BuiltinExchangeType.HEADERS, true);
        channel.close();
    }

    // 2. QUEUE 생성
    private static void queueDeclare() throws IOException, TimeoutException {
        Channel channel =  RabbitMQConnectionManager.getConnection().createChannel();
        channel.queueDeclare(HEADER_QUEUE_EMAIL, true, false, false, null);
        channel.queueDeclare(HEADER_QUEUE_SMS, true, false, false, null);
        channel.queueDeclare(HEADER_QUEUE_SNS, true, false, false, null);
        channel.close();
    }

    // 3. Exchang에 QUEUE Binding
    private static void queueBind() throws IOException, TimeoutException {
        Channel channel = RabbitMQConnectionManager.getConnection().createChannel();

        Map<String, Object> emailArgs = new HashMap<>();
        emailArgs.put("x-match", "any");
        emailArgs.put("type1", "email");
        emailArgs.put("type2", "mail");
        channel.queueBind(HEADER_QUEUE_EMAIL, HEADER_EXCHANGE, "", emailArgs);

        Map<String, Object> smsArgs = new HashMap<>();
        smsArgs.put("x-match", "all");
        smsArgs.put("type1", "email");
        smsArgs.put("type2", "sms");
        channel.queueBind(HEADER_QUEUE_SMS, HEADER_EXCHANGE, "", smsArgs);

        Map<String, Object> snsArgs = new HashMap<>();
        snsArgs.put("x-match", "any");
        snsArgs.put("type1", "email");
        snsArgs.put("type2", "sns");
        channel.queueBind(HEADER_QUEUE_SNS, HEADER_EXCHANGE, "", snsArgs);

        channel.close();
    }

    // 4. 메시지 발송 publish
    private static void publishMessage() throws IOException, TimeoutException, InterruptedException {
        Channel channel = RabbitMQConnectionManager.getConnection().createChannel();

        // 3. 전송 메세지
        String emai = "이메일  전송 ......";
        String sms = "sms 전송 ......";
        String sns = "sns 전송 ......";

        logger.info("이메일 basicPublish / email mail =======================================");
        Map<String, Object> emailMap = new HashMap<>();
        emailMap.put("type1", "email");
        emailMap.put("type2", "mail");
        AMQP.BasicProperties emailProperties = new AMQP.BasicProperties()
                .builder().headers(emailMap).build();
        // 4. publish  (exchange, routingKey, properties, messageBody)
        channel.basicPublish(HEADER_EXCHANGE, "", emailProperties, emai.getBytes());

        Thread.sleep(1000);
        logger.info("sms basicPublish / email sms =======================================");
        Map<String, Object> smsMap = new HashMap<>();
        smsMap.put("type1", "email");
        smsMap.put("type2", "sms");
        AMQP.BasicProperties smsProperties = new AMQP.BasicProperties()
                .builder().headers(smsMap).build();

        channel.basicPublish(HEADER_EXCHANGE, "", smsProperties, sms.getBytes());
        Thread.sleep(1000);

        logger.info("sns basicPublish / email sns=======================================");
        Map<String, Object> snsMap = new HashMap<>();
        snsMap.put("type1", "email");
        snsMap.put("type2", "sns");
        AMQP.BasicProperties snsProperties = new AMQP.BasicProperties()
                .builder().headers(snsMap).build();
        channel.basicPublish(HEADER_EXCHANGE, "", snsProperties, sns.getBytes());

        channel.close();
    }

    // 5. 메시지 수신
    private static void consumMessage() throws IOException, TimeoutException, InterruptedException {
        Channel channel = RabbitMQConnectionManager.getConnection().createChannel();

        channel.basicConsume(HEADER_QUEUE_EMAIL,
                true,
                ((consumerTag, message) -> {
                    logger.info("any, email, mail");
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(HEADER_QUEUE_EMAIL + " : " + new String(message.getBody(), "UTF-8"));
                    logger.info(message.getEnvelope().toString());
                    logger.info("=======================================");
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });

        channel.basicConsume(HEADER_QUEUE_SMS,
                true,
                ((consumerTag, message) -> {
                    logger.info("all, email, sms");
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(HEADER_QUEUE_SMS + " : " + new String(message.getBody(), "UTF-8"));
                    logger.info(message.getEnvelope().toString());
                    logger.info("=======================================");
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });

        channel.basicConsume(HEADER_QUEUE_SNS,
                true,
                ((consumerTag, message) -> {
                    logger.info("any, email, sns");
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(HEADER_QUEUE_SNS + " : " + new String(message.getBody(),"UTF-8"));
                    logger.info(message.getEnvelope().toString());
                    logger.info("=======================================");
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });
    }
}
```

## 3. 결과

```
이메일 basicPublish / email mail =======================================
any, email, mail
consumerTag : amq.ctag-RWSncdyxA1UT2IFW4w_JBw
My-Header-Email-Q : 이메일  전송 ......
Envelope(deliveryTag=1, redeliver=false, exchange=My-Header-Exchange, routingKey=)
=======================================
any, email, sns
consumerTag : amq.ctag-xralDIfp65DLw-pJ7hP5Kw
My-Header-Sns-Q : 이메일  전송 ......
Envelope(deliveryTag=2, redeliver=false, exchange=My-Header-Exchange, routingKey=)
=======================================


sms basicPublish / email sms =======================================
any, email, sns
consumerTag : amq.ctag-xralDIfp65DLw-pJ7hP5Kw
My-Header-Sns-Q : sms 전송 ......
Envelope(deliveryTag=3, redeliver=false, exchange=My-Header-Exchange, routingKey=)
=======================================
all, email, sms
consumerTag : amq.ctag-irwUj_503zk9izcjyqRrrw
My-Header-Sms-Q : sms 전송 ......
Envelope(deliveryTag=4, redeliver=false, exchange=My-Header-Exchange, routingKey=)
=======================================
any, email, mail
consumerTag : amq.ctag-RWSncdyxA1UT2IFW4w_JBw
My-Header-Email-Q : sms 전송 ......
Envelope(deliveryTag=5, redeliver=false, exchange=My-Header-Exchange, routingKey=)
=======================================


sns basicPublish / email sns=======================================
any, email, sns
consumerTag : amq.ctag-xralDIfp65DLw-pJ7hP5Kw
My-Header-Sns-Q : sns 전송 ......
Envelope(deliveryTag=6, redeliver=false, exchange=My-Header-Exchange, routingKey=)
=======================================
any, email, mail
consumerTag : amq.ctag-RWSncdyxA1UT2IFW4w_JBw
My-Header-Email-Q : sns 전송 ......
Envelope(deliveryTag=7, redeliver=false, exchange=My-Header-Exchange, routingKey=)
=======================================

```
