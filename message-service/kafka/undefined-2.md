# 설치

아파치 카프카는 아파치 소프트웨어 재단에서 배포하고 있는 [아파치 카프카](https://kafka.apache.org/) 통해서 배포판을 다운로드 받아서 설치하면 됩니다.&#x20;

참고: [https://kafka.apache.org/quickstart](https://kafka.apache.org/quickstart)

우분투에서 설치하는 방법에 대해서 살펴보겠습니다.

자바가 설치 되어 있지 않으면 자바를 먼저 설치 합니다.

```sh
sudo apt update
sudo apt upgrades
udo apt install default-jdk // 1.8 버전 이상 설치
```

## 1. Kafka 설치

카프카는 카프카 배포판을 다운받은 후 압축을 풀고 원하는 폴더에 넣은 후 실행되므로 wget를 사용한 다운로드는 다음과 같습니다.

### 1-1. 다운로드

```sh
wget https://dlcdn.apache.org/kafka/3.2.0/kafka_2.13-3.7.0.tgz
```

* 파일 다운 로드 위치: [https://kafka.apache.org/downloads](https://kafka.apache.org/downloads)

### 1-2. 압축 해제 및 폴더 지정

<pre class="language-bash"><code class="lang-bash">$ tar -xzf kafka_2.13-3.7.0.tgz

// 압축 해제 후 원하는 폴더로 이동 
<strong>$ sudo mv kafka_2.13-3.2.0 /usr/local/kafka
</strong>$ cd kafka_2.13-3.7.0
</code></pre>

## 2. Kafka 시작

카프카는 주키퍼 또는 KRaft와 함께 시작을 해야 합니다. 여기서는 주키퍼와 함꼐 시작합니다.

```sh
// 주키퍼 시작
$ bin/zookeeper-server-start.sh config/zookeeper.properties

// 카프카 시작 
$ bin/kafka-server-start.sh config/server.properties
```

### 2-1.  properties

주키퍼와 카프카 옵션을 조정 하기 위해 zookeeper.properties. server.properties 을 수정하여 사용해야 합니다.

* **파일위치**: 카프카 설치 폴더/config

{% code title="zookeeper.properties" lineNumbers="true" %}
```sh
# the directory where the snapshot is stored.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
# Disable the adminserver by default to avoid port conflicts.
# Set the port to something non-conflicting if choosing to enable this
admin.enableServer=false
# admin.serverPort=8080

```
{% endcode %}

{% code title="server.properties" lineNumbers="true" %}
```sh
# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0

############################# Socket Server Settings #############################

# The address the socket server listens on. If not configured, the host name will be equal to the value of
# java.net.InetAddress.getCanonicalHostName(), with PLAINTEXT listener name, and port 9092.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://:9092

# Listener name, hostname and port the broker will advertise to clients.
# If not set, it uses the value for "listeners".
advertised.listeners=PLAINTEXT://x.x.x.x:9092

......
```
{% endcode %}

### 2-2. 우분트 서비스  대몬

서비스 대몬에 등록하기 위해서는 서비스 파일을 만들어서 시비스 등록을 해야 합니다.

#### 2-1-1.  주키퍼 서비스 파일 생성

vi 또는 nano와 같은 에디터를 통해 서비스 파일을 생성 합니다.

* sudo nano /etc/systemd/system/zookeeper.service

{% code lineNumbers="true" %}
```sh
[Unit]
Description=Apache Zookeeper server
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
ExecStart=/usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.>
ExecStop=/usr/local/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target

```
{% endcode %}

* 9 Line: 주키퍼 시작&#x20;
* 10 Line:  주키퍼 종료
* 11 Line: 프로세스가 비정상 종료되었을 때 서비스가 다시 시작 합니다.

#### 2-1-2.  카프카서비스 파일 생성

* sudo nano /etc/systemd/system/kafka.service

{% code lineNumbers="true" %}
```sh
[Unit]
Description=Apache Kafka Server
Documentation=http://kafka.apache.org/documentation.html
Requires=zookeeper.service

[Service]
Type=simple
Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
ExecStart=/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.propert>
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target

```
{% endcode %}

* 8 Line: 카프카 시작
* 9 Line:  카프카 종료

#### 2-1-3.  서비스 시작&#x20;

```sh
$ sudo systemctl daemon-reload
$ sudo systemctl start zookeeper 
$ sudo systemctl start kafka
```

## 3. Kafka Topic 생성&#x20;

```sh
$ bin/kafka-topics.sh --create 
                      --bootstrap-server localhost:9092 
                      --replication-factor 1 
                      --partitions 1 
                      --topic iabacus-test
```

* Topic 확인 :&#x20;

```sh
$ bin/kafka-topics.sh --describe 
                      --topic iabacus-test 
                      --bootstrap-server localhost:9092
```

* 속성&#x20;
  * topic: 토픽 이름
  * partitions: 파티션 수
  * replication-factor: 복제 수
  * create: 토픽 생성
  * list: 토픽 목록 확인
  * describe: 토픽 상세 확인
* 토픽 리스트:  bin/kafka-topics.sh --list --bootstrap-server localhost:9092

<figure><img src="../../.gitbook/assets/image (293).png" alt=""><figcaption></figcaption></figure>

참고: [https://kafka.apache.org/documentation/#intro\_concepts\_and\_terms](https://kafka.apache.org/documentation/#intro\_concepts\_and\_terms)

## 4. Topic 쓰기/읽기

* 프로규서를 메세지 송신

```sh
$ bin/kafka-console-producer.sh --broker-list localhost:9092 
                                --topic iabacus-test
```

* 컨슈머로 메세지 수신

```sh
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 
                              --topic iabacus-test 
```

<figure><img src="../../.gitbook/assets/image (292).png" alt=""><figcaption></figcaption></figure>

*   처음 부터 메세지 수신 (--from-beginning)\


    <figure><img src="../../.gitbook/assets/image (295).png" alt=""><figcaption></figcaption></figure>

그외 카프카의 기능을 학습하기 위해서는 [https://kafka.apache.org/quickstart](https://kafka.apache.org/quickstart) 확인 해 보시기 바랍니다.
