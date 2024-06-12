---
description: Java Message Service
---

# JMS

## 1. JMS&#x20;

**JMS는 Java Message Service를 의미하며, Java 메시징 미들웨어 서버에 접근할 수 있도록 표준 JAVA API를 제공**합니다. 이는 애플리케이션과 서비스 간의 데이터 교환을 위해 비동기적이고 안정적인 통신을 가능하게 하는 전통적인 서비스 지향 아키텍처뿐만 아니라 현재 마이크로서비스 아키텍처에서도 널리 사용되고 있습니다.

메시징 시스템의 대표적인 제품으로는 ActiveMQ, RabbitMQ, ActiveMQ Artemis, Apache Kafka, Application Servers (Glassfish, WebsphereMQ)등이 있습니다.

JMS는 자바 또는 JVM 기반 애플리케이션 전용이며, 다른 언어로 작성된 애플리케이션과의 **상호 운용이 불가능**합니다. 따라서 더 나은 상호 운용성을 지원하는 메시징 프로토콜(예: NodeJS, Python, C# 등)이 필요하다면 AMQP(Advanced Message Queuing Protocol)를 지원하는 RabbitMQ나 Kafka와 같은 제품을 고려해야 합니다.

JMS는 로드 밸런싱과 내결합성에 한계를 가지며, 메시지 저장소가 없어 발생하는 문제점들이 있습니다. 또한, **JMS는 메시지의 무결성이나 개인정보 보호를 제어하거나 구성하는 기능을 제공하지 않기 때문에**, 이러한 기능들은 JMS 제공자들이 제공해야 합니다.

## 2. JMS 구성요소

JMS는 PTP(Point-To-Point)와 Pub/Sub 모델을 제공하며, JMS 응용 프로그램은 다음 요소들로 구성됩니다.

* **JMS 클라이언트** – 메시지를 보내고 받는 Java 코드입니다.
* **JMS가 아닌 클라이언트** – 시스템의 기본 API를 사용합니다.
* **메시지** – 보내거나 받는 비즈니스 데이터입니다.
* **JMS 공급자** –JMS는 관리 기능을 포함하여 메시징 시스템을 구현합니다. 이는 메시지 지향 미들웨어(MOM)로도 알려져 있습니다.
* **관리 오브젝트** – 기본적으로 대상(큐, 토픽) 및 JMS 제공자에 사전 구성된 연결 팩토리입니다.

현재(2024년 5월) 메시지 시스템에서는 RabbitMQ나 Kafka와 같은 제품이 널리 사용되고 있습니다. 이 글에서는 윈도우 기반으로 서버를 설치하고, 간단한 예제를 통해 JMS를 학습하는 방법을 소개하고자 합니다.

## 3. 설치

1.  **GlassFish 7.0.14 다운로드:** GlassFish 7.0.14는 Jakarta EE 10 호환 구현체로, Eclipse GlassFish의 최신 릴리스입니다. 이 버전은 MicroProfile Config, REST Client, 그리고 JWT를 주요한 새로운 기능으로 지원합니다. 공식적으로 JDK 11과 JDK 17을 지원하며, JDK 18부터 JDK 21에서도 실행됩니다.\
    \
    참고: [Eclipse GlassFish](https://glassfish.org/)\
    \
    다운로드: [Eclipse GlassFish 7.0.14, Jakarta EE Platform, 10](https://www.eclipse.org/downloads/download.php?file=/ee4j/glassfish/glassfish-7.0.14.zip)\


    <figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>


2.  **압축해제:** 다운로드한 파일을 압축을 풉니다.\


    <figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>


3.  **서버   실행**:  assfish7/bin 폴더에 있는 startserv.bat를 실행 합니다.\


    <figure><img src="../.gitbook/assets/image (5) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>
4.  **Admin Console 열기**: [http://localhost:4848](http://localhost:4848/) 에서 관리 콘솔을 오픈 합니다.\


    <figure><img src="../.gitbook/assets/image (6) (1) (2).png" alt=""><figcaption></figcaption></figure>
5.  JMS 자원 : Resource/JMS Resources/Connection Factories에 Glassfish가 만든 기본 JMS 팩토리를 볼 수 있습니다. ( jms/\_\_defaultConnectionFactory )\


    <figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>
6.  JMS 자원 생성: JMS 리소스가 있어서 메세지를 보내고 받을수 있으므로 리소스를 생성 합니다.

    <figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

    *   **jms/PTPQueue 생성**: JNDI 리소스를 생성 하기 위해 New 를 선택하여 다음과  같이 생성합니다. \


        <figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>


    *   jms/ReplyQueue **생성**: JNDI 리소스를 생성 하기 위해 New 를 선택하여 다음과  같이 생성합니다. \


        <figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>
    *   jms/Topic **성**: JNDI 리소스를 생성 하기 위해 New 를 선택하여 다음과  같이 생성합니다. \


        <figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
    *   생성되 최종 모습 입니다.\


        <figure><img src="../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 4. 예제 코드&#x20;

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

```java
import jakarta.jms.*;

import javax.naming.InitialContext;
import javax.naming.NamingException;

public class JmsPtpQueue {
    public static void main(String[] args)  {
        ConnectionFactory connectionFactory = null;
        Queue queue = null;

        try {
            InitialContext initialContext = new InitialContext();

            //Step-1 Create ConnectionFactory
            connectionFactory
                    = (ConnectionFactory) initialContext.lookup("jms/__defaultConnectionFactory");

            //Step-2
            queue = (Queue) initialContext.lookup("jms/PTPQueue");
        } catch (NamingException e) {
            e.printStackTrace();
        }

        //Step-3
        try (JMSContext jmsContext = connectionFactory.createContext()) {

            //Step-4a
            TextMessage textMessage = jmsContext.createTextMessage("Message using JMS 2.0");
            JMSProducer jmsProducer = jmsContext.createProducer().send(queue, textMessage);

            //Step-4b
            TextMessage message = (TextMessage) jmsContext.createConsumer(queue).receive();
            System.out.println(message.getText());
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }
}
```

