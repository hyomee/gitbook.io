# Debezium

Debezium은 데이터베이스 변경사항을 캡처하기 위한 오픈 소스 분산 플랫폼입니다. 이는 데이터베이스에 커밋된 모든 삽입, 업데이트, 삭제 작업의 변경 내용을 스트리밍합니다. Debezium은 뛰어난 내구성과 빠른 반응 속도를 자랑하여, 애플리케이션이 신속하게 대응할 수 있고, 문제 발생 시에도 이벤트 손실이 없습니다.

{% embed url="https://debezium.io/" %}

## 1. Debezium Kafka Plugin 설치

1. **다운로드:** Debezium Kafka Plugin Maven 사이트에서 사용하고자 하는 DB를 받습니다.
   * MySql: [https://central.sonatype.com/artifact/io.debezium/debezium-connector-mysql/2.6.2.Final](https://central.sonatype.com/artifact/io.debezium/debezium-connector-mysql/2.6.2.Final)
   * Mariadb: [https://central.sonatype.com/artifact/io.debezium/debezium-connector-mariadb](https://central.sonatype.com/artifact/io.debezium/debezium-connector-mariadb) 에서 제공되는 버전은 2.7.0 Beta1 입니다.
2. **압축해제:** 다운로드한 파일을 다음 폴더에 압축을 풉니다.
   * 다운로드파일: \
     \- **wget** https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/2.6.1.Final/debezium-connector-mysql-2.6.2.Final-plugin.tar.gz\
     \- **wget** https://repo1.maven.org/maven2/io/debezium/debezium-connector-jdbc/2.6.2.Final/debezium-connector-jdbc-2.6.2.Final-plugin.tar.gz
   * 압축해제: tar -zxvf debezium-connector-mysql-2.6.2.Final-plugin.tar.gz\
     **-  tar** zxvf debezium-connector-mysql-2.6.1.Final-plugin.tar.gz&#x20;
   * 카프카폴더: cd /usr/local/kafka
   * 폴더생성: mkdir connectors
   * plugin 이동: mv /압축해제폴더/debezium-connector-mysql /usr/local/kafka/connectors
3. **Plugin 속성 추가**: connect-distributed.properties
   * vi, nano와 에이터를 사용해서 다음을 추가 합니다.
     *   plugin.path=/usr/local/kafka/connectors \


         <figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
4. &#x20;**설치 테스트**: 다음과 같이 실행 합니다.
   *   실행

       ```
       // 주키퍼 시작
       $ bin/zookeeper-server-start.sh config/zookeeper.properties

       // 카프카 시작 
       $ bin/kafka-server-start.sh config/server.properties

       // 카프카 distributed 시젇
       $ bin/connect-distributed.sh config/connect-distributed.properties

       $ bin/kafka-topics.sh --list --bootstrap-server localhost:9092 
       ```
   *   확인

       ```
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
5.  &#x20;**Connector Config**: &#x20;

    ```url

    curl --location 'localhost:8083/connectors' \
    --header 'Content-Type: application/json' \
    --data '{
      "name": "kafka-debezium-connector",
      "config": {
        "name": "kafka-debezium-connector",
        "connector.class": "io.debezium.connector.mysql.MySqlConnector",
        "tasks.max": "1",
        "connector.adapter": "mariadb",
        "database.protocol": "jdbc:mariadb",
        "database.jdbc.driver": "org.mariadb.jdbc.Driver",
        "database.hostname": "xxx.xx.xx.xxx",
        "database.port": "3306",
        "database.user": "hong",
        "database.password": "1234",
        "database.server.id": "112233",
        "database.server.name": "hong",
        "database.ssl.mode": "disabled", 
        "database.allowPublicKeyRetrieval":"true",     
        "database.history.kafka.bootstrap.servers": "localhost:9092", 
        "database.history.kafka.topic": "dbhistory.hong",   
        "snapshot.mode": "when_needed",
        "include.schema.changes": "false",
        "topic.prefix": "debezium.prfix",
        "table.include.list": "kafka_connect",
        "skip.messages.without.change": "true",
        "key.converter": "org.apache.kafka.connect.json.JsonConverter",
        "key.converter.schemas.enable": "true",
        "value.converter": "org.apache.kafka.connect.json.JsonConverter",
        "value.converter.schemas.enable": "true",
        "transforms": "unwrap,addTopicPrefix",
        "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
        "transforms.addTopicPrefix.type":"org.apache.kafka.connect.transforms.RegexRouter",
        "transforms.addTopicPrefix.regex":"(.*)",
        "transforms.addTopicPrefix.replacement":"$1",
        "snapshot.locking.mode": "none"
      }
    }
    ```

    * 설정 timezone 에러 조치 : $ mysql\_tzinfo\_to\_sql /usr/share/zoneinfo | mysql -u root -p mysql\
      \- 참고 : [https://database.guide/how-to-set-up-named-time-zones-in-mariadb/](https://database.guide/how-to-set-up-named-time-zones-in-mariadb/)
    * 오류 : Change the MySQL configuration to use a binlog\_format=ROW and restart the connector. 이면 다음과 같이 수
      * mysql> show global variables like 'binlog\_format';
      * mysql> SET GLOBAL binlog\_format = ROW;
      * mysql> flush privileges;
    * 오류: BINARY LOGS 관련 오류
      * DB확인: SHOW BINARY LOGS;
      *   설정:  sudo nano /etc/my.cnf

          ```
          [client]
          [mysqld] 
          log-bin=mysql-bin 
          expire-logs-days=1
          binlog_format=ROW 
          ```
      * 파일 기본 위치: /var/lib/mysql
      * 확인: show variables like 'log\_bin';
    *   결과 확인 : http://localhost:8083/connectors\


        <figure><img src="../../../.gitbook/assets/image (348).png" alt=""><figcaption></figcaption></figure>

        *   상세정보: http://localhost:8083/connectors/{connector-name}/config\
            \- 예: http://localhost:8083/connectors/kafka-debezium-connector/config\


            <figure><img src="../../../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>
        * 삭제: curl --location --request DELETE 'http://localhost:8083/connectors/{connector-name}
    *   속성: \


        <table><thead><tr><th width="221">속성</th><th>설명</th></tr></thead><tbody><tr><td>name</td><td>Connector 이름</td></tr><tr><td>tasks.max</td><td>Kafka Connect Cluster 의 인스턴스의 갯수를 지정</td></tr><tr><td>database.server.id</td><td>Primary - Replica 구조로 Replication 을 수행하게 될 때에 각 MySQL 인스턴스에 server-id 를 설정<br>- database.server.id 를 부여하여 정상적인 Replication 을 수행</td></tr><tr><td>database.server.name</td><td><p>Kafka Connect 에서 사용할 고유한 이름</p><ul><li>Debezium Connector 와 연결할 MySQL 서버를 지칭하는 논리적 이름</li><li>Debezium Kafka Connector 와 관련된 Topic 의 Prefix 로 사용</li></ul></td></tr><tr><td>database.history.kafka.bootstrap.servers</td><td>부트스트랩 서버 주소</td></tr><tr><td>database.history.kafka.topic</td><td>데이터베이스 스키마 변경을 추적하기 위해 Debezium에서 내부적으로 사용하는 Kafka 주제</td></tr><tr><td>database.include.list</td><td><p>Change Data Capture (CDC) 를 수행할 MySQL 의 database 목록</p><ul><li>database가 db1, db2, db3 인 경우 CDC 수행 DB가 db1, db2 이면 database.include.list 는 db1, db2 가 됩니다.</li></ul></td></tr><tr><td>database.exclude.list</td><td>database.include.list 반대 개념</td></tr><tr><td>table.include.list</td><td>CDC 모니터링 대상인 table 을 지정</td></tr><tr><td>table.exclude.list</td><td>table.include.list 반대 개</td></tr></tbody></table>

        \
        참고: [https://debezium.io/documentation/reference/stable/connectors/mysql.html#mysql-connector-properties](https://debezium.io/documentation/reference/stable/connectors/mysql.html#mysql-connector-properties)\

6. topic 확인\
   bin/kafka-topics.sh -- list --bootstrap-server localhost:9092\

7. curl -XPOST http://localhost:8083/connectors/connector\_name/restart \
   curl -XPOST http://localhost:8083/connectors/connector\_name/tasks/n/restart \
   curl -XPUT http://localhost:8083/connectors/connector\_name/pause \
   curl -XPUT http://localhost:8083/connectors/connector\_name/resume
8.





<pre><code><strong>http://localhost:8083/connectors
</strong></code></pre>



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
