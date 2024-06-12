# Debezium

Debezium은 데이터베이스 변경사항을 캡처하기 위한 오픈 소스 분산 플랫폼입니다. 이는 데이터베이스에 커밋된 모든 삽입, 업데이트, 삭제 작업의 변경 내용을 스트리밍합니다. Debezium은 뛰어난 내구성과 빠른 반응 속도를 자랑하여, 애플리케이션이 신속하게 대응할 수 있고, 문제 발생 시에도 이벤트 손실이 없습니다.

{% embed url="https://debezium.io/" %}

## 1. Debezium Kafka Plugin 설치

1. **다운로드:** Debezium Kafka Plugin Maven 사이트에서 사용하고자 하는 DB를 받습니다.
   * MySql: [https://central.sonatype.com/artifact/io.debezium/debezium-connector-mysql/2.6.2.Final](https://central.sonatype.com/artifact/io.debezium/debezium-connector-mysql/2.6.2.Final)
   * Mariadb: [https://central.sonatype.com/artifact/io.debezium/debezium-connector-mariadb](https://central.sonatype.com/artifact/io.debezium/debezium-connector-mariadb) 에서 제공되는 버전은 2.7.0 Beta1 입니다.
2. **압축해제:** 다운로드한 파일을 다음 폴더에 압축을 풉니다.
   * 다운로드파일: debezium-connector-mysql-2.6.2.Final-plugin.tar.gz\
     \- **wget https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/2.6.1.Final/debezium-connector-mysql-2.6.2.Final-plugin.tar.gz**\
     **-** [**https://repo1.maven.org/maven2/io/debezium/debezium-connector-jdbc/2.6.2.Final/debezium-connector-jdbc-2.6.2.Final-plugin.tar.gz**](https://repo1.maven.org/maven2/io/debezium/debezium-connector-jdbc/2.6.2.Final/debezium-connector-jdbc-2.6.2.Final-plugin.tar.gz)
   * 압축해제: tar -zxvf debezium-connector-mysql-2.6.2.Final-plugin.tar.gz\
     **-  tar zxvf debezium-connector-mysql-2.6.1.Final-plugin.tar.gz**&#x20;
   * 카프카폴더: cd /usr/local/kafka
   * 폴더생성: mkdir connectors
   * plugin 이동: mv /압축해제폴더/debezium-connector-mysql /usr/local/kafka/connectors
3. Plugin 속성 추가: connect-distributed.properties
   * vi, nano와 에이터를 사용해서 다음을 추가 합니다.
     *   plugin.path=/usr/local/kafka/connectors \


         <figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
4.  &#x20;설치 테스트: 다음과 같이 실행 합니다.\


    ```
    // 주키퍼 시작
    $ bin/zookeeper-server-start.sh config/zookeeper.properties

    // 카프카 시작 
    $ bin/kafka-server-start.sh config/server.properties

    // 카프카 distributed 시젇
    $ bin/connect-distributed.sh config/connect-distributed.properties

    // 결과 확인 
    $ curl http://localhost:8083
    {
        "version": "3.7.0",
        "commit": "2ae524ed625438c5",
        "kafka_cluster_id": "4L-j6V4mRHij37UoX02SjQ"
    }

    $ curl --location --request GET http://localhost:8083/connector-plugins
    [
        {
            "class": "io.debezium.connector.mysql.MySqlConnector",
            "type": "source",
            "version": "2.6.2.Final"
        },
        {
            "class": "org.apache.kafka.connect.mirror.MirrorCheckpointConnector",
            "type": "source",
            "version": "3.7.0"
        },
        {
            "class": "org.apache.kafka.connect.mirror.MirrorHeartbeatConnector",
            "type": "source",
            "version": "3.7.0"
        },
        {
            "class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
            "type": "source",
            "version": "3.7.0"
        }
    ]
    ```



```json
{
    "name": "source-mysql",
    "config": {
        "connector.class": "io.debezium.connector.mysql.MySqlConnector",
        "tasks.max": "1",
        "connector.adapter": "mariadb",
        "database.protocol": "jdbc:mariadb",
        "database.jdbc.driver": "org.mariadb.jdbc.Driver",
        "database.hostname": "xxx",
        "database.port": "3306",
        "database.user": "xxx",
        "database.password": "xxx",
        "database.ssl.mode": "disabled",
        "database.server.id": "184054",
        "topic.prefix": "mydb",
        "database.history.kafka.bootstrap.servers": "localhost:9092",
        "database.history.kafka.topic": "dbhistory.mydb",
        "key.converter": "org.apache.kafka.connect.json.JsonConverter",
        "key.converter.schemas.enable": "true",
        "value.converter": "org.apache.kafka.connect.json.JsonConverter",
        "value.converter.schemas.enable": "true",
        "name": "source-mysql"
    },
    "tasks": [],
    "type": "source"
}
```

[https://debezium.io/documentation/reference/stable/connectors/mysql.html#mysql-connector-properties](https://debezium.io/documentation/reference/stable/connectors/mysql.html#mysql-connector-properties)
