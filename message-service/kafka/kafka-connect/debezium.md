# Debezium

Debezium은 데이터베이스 변경사항을 캡처하기 위한 오픈 소스 분산 플랫폼입니다. 이는 데이터베이스에 커밋된 모든 삽입, 업데이트, 삭제 작업의 변경 내용을 스트리밍합니다. Debezium은 뛰어난 내구성과 빠른 반응 속도를 자랑하여, 애플리케이션이 신속하게 대응할 수 있고, 문제 발생 시에도 이벤트 손실이 없습니다.

{% embed url="https://debezium.io/" %}

## 1. Debezium Kafka Plugin 설치

* **다운로드:** Debezium Kafka Plugin Maven 사이트에서 사용하고자 하는 DB를 받습니다.
  * MySql: [https://central.sonatype.com/artifact/io.debezium/debezium-connector-mysql/2.6.2.Final](https://central.sonatype.com/artifact/io.debezium/debezium-connector-mysql/2.6.2.Final)
  * Mariadb: [https://central.sonatype.com/artifact/io.debezium/debezium-connector-mariadb](https://central.sonatype.com/artifact/io.debezium/debezium-connector-mariadb) 에서 제공되는 버전은 2.7.0 Beta1 입니다.
* **압축해제:** 다운로드한 파일을 다음 폴더에 압축을 풉니다.
  * 다운로드파일: \
    \- **wget** https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/2.6.1.Final/debezium-connector-mysql-2.6.2.Final-plugin.tar.gz\
    \- **wget** https://repo1.maven.org/maven2/io/debezium/debezium-connector-jdbc/2.6.2.Final/debezium-connector-jdbc-2.6.2.Final-plugin.tar.gz
  * 압축해제: tar -zxvf debezium-connector-mysql-2.6.2.Final-plugin.tar.gz\
    **-  tar** zxvf debezium-connector-mysql-2.6.1.Final-plugin.tar.gz&#x20;
  * 카프카폴더: cd /usr/local/kafka
  * 폴더생성: mkdir connectors
  * plugin 이동: mv /압축해제폴더/debezium-connector-mysql /usr/local/kafka/connectors
* **Plugin 속성 추가**: connect-distributed.properties
  * vi, nano와 에이터를 사용해서 다음을 추가 합니다.
    *   plugin.path=/usr/local/kafka/connectors \


        <figure><img src="../../../.gitbook/assets/image (348).png" alt=""><figcaption></figcaption></figure>
* &#x20;**설치 테스트**: 다음과 같이 실행 합니다.
  *   **실행**

      ```
      // 주키퍼 시작
      $ bin/zookeeper-server-start.sh config/zookeeper.properties

      // 카프카 시작 
      $ bin/kafka-server-start.sh config/server.properties

      // 카프카 distributed 시젇
      $ bin/connect-distributed.sh config/connect-distributed.properties

      $ bin/kafka-topics.sh --list --bootstrap-server localhost:9092  
      ```
  *   **확인**

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

## &#x20;**2. Connector Config**: &#x20;

### 2-1. MySQL 충족 조건

Debezium Kafka Connect를 다음과 같은 조건이 만족 되어야 합니다.

* timezone: UTCㅇ어야 합니다.
* binlog: 활성이 되어 있어야 합니다.
  * SHOW BINARY LOGS;
* 권한: SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT이 접근하는 사용자에게 있어야 합니다.
  * GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON hongdb.\* TO 'hong'@'localhost';&#x20;
  * GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON _._ TO 'hong'@'localhost';
* MySql 설치 방법: [기본적인 설치 방법 및 계정](https://hyomee.gitbook.io/solution/undefined/mysql)&#x20;

**my.cnf 파일은 다음과 같습니다.**

```
[mysqld]
server-id         = 112233
log_bin           = mysql-bin
binlog_format     = row
binlog_row_image  = full
expire_logs_days  = 1
default-time-zone='+0:00'
```

### 2-1. Connect Topic 생성

Connect Topic 생성은 API로 작업을 생성 작업은 다음과 같습니다.

```url
curl --location 'localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data '{
  "name": "debezium-mysql-01",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "database.hostname": "localhost",
    "database.port": "3306",
    "database.user": "hong",
    "database.password": "hongxxxx",
    // my.cnf 설정한 server-id와 같아여 함
    "database.server.id": "112233",
    "database.server.name": "mysql",
    "topic.prefix": "mysql_topic",
    "database.whitelist": "hongdb",
    // 아래 주석 부분 주석을 풀면 Database Topic 이 생성 되지 않음 
    // "database.include.list": "hongdb",
    // "table.include.list": "hongdb.kafka_connect",
    "table.whitelist": "hongdb.kafka_connect",
    "database.history.kafka.bootstrap.servers": "172.24.239.164:9092",
    "database.history.kafka.recovery.attempts": "10000",
    "database.history.kafka.topic": "debezium.dbhistory.mysql",    
    "schema.history.internal.kafka.topic": "schema-history-01",
    "schema.history.internal.kafka.bootstrap.servers": "172.24.239.164:9092",
    "include.schema.changes": "true"  
  }
}'

```

*   속성: \


    <table><thead><tr><th width="221">속성</th><th>설명</th></tr></thead><tbody><tr><td>name</td><td>Connector 이름</td></tr><tr><td>tasks.max</td><td>Kafka Connect Cluster 의 인스턴스의 갯수를 지정</td></tr><tr><td>database.server.id</td><td>Primary - Replica 구조로 Replication 을 수행하게 될 때에 각 MySQL 인스턴스에 server-id 를 설정<br>- database.server.id 를 부여하여 정상적인 Replication 을 수행</td></tr><tr><td>database.server.name</td><td><p>Kafka Connect 에서 사용할 고유한 이름</p><ul><li>Debezium Connector 와 연결할 MySQL 서버를 지칭하는 논리적 이름</li><li>Debezium Kafka Connector 와 관련된 Topic 의 Prefix 로 사용</li></ul></td></tr><tr><td>database.history.kafka.bootstrap.servers</td><td>부트스트랩 서버 주소</td></tr><tr><td>database.history.kafka.topic</td><td>데이터베이스 스키마 변경을 추적하기 위해 Debezium에서 내부적으로 사용하는 Kafka 주제</td></tr><tr><td>topic.prefix</td><td>Kafka Topic 자동 생성 시 접두사</td></tr><tr><td>database.whitelist</td><td><p>Change Data Capture (CDC) 를 수행할 MySQL 의 database 목록</p><ul><li>database가 db1, db2, db3 인 경우 CDC 수행 DB가 db1, db2 이면 database.include.list 는 db1, db2 가 됩니다.</li></ul></td></tr><tr><td>database.include.list</td><td><p>Change Data Capture (CDC) 를 수행할 MySQL 의 database 목록</p><ul><li>database가 db1, db2, db3 인 경우 CDC 수행 DB가 db1, db2 이면 database.include.list 는 db1, db2 가 됩니다.</li></ul></td></tr><tr><td>database.exclude.list</td><td>database.include.list 반대 개념</td></tr><tr><td>table.include.list</td><td>CDC 모니터링 대상인 table 을 지정</td></tr><tr><td>table.exclude.list</td><td>table.include.list 반대 개</td></tr></tbody></table>

    \
    참고: [https://debezium.io/documentation/reference/stable/connectors/mysql.html#mysql-connector-properties](https://debezium.io/documentation/reference/stable/connectors/mysql.html#mysql-connector-properties)\

* 결과 확인:
  *   **Connectors** :  http://localhost:8083/connectors\


      <figure><img src="../../../.gitbook/assets/image (348) (1).png" alt=""><figcaption></figcaption></figure>
  *   **설정정보**:  http://localhost:8083/connectors/{connector-name}/config\
      \- 예: http://localhost:8083/connectors/debezium-mysql-01/config\


      <figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

      * **삭제**:\
        \-  curl --location --request DELETE 'http://localhost:8083/connectors/{connector-name}
*   **topic 확인**\
    \-  sudo ./bin/kafka-topics.sh --list --bootstrap-server localhost:9092

    <figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
* 그와  사용 API
  * curl -XPOST http://localhost:8083/connectors/connector\_name/restart&#x20;
  * curl -XPOST http://localhost:8083/connectors/connector\_name/tasks/n/restart&#x20;
  * curl -XPUT http://localhost:8083/connectors/connector\_name/pause&#x20;
  * curl -XPUT http://localhost:8083/connectors/connector\_name/resume



### 2-3. 테스트

*   **테이블 생성**:

    ```sql
    create table kafka_connect
    (
        id   int auto_increment
            primary key,
        name varchar(100) null
    );
    ```
*   **Kafka 실행**:

    ```sh
    // 주키퍼 시작
    $ bin/zookeeper-server-start.sh config/zookeeper.properties

    // 카프카 시작 
    $ bin/kafka-server-start.sh config/server.properties

    // 카프카 distributed 시젇
    $ bin/connect-distributed.sh config/connect-distributed.properties

    $ bin/kafka-topics.sh --list --bootstrap-server localhost:9092 

    ```
*   insert 진행 \


    ```xquery
    INSERT INTO hongdb.kafka_connect (name) VALUES ('홍길동');
    INSERT INTO hongdb.kafka_connect (name) VALUES ('김길동'); 
    commit;
    ```
*   컨슈머:&#x20;



    ```sh
    $ sudo ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mysql_topic.hongdb.kafka_connect
    ```





    ```json
    {
        "schema": {
            "type": "struct",
            "fields": [
                {
                    "type": "struct",
                    "fields": [
                        {
                            "type": "int32",
                            "optional": false,
                            "field": "id"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "name"
                        }
                    ],
                    "optional": true,
                    "name": "mysql_topic.hongdb.kafka_connect.Value",
                    "field": "before"
                },
                {
                    "type": "struct",
                    "fields": [
                        {
                            "type": "int32",
                            "optional": false,
                            "field": "id"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "name"
                        }
                    ],
                    "optional": true,
                    "name": "mysql_topic.hongdb.kafka_connect.Value",
                    "field": "after"
                },
                {
                    "type": "struct",
                    "fields": [
                        {
                            "type": "string",
                            "optional": false,
                            "field": "version"
                        },
                        {
                            "type": "string",
                            "optional": false,
                            "field": "connector"
                        },
                        {
                            "type": "string",
                            "optional": false,
                            "field": "name"
                        },
                        {
                            "type": "int64",
                            "optional": false,
                            "field": "ts_ms"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "name": "io.debezium.data.Enum",
                            "version": 1,
                            "parameters": {
                                "allowed": "true,last,false,incremental"
                            },
                            "default": "false",
                            "field": "snapshot"
                        },
                        {
                            "type": "string",
                            "optional": false,
                            "field": "db"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "sequence"
                        },
                        {
                            "type": "int64",
                            "optional": true,
                            "field": "ts_us"
                        },
                        {
                            "type": "int64",
                            "optional": true,
                            "field": "ts_ns"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "table"
                        },
                        {
                            "type": "int64",
                            "optional": false,
                            "field": "server_id"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "gtid"
                        },
                        {
                            "type": "string",
                            "optional": false,
                            "field": "file"
                        },
                        {
                            "type": "int64",
                            "optional": false,
                            "field": "pos"
                        },
                        {
                            "type": "int32",
                            "optional": false,
                            "field": "row"
                        },
                        {
                            "type": "int64",
                            "optional": true,
                            "field": "thread"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "query"
                        }
                    ],
                    "optional": false,
                    "name": "io.debezium.connector.mysql.Source",
                    "field": "source"
                },
                {
                    "type": "string",
                    "optional": false,
                    "field": "op"
                },
                {
                    "type": "int64",
                    "optional": true,
                    "field": "ts_ms"
                },
                {
                    "type": "int64",
                    "optional": true,
                    "field": "ts_us"
                },
                {
                    "type": "int64",
                    "optional": true,
                    "field": "ts_ns"
                },
                {
                    "type": "struct",
                    "fields": [
                        {
                            "type": "string",
                            "optional": false,
                            "field": "id"
                        },
                        {
                            "type": "int64",
                            "optional": false,
                            "field": "total_order"
                        },
                        {
                            "type": "int64",
                            "optional": false,
                            "field": "data_collection_order"
                        }
                    ],
                    "optional": true,
                    "name": "event.block",
                    "version": 1,
                    "field": "transaction"
                }
            ],
            "optional": false,
            "name": "mysql_topic.hongdb.kafka_connect.Envelope",
            "version": 2
        },
        "payload": {
            "before": null,
            "after": {
                "id": 75,
                "name": "홍길동"
            },
            "source": {
                "version": "2.6.2.Final",
                "connector": "mysql",
                "name": "mysql_topic",
                "ts_ms": 1718303427000,
                "snapshot": "false",
                "db": "hongdb",
                "sequence": null,
                "ts_us": 1718303427000000,
                "ts_ns": 1718303427000000000,
                "table": "kafka_connect",
                "server_id": 112233,
                "gtid": null,
                "file": "mysql-bin.000010",
                "pos": 17182,
                "row": 0,
                "thread": 15,
                "query": null
            },
            "op": "c",
            "ts_ms": 1718303427385,
            "ts_us": 1718303427385117,
            "ts_ns": 1718303427385117710,
            "transaction": null
        }
    }
    {
        "schema": {
            "type": "struct",
            "fields": [
                {
                    "type": "struct",
                    "fields": [
                        {
                            "type": "int32",
                            "optional": false,
                            "field": "id"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "name"
                        }
                    ],
                    "optional": true,
                    "name": "mysql_topic.hongdb.kafka_connect.Value",
                    "field": "before"
                },
                {
                    "type": "struct",
                    "fields": [
                        {
                            "type": "int32",
                            "optional": false,
                            "field": "id"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "name"
                        }
                    ],
                    "optional": true,
                    "name": "mysql_topic.hongdb.kafka_connect.Value",
                    "field": "after"
                },
                {
                    "type": "struct",
                    "fields": [
                        {
                            "type": "string",
                            "optional": false,
                            "field": "version"
                        },
                        {
                            "type": "string",
                            "optional": false,
                            "field": "connector"
                        },
                        {
                            "type": "string",
                            "optional": false,
                            "field": "name"
                        },
                        {
                            "type": "int64",
                            "optional": false,
                            "field": "ts_ms"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "name": "io.debezium.data.Enum",
                            "version": 1,
                            "parameters": {
                                "allowed": "true,last,false,incremental"
                            },
                            "default": "false",
                            "field": "snapshot"
                        },
                        {
                            "type": "string",
                            "optional": false,
                            "field": "db"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "sequence"
                        },
                        {
                            "type": "int64",
                            "optional": true,
                            "field": "ts_us"
                        },
                        {
                            "type": "int64",
                            "optional": true,
                            "field": "ts_ns"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "table"
                        },
                        {
                            "type": "int64",
                            "optional": false,
                            "field": "server_id"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "gtid"
                        },
                        {
                            "type": "string",
                            "optional": false,
                            "field": "file"
                        },
                        {
                            "type": "int64",
                            "optional": false,
                            "field": "pos"
                        },
                        {
                            "type": "int32",
                            "optional": false,
                            "field": "row"
                        },
                        {
                            "type": "int64",
                            "optional": true,
                            "field": "thread"
                        },
                        {
                            "type": "string",
                            "optional": true,
                            "field": "query"
                        }
                    ],
                    "optional": false,
                    "name": "io.debezium.connector.mysql.Source",
                    "field": "source"
                },
                {
                    "type": "string",
                    "optional": false,
                    "field": "op"
                },
                {
                    "type": "int64",
                    "optional": true,
                    "field": "ts_ms"
                },
                {
                    "type": "int64",
                    "optional": true,
                    "field": "ts_us"
                },
                {
                    "type": "int64",
                    "optional": true,
                    "field": "ts_ns"
                },
                {
                    "type": "struct",
                    "fields": [
                        {
                            "type": "string",
                            "optional": false,
                            "field": "id"
                        },
                        {
                            "type": "int64",
                            "optional": false,
                            "field": "total_order"
                        },
                        {
                            "type": "int64",
                            "optional": false,
                            "field": "data_collection_order"
                        }
                    ],
                    "optional": true,
                    "name": "event.block",
                    "version": 1,
                    "field": "transaction"
                }
            ],
            "optional": false,
            "name": "mysql_topic.hongdb.kafka_connect.Envelope",
            "version": 2
        },
        "payload": {
            "before": null,
            "after": {
                "id": 76,
                "name": "김길동"
            },
            "source": {
                "version": "2.6.2.Final",
                "connector": "mysql",
                "name": "mysql_topic",
                "ts_ms": 1718303427000,
                "snapshot": "false",
                "db": "hongdb",
                "sequence": null,
                "ts_us": 1718303427000000,
                "ts_ns": 1718303427000000000,
                "table": "kafka_connect",
                "server_id": 112233,
                "gtid": null,
                "file": "mysql-bin.000010",
                "pos": 17489,
                "row": 0,
                "thread": 15,
                "query": null
            },
            "op": "c",
            "ts_ms": 1718303427497,
            "ts_us": 1718303427497570,
            "ts_ns": 1718303427497570905,
            "transaction": null
        }
    }
    ```

    \


    <figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

***

* 참고
  * [https://debezium.io/documentation/reference/2.6/connectors/mysql.html#setting-up-mysql](https://debezium.io/documentation/reference/2.6/connectors/mysql.html#setting-up-mysql)
  * [https://debezium.io/documentation/reference/stable/connectors/mysql.html#mysql-connector-properties](https://debezium.io/documentation/reference/stable/connectors/mysql.html#mysql-connector-properties)
