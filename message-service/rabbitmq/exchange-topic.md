# Exchange-Topic

**라우팅 키가 일치하는 Queue로 메시지를 전달하는 방식으로 라우팅 키는 점(.)으로 구분된 단어를 조합해서 정의하며, 와일드 카드(\*)와(#)을 사용하여 패턴을 지정할 수 있습니다**

* 라우팅 키는 점으로 구분된 0개 이상의 단어로 구성되어야 합니다.
* 라우팅 패턴은 , **`.`** 및 **`#`**만 허용되는 정규식과 같습니다.
  * 기호 별(**\***)은 정확히 한 단어가 허용됨을 의미
  * 기호 해시(**#**)는 허용되는 단어 수가 0개 이상임을 의미
  * 기호 점(**.**)은 단어 구분 기호를 의미(여러 키 용어는 점 구분 기호로 구분)
  *   라우팅키 패턴:\


      | 턴                                        | 유효한 라우팅 키                                                | 잘못된 라우팅 키                                                      |
      | ---------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------- |
      | email**.\*** – 첫 번째 단어 다음에 한 단어가 옵니다.    | <p>email.naver,<br>email.iabacus,<br>email.daum</p>      | <p>email, <br>email.naver.anything, email.comnpany.iabacus</p> |
      | **#.sms.\*** – 0개 이상의 단어, 그 뒤에 정확히 한 단어. | <p>sms.featurephone<br>sms.sms.sms,</p><p>sm.sms<br></p> | <p>sms,<br>company.sms,<br>anything.sms.anything.xyz</p>       |
      | **#.sns**– 0개 이상의 단어 뒤에 단어가 옵니다.         | <p>kakao.sns,</p><p>sns.sns,<br>sns</p>                  | <p>sns.company,<br>anything.sns.anything</p>                   |

      \

*   Topic  Exchange 흐\


    <figure><img src="../../.gitbook/assets/image (334).png" alt=""><figcaption></figcaption></figure>

    * 큐는 라우팅 키 패턴을 사용하여 Exchange애 바인딩 됩니다.
    * Publisher가 메시지를 게시 할 때 라우팅 key를 포함해서 Exchange에 보냅니다.
    * Exchange는  라우팅 키패턴과 라우팅 key가 일치하는 큐로 메시지를 전달합니다.

## 1. 관리자 UI

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* **Exchange 이름:** My-Topic-Exchange&#x20;
* Binding Queue
  *   My-Topic-Email-Q: Routing Key Pattern (email.\*)로 지정\


      <figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
  *   My-Topic-Sms-Q: Routing Key Pattern (#.sms.\*)로 지정\


      <figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
  *   My-Topic-Sns-Q: Routing Key Pattern (#.sns)로 지정\


      <figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## 2. Topic 소스

```java
public class TopicExchange {

    private static Logger logger =  LoggerFactory.getLogger(TopicExchange.class);

    public static final String TOPIC_EXCHANGE = "My-Topic-Exchange";
    public static final String TOPIC_QUEUE_EMAIL = "My-Topic-Email-Q";
    public static final String TOPIC_QUEUE_SMS = "My-Topic-Sms-Q";
    public static final String TOPIC_QUEUE_SNS = "My-Topic-Sns-Q";

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


    // 1. Exchang TOPIC로 생성
    private static void exchangeDeclare() throws IOException, TimeoutException {
        Channel channel =  RabbitMQConnectionManager.getConnection().createChannel();
        channel.exchangeDeclare(TOPIC_EXCHANGE, BuiltinExchangeType.TOPIC, true);
        channel.close();
    }

    // 2. QUEUE 생성
    private static void queueDeclare() throws IOException, TimeoutException {
        Channel channel =  RabbitMQConnectionManager.getConnection().createChannel();
        channel.queueDeclare(TOPIC_QUEUE_EMAIL, true, false, false, null);
        channel.queueDeclare(TOPIC_QUEUE_SMS, true, false, false, null);
        channel.queueDeclare(TOPIC_QUEUE_SNS, true, false, false, null);
        channel.close();
    }

    // 3. Exchang에 QUEUE Binding
    private static void queueBind() throws IOException, TimeoutException {
        Channel channel = RabbitMQConnectionManager.getConnection().createChannel();

        channel.queueBind(TOPIC_QUEUE_EMAIL, TOPIC_EXCHANGE, "email.*");
        channel.queueBind(TOPIC_QUEUE_SMS, TOPIC_EXCHANGE, "#.sms.*");
        channel.queueBind(TOPIC_QUEUE_SNS, TOPIC_EXCHANGE, "#.sns");

        channel.close();
    }

    // 4. 메시지 발송 publish
    private static void publishMessage() throws IOException, TimeoutException, InterruptedException {
        Channel channel = RabbitMQConnectionManager.getConnection().createChannel();

        // 3. 전송 메세지
        String emai = "이메일 email.iabacus Topic 전송 ......";
        String sms = "sms sms.sms.sms Topic 전송 ......";
        String sns = "sns kakao.sns Topic 전송 ......";

        // 4. publish  (exchange, routingKey, properties, messageBody)
        channel.basicPublish(TOPIC_EXCHANGE, "email.iabacus,\n", null, emai.getBytes());
        Thread.sleep(1000);
        channel.basicPublish(TOPIC_EXCHANGE, "sms.sms.sms", null, sms.getBytes());
        Thread.sleep(1000);
        channel.basicPublish(TOPIC_EXCHANGE, "kakao.sns", null, sns.getBytes());

        channel.close();
    }

    // 5. 메시지 수신
    private static void consumMessage() throws IOException, TimeoutException, InterruptedException {
        Channel channel = RabbitMQConnectionManager.getConnection().createChannel();

        channel.basicConsume(TOPIC_QUEUE_EMAIL,
                true,
                ((consumerTag, message) -> {
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(TOPIC_QUEUE_EMAIL + " : " + new String(message.getBody(), "UTF-8"));
                    logger.info(message.getEnvelope().toString());
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });

        channel.basicConsume(TOPIC_QUEUE_SMS,
                true,
                ((consumerTag, message) -> {
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(TOPIC_QUEUE_SMS + " : " + new String(message.getBody(), "UTF-8"));
                    logger.info(message.getEnvelope().toString());
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });

        channel.basicConsume(TOPIC_QUEUE_SNS,
                true,
                ((consumerTag, message) -> {
                    logger.info("consumerTag : " + consumerTag);
                    logger.info(TOPIC_QUEUE_SNS + " : " + new String(message.getBody(),"UTF-8"));
                    logger.info(message.getEnvelope().toString());
                }),
                consumerTag -> {
                    logger.info(consumerTag);
                });
    }
}
```

## 3. 결과

```
- consumerTag : amq.ctag--47cQoscKLFhW6-cfULImA 
- My-Topic-Email-Q : 이메일 email.iabacus Topic 전송 ...... 
- Envelope(deliveryTag=1, redeliver=false, exchange=My-Topic-Exchange, routingKey=email.iabacus, ) 

- consumerTag : amq.ctag-bk5DJ4sIq_R0F6rsVl8u2w
- My-Topic-Sms-Q : sms sms.sms.sms Topic 전송 ...... 
- Envelope(deliveryTag=2, redeliver=false, exchange=My-Topic-Exchange, routingKey=sms.sms.sms) 

- consumerTag : amq.ctag-T8g3log9vwtUDInYn-khGw 
- My-Topic-Sns-Q : sns kakao.sns Topic 전송 ...... 
- Envelope(deliveryTag=3, redeliver=false, exchange=My-Topic-Exchange, routingKey=kakao.sns)
```

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
