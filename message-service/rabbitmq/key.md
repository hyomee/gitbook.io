# 정의된 Key 이외정보 라우팅

Exchange에서 받은 메시지를 Queue로 전달할 떄 정의되지 않은 라우팅 키에 대해서는 메세지 유실(정확히 삭제됨)이 발생합니다. 이것을 해결 하기 위해 Fanout을 사용해서 Topic, Exchange To Exchange를 통해서 한 곳에서 확인 할 수 있습니다.

* Topic 인 경우

{% code lineNumbers="true" %}
```java
Channel channel = RabbitMQConnectionManager.getConnection().createChannel();

channel.queueBind(TOPIC_QUEUE_EMAIL, TOPIC_EXCHANGE, "email.*");
channel.queueBind(TOPIC_QUEUE_SMS, TOPIC_EXCHANGE, "#.sms.*");
channel.queueBind(TOPIC_QUEUE_SNS, TOPIC_EXCHANGE, "#.sns");
channel.queueBind(TOPIC_QUEUE_EMPTY, TOPIC_EXCHANGE, "*");

channel.close();

=========================================================================
channel.basicPublish(TOPIC_EXCHANGE, "k", null, empty.getBytes());
```
{% endcode %}

* 6 Line: 패턴에 정의 되지 않은 \*에 대한 라우팅&#x20;
* 11 Line: TOPIC\_QUEUE\_EMPTY로 메세지 인위적으로 보냄&#x20;

