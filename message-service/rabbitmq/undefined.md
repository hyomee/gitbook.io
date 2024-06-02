# 설치

**RabbitMQ** 서버를 설치하는 가이드를 바탕으로 설치 진행을 합니다.

참고: [Debian 및 Ubuntu에 설치](https://www.rabbitmq.com/docs/install-debian#package-dependencies)

## 1. 필수 종속 설치

```sh
sudo apt-get update -y

sudo apt-get install curl gnupg -y
```

## 2. Enable apt HTTPS Transport

apt가 Cloudsmith.io 미러 또는 런치 패드에서 RabbitMQ 및 Erlang 패키지를 다운로드 할 수 있도록하려면 패키지를 설치해야 합니다

```sh
sudo apt-get install apt-transport-https
```

## 3. 리포지토리 서명 키 추가

모든 제 3 자 apt 저장소와 마찬가지로 RabbitMQ 및 Erlang 패키지 저장소를 설명하는 파일 디렉터리 아래에 배치해야 하는데   권장 위치 /etc/apt/sources.list.d/, /etc/apt/sources.list.d/rabbitmq.list 입니다.

```sh
## Team RabbitMQ's main signing key
curl -1sLf "https://keys.openpgp.org/vks/v1/by-fingerprint/0A9AF2115F4687BD29803A206B73A36E6026DFCA" 
sudo gpg --dearmor 
sudo tee /usr/share/keyrings/com.rabbitmq.team.gpg > /dev/null

## Community mirror of Cloudsmith: modern Erlang repository
curl -1sLf https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key
sudo gpg --dearmor 
sudo tee /usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg > /dev/null

## Community mirror of Cloudsmith: RabbitMQ repository
curl -1sLf https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key 
sudo gpg --dearmor 
sudo tee /usr/share/keyrings/rabbitmq.9F4587F226208342.gpg > /dev/null
```

apt 리포지토리를 소스 목록 디렉터리(아래 )에 추가하려면 다음을 사용합니다 (/etc/apt/sources.list.d)

| Release   | Distribution |
| --------- | ------------ |
| 우분투 23.04 | `jammy`      |
| 우분투 22.04 | `jammy`      |
| 우분투 20.04 | `focal`      |
| 우분투 18.04 | `bionic`     |
| 데비안 책벌레   | `bullseye`   |
| 데비안 불스아이  | `bullseye`   |
| 데비안 시드    | `bullseye`   |

$distribution 에 부분에 서버 버전에 따른 Distribution를 변경 해야 합니다.

```sh
sudo tee /etc/apt/sources.list.d/rabbitmq.list <<EOF
## Provides modern Erlang/OTP releases from a Cloudsmith mirror
##
deb [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu $distribution main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu $distribution main

# another mirror for redundancy
deb [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu $distribution main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu $distribution main

## Provides RabbitMQ from a Cloudsmith mirror
##
deb [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu $distribution main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu $distribution main

# another mirror for redundancy
deb [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu $distribution main
deb-src [signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa2.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu $distribution main
EOF
```

## 4. Erlang Packages, RabbitMQ Server 설치하기 <a href="#erlang-packages-rabbitmq-server" id="erlang-packages-rabbitmq-server"></a>

```sh
sudo apt-get update
sudo apt-get install rabbitmq-server
```

## 5. RabbitMQ Server 시작 <a href="#erlang-packages-rabbitmq-server" id="erlang-packages-rabbitmq-server"></a>

```sh
service rabbitmq-server start
```

## 6. Management UI 플러그인 활성화

RabbitMQ는 Management 라는 UI 관리 도구를 사용하기 위해서는 [RabbitMQ Plugin](https://www.rabbitmq.com/docs/management)을 활성화 시켜야 한다.

```sh
rabbitmq-plugins enable rabbitmq_management
```

## 7. Management UI 사용자 추가

Tag 종류는 공식 가이드( [Management Plugin - Access and Permissions](https://www.rabbitmq.com/management.html#permissions) )에서 더 확인 할 수 있고, 아래에서는 관리자(‘administrator’) 권한을 사용 할 수 있다.

```sh
# 사용자 목록
$ sudo rabbitmqctl list_users
    
# 사용자 추가
$ sudo rabbitmqctl add_user hong 'hongrabbit'
    
# 사용자 태그 추가
$ sudo rabbitmqctl set_user_tags hong administrator
```

## 8. 포트 설정

방화벽을 사용하고 있다면 아래 포트를 열어줘야 한다.

* 4369: epmd , 여러 rabbitmq 서버끼리 서로를 찾을 수 있는 네임 서버 역할을 하는 데몬에서 사용
* 5672, 5671: AMQP 를 사용한 메시지 전달
* 25672: inter-node 와 CLI Tool 연결
* 15672: HTTP API, Management UI

## 9. Virtual Hosts

connection, exchange, queue, binding, user, policy 들을 **virtual hosts** 를 통해 논리적인 그룹으로 분리해서 운영할 수 있습니다. 설정은 [이곳](https://www.rabbitmq.com/docs/vhosts)에서 확인 할 수 있습니다,

## 10. 로그&#x20;

* 위치: /var/log/rabbitmq
* 로그 파일 로테이션을 위해 `logrotate` 를 사용해서 변경 할 수 있습니다.
* 설정: `/etc/logrotate.d/rabbitmq-server` 에서 확인하고 변경할 수 있습니다.

```
/var/log/rabbitmq/*.log {
        daily
        missingok
        compress
        delaycompress
        notifempty
}
```

## 11. 관리

* rabbitmq 상태:  sudo rabbitmq-diagnostics status
* rabbitmq  환경 구성: [https://www.rabbitmq.com/docs/configure](https://www.rabbitmq.com/docs/configure)

## 12. 예제

자바에서 RabbitMQ Client(Producer, Consumer)에 대한 코드를 작성하기 위해서 다음 절차로 코드를 작성합니다,

1. Maven 의존성 추가
2. Producer 코드 작성&#x20;
3. Consumer 코드 작성

### 12-1. Maven 의존성&#x20;

* pom.xml 에 의존성을 추가 합니다.

```xml
<dependency>
  <groupId>com.rabbitmq</groupId>
  <artifactId>amqp-client</artifactId>
  <version>5.21.0</version>
</dependency>


<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>2.0.13</version>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-simple</artifactId>
  <version>2.0.13</version>
</dependency>
```

일반적인 일반적인 TCP/IP 코드는 다음과 같습니다.

* 서버&#x20;
  1. 소켓을 생성(socket)하고 포트 번호를 설정 (.bind)
  2. 메세지를 받기위해설정된 소켓의 리스너를 등록 (.listen)
  3. 연결 수락을 하여 메세지를 받아들인다.(.accept)
* 클라이언트
  1. 소켓을 생성(socket)하고 서버에 연결을 요청(.connect)
  2. 연결된 소켓에 데이터 보내기 (.sendall)
  3. 연결된 소켓에 데이터 수신(.recv)

RabbitMQ 서버에 데이터를 전송하고 받기 위한 코드도 기본적인 TCP/IP 통신을 위한 코드와 크게 차이가 없습니다. APQP의 구현체인 RabbitMQ와 데이터 수신/발신을 위한 코드 작성에 대해서 알아보고자 합니다.

### 12-2. Producer 코드 작성

RabbitMQ 서버와 연결하기 위해 ConnectionFactory를 사용해서 연결하고 메세지 발송을 위해 basicPublish 메서드를 사용합니다.itMQ에 연결하는 코드를 작성합니다.

{% code lineNumbers="true" %}
```java
public class CommonConfigs {
    public static final String DEFAULT_QUEUE = "My-Queue-03";
    public static final String RABBITMQ_HOST = "x.x.x.x";
    public static final int RABBITMQ_PORT = 0000;
    public static final String RABBITMQ_ID = "id";
    public static final String RABBITMQ_PWD = "pwd"; 

    public static Connection getRabittMQConnection() throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();

        factory.setHost(RABBITMQ_HOST);
        factory.setPort(RABBITMQ_PORT);
        factory.setUsername(RABBITMQ_ID);
        factory.setPassword(RABBITMQ_PWD);
        Connection connection = factory.newConnection();  

        return connection;
    }
}
```
{% endcode %}

* 9 \~ 17 Line: ConnectionFactory를 사용하여 서버에 연결하는 코드로 호스트, 포트, 아이디, 비밀번호를 설정하고 15line 처럼 커넥션을 요청하여 Connection 객체를 생성합니다.

{% code lineNumbers="true" %}
```java
public class MessagePublisher {
    private final static Logger logger = LoggerFactory.getLogger(MessagePublisher.class);

    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        logger.info("RabbitMQ Queue Publisher Start [" + CommonConfigs.DEFAULT_QUEUE + "]");
        
        Connection connection = CommonConfigs.getRabittMQConnection();
        Channel channel = connection.createChannel();
        
        for (int i = 0; i < 4; i++) {
            String message = "Getting started with rabbitMQ - Msg :: " + i;
            Thread.sleep(1000);
            // channel.queueDeclare( CommonConfigs.DEFAULT_QUEUE, true, false, false, null);
            channel.basicPublish("", CommonConfigs.DEFAULT_QUEUE,null, message.getBytes());
        }
        
        channel.close();
        connection.close();
    }
}
```
{% endcode %}

* 7 Line: RabbitMQ 서버에 연결하기 위해 Connection 객체을 생성 합니다.
* 8 Line: 연결된 객체(Connection)에 채널을 생성 합니다.
* 10 \~ 15 Line: 메세지를 생성 하고 생성한 채널로 basicPublish를 사용하여 데이터를 전송 합니다.
  * **basicPublish 메서드의 각 파라미터의 의미**
    * **var1 (String)**: 메시지를 보낼 교환의 이름입니다. 교환은 메시지를 받아서 큐로 전달하는 역할을 합니다. 라우팅 키(routing key)와 함께 교환에 메시지를 보냅니다.&#x20;
    * **var2 (String)**: 메시지를 전달할 큐의 라우팅 키입니다. 라우팅 키는 큐를 식별하는 역할을 합니다. 해당 큐로 메시지가 전달됩니다.&#x20;
    * v**ar3 (AMQP.BasicProperties)**: 메시지의 기타 속성을 설정하는 객체입니다. 예를 들어, 메시지의 우선순위, 타임스탬프, 헤더 등을 설정할 수 있습니다.
    * &#x20;**var4 (byte\[])**: 전달할 메시지의 내용을 바이트 배열로 표현한 값입니다. 메시지의 실제 내용이 여기에 포함됩니다.
* 17 \~ 18 Line: 메세지 사용이 끝났으면 채널과 커넥션 객체를 닫아서 리소스 자원을 정리 합니다. 만약 자원 정리를 하지 않으면 실무에서는 메모리 누수  및포트 충돌이 발생하고, 다른 프로세스의 동작에 영향을 줄 수는 등 문제가 발생합니다.

<details>

<summary>소켓 프로그램에서 리소스를 닫지 않으면 다양한 문제</summary>

1. **메모리 누수 (Memory Leaks):**
   * 소켓을 닫지 않으면 해당 소켓이 사용하는 메모리가 해제되지 않습니다.
   * 이로 인해 메모리 누수가 발생하며, 시스템 자원이 점차 고갈될 수 있습니다.
2. **파일 디스크립터 누수:**
   * 소켓은 파일 디스크립터로 표현됩니다.
   * 소켓을 닫지 않으면 해당 파일 디스크립터가 계속해서 열려 있게 됩니다.
   * 이로 인해 시스템 리소스가 부족해지고, 파일 디스크립터 누수가 발생할 수 있습니다.
3. **네트워크 연결 문제:**
   * 소켓을 닫지 않으면 서버와 클라이언트 간의 연결이 계속 유지됩니다.
   * 이로 인해 서버 리소스가 과도하게 사용되거나, 클라이언트가 불필요한 연결을 유지하게 됩니다.
4. **다른 프로세스에 영향:**
   * 소켓을 닫지 않으면 다른 프로세스가 해당 포트를 사용할 수 없게 됩니다.
   * 이로 인해 포트 충돌이 발생하고, 다른 프로세스의 동작에 영향을 줄 수 있습니다.

</details>

### 12-3.  Consumer 코드 작성

RabbitMQ 서버와 연결하여 데이터를 받아오기 위해서는 basicConsume를 메서드를 사용합니다.

<figure><img src="../../.gitbook/assets/image (314).png" alt=""><figcaption><p>메세지 수신 모델</p></figcaption></figure>

{% code lineNumbers="true" %}
```java
public class MessageSubscriber {
    private final static Logger logger = LoggerFactory.getLogger(MessageSubscriber.class);

    public static void main(String[] args) throws IOException, TimeoutException {
        logger.info("RabbitMQ Queue Consumer Start [" + CommonConfigs.DEFAULT_QUEUE + "]");
        
        Connection connection = CommonConfigs.getRabittMQConnection();
        Channel channel = connection.createChannel();
        
        DeliverCallback deliverCallback = (s, delivery) -> {
            System.out.println(new String(delivery.getBody()));
        };
        
        CancelCallback cancelCallback = s -> {
            System.out.println(s);
        };
        
        channel.basicConsume(CommonConfigs.DEFAULT_QUEUE, true, deliverCallback, cancelCallback);
    }
}
```
{% endcode %}

* 7 \~ 8 Line: 메시지를 받아 올 연결 객체와 채널을 생성 합니다.
* 10 Line: RabbitMQ Java 클라이언트에서 메시지 소비에 관련된 콜백 인터페이스로 메시지가 소비자에게 전달될 때 호출되는 콜백입니다.&#x20;
  * 메시지가 큐에서 소비자로 전달될 때 실행되는 로직을 정의할 수 있습니다.
  * 주로 메시지를 처리하고 응용 프로그램에서 필요한 작업을 수행하는 데 사용합니다.
* 11 Line: RabbitMQ Java 클라이언트에서 메시지 소비에 관련된 콜백 인터페이스로 소비자가 취소될 때 호출되는 콜백입니다.&#x20;
  * 소비자가 명시적으로 취소되거나 연결이 끊겼을 때 실행됩니다.
  * 소비자의 취소 상태를 처리하고 필요한 후속 작업을 수행하는 데 사용됩니다.
* 18 Line:  소비자는 브로커에게 메시지를 받기 위해 이 메서드를 호출합니다. 호출 후, 브로커는 basic.deliver를 통해 메시지를 소비자에게 전달하며, 소비자는 메시지를 확인(acknowledge)합니다. 만약 no-ack 옵션이 설정되어 있다면 확인 없이 메시지를 받습니다.



참고: [rabbitmqctl](https://www.rabbitmq.com/rabbitmqctl.8.html)
