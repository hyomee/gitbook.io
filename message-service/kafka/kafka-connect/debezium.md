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

      ````
      $ curl http://localhost:8083
      {
          "version": "3.7.0",
          "commit": "2ae524ed625438c5",
          "kafka_cluster_id": "4L-j6V4mRHij37UoX02SjQ"
      }

      $ curl --location --request GET http://localhost:8083/connector-plugins
      ```json
      [
          {
              "class": "io.debezium.connector.jdbc.JdbcSinkConnector",
              "type": "sink",
              "version": "2.6.2.Final"
          },
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

      ````

## **2. Source Connector**&#x20;

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
    "database.password": "hong1234",
    "database.server.id": "112233",
    "database.server.name": "mysql",
    "topic.prefix": "mysql_topic",                  // prefix for topiucs   
    "database.include.list": "hongdb",              // select a DB to replicate
    "table.include.list": "hongdb.kafka_connect",   // 변경 이벤트 레코드에 포함할 테이블을 정규 표현식으로 지정
    "database.whitelist": "hongdb",                 // monitor a DB
    "table.whitelist": "hongdb.kafka_connect",      // 연결할 데이터베이스 테이블을 지정
    "topic.creation.enable": "true",                //  auto-create topics  
    "topic.creation.default.replication.factor": 1,  
    "topic.creation.default.partitions": 3,  
    "topic.creation.default.cleanup.policy": "compact",  
    "topic.creation.default.compression.type": "lz4",       
    "database.history.kafka.bootstrap.servers": "localhost:9092", // list of brokers
    "database.history.kafka.recovery.attempts": "10000",
    "database.history.kafka.topic": "debezium.dbhistory.mysql",    
    "schema.history.internal.kafka.topic": "schema-history-01",
    "schema.history.internal.kafka.bootstrap.servers": "localhost:9092",
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


      <figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

      * **삭제**:\
        \-  curl --location --request DELETE 'http://localhost:8083/connectors/{connector-name}
*   **topic 확인**\
    \-  sudo ./bin/kafka-topics.sh --list --bootstrap-server localhost:9092

    <figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
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
* **Kafka 실행**:

```
// 주키퍼 시작
$ bin/zookeeper-server-start.sh config/zookeeper.properties

// 카프카 시작 
$ bin/kafka-server-start.sh config/server.properties
```

* **카프카 distributed 실행**

```sh
// 카프카 distributed 실행
$ bin/connect-distributed.sh config/connect-distributed.properties

$ bin/kafka-topics.sh --list --bootstrap-server localhost:9092 
__consumer_offsets
connect-configs
connect-offsets
connect-status
```

* **Topic 생성**: 2-1 Connect Topic 생성 curl 실행&#x20;

```sh
$ bin/kafka-topics.sh --list --bootstrap-server localhost:9092 
__consumer_offsets
connect-configs
connect-offsets
connect-status
mysql_topic
mysql_topic.hongdb.kafka_connect
schema-history-01
```

*   **MsSql  insert 쿼리  실행**&#x20;

    ```xquery
    INSERT INTO hongdb.kafka_connect (name) VALUES ('홍길동');
    INSERT INTO hongdb.kafka_connect (name) VALUES ('김길동'); 
    commit;
    ```
*   컨슈머:&#x20;



    ```sh
    $ sudo ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mysql_topic.hongdb.kafka_connect
    ```

<details>

<summary>JSON 결과 </summary>

<pre class="language-json"><code class="lang-json"><strong>{
</strong>    "schema": {
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
</code></pre>

</details>

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

***

참고

* [https://debezium.io/documentation/reference/2.6/connectors/mysql.html#setting-up-mysql](https://debezium.io/documentation/reference/2.6/connectors/mysql.html#setting-up-mysql)
* [https://debezium.io/documentation/reference/stable/connectors/mysql.html#mysql-connector-properties](https://debezium.io/documentation/reference/stable/connectors/mysql.html#mysql-connector-properties)

## **3. Sink Connector**

Debezium JDBC 커넥터는 Kafka Connect의 Sink 커넥터 구현으로, 여러 소스 토픽에서 이벤트를 소비한 후 JDBC 드라이버를 사용해 이벤트를 관계형 데이터베이스에 저장합니다.&#x20;

* Debezium JDBC 커넥터는 Kafka Connect의 싱크 커넥터이기 때문에 Kafka Connect 런타임이 필요합니다.&#x20;
* 구독 중인 Kafka 토픽을 주기적으로 폴링하여 토픽의 이벤트를 가져오고, 이를 구성된 관계형 데이터베이스에 기록합니다.
* Db2, MySQL, Oracle, PostgreSQL 및 SQL Server를 포함한 다양한 데이터베이스 언어를 지원합니다.
* **동작 방식**:
  * 커넥터는 주기적으로 구독하는 Kafka 토픽에서 이벤트를 가져와 설정된 관계형 데이터베이스에 메시지를.저장합니다
  * 커넥터는 upsert  및 기본 스키마를 사용하여 멱등성 쓰기 작업을 지원합니다.
  * 다음과 같은 기능을 제공합니다:
    * [Consuming complex Debezium change events](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-consume-complex-debezium-events) (복잡한 Debezium 변경 이벤트 소비)
    * [At-least-once delivery](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-at-least-once-delivery) (최소 한 번 이상의 전달 보장)
    * [Multiple tasks](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-multiple-tasks) (여러 작업 실행): tasks.max
    * [Data and column type mappings](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-data-and-type-mappings) (데이터 및 열 유형 매핑)
    * [Primary key handling](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-primary-key-handling) (기본 키 생성): 이벤트 기본키 정
      * primary.key.mode: none, kafka, record\_key, record\_value 등
      * primary.key.fields 속성 합니다.
    * [Delete mode](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-delete-mode) (행 삭제 여부): delete.enabled=true를 명시적으로 설정해야 합니다
      * delete.enabled: true, none
    *   [Idempotent writes](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-idempotent-writes) ( 멱등성 쓰기 )\


        | Dialect    | Upsert Syntax                               |
        | ---------- | ------------------------------------------- |
        | Db2        | `MERGE …​`                                  |
        | MySQL      | `INSERT …​ ON DUPLICATE KEY UPDATE …​`      |
        | Oracle     | `MERGE …​`                                  |
        | PostgreSQL | `INSERT …​ ON CONFLICT …​ DO UPDATE SET …​` |
        | SQL Server | `MERGE …​`                                  |
    * [Schema evolution](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-schema-evolution) (테이블을 자동으로 생성하거나 변경)
      * schema.evolution: none, basic
    * [Quoting and case sensitivity](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-quoting-case-sensitivity) (DDL (스키마 변경), DML(데이터 변경)  에)대한 대소문자 구분)
      * quote.identifiers: true(테이블 및 필드 이름의 대소문자를 명시적으로 유지), false&#x20;
    * [Connection Idle Timeouts](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-connection-idle-timeouts) (연결 유휴 시간 초과)

### 3-1.  사용 방법

#### 3-1-1. 필수 구성 요소

* [Apache ZooKeeper](https://zookeeper.apache.org/), [Apache Kafka](http://kafka.apache.org/) 및 [Kafka Connect](https://kafka.apache.org/documentation.html#connect)가 설치되어 있습니다.
* 대상 데이터베이스가 설치되고 JDBC 연결을 허용하도록 구성됩니다.

#### 3-1-2. 절차

1. Debezium [JDBC 커넥터 플러그인 아카이브](https://repo1.maven.org/maven2/io/debezium/debezium-connector-jdbc/2.6.2.Final/debezium-connector-jdbc-2.6.2.Final-plugin.tar.gz)를 다운로드합니다.
2. Kafka Connect 환경으로 파일을 추출합니다.
3. 필요에 따라 Maven Central에서 JDBC 드라이버를 다운로드하고 다운로드한 드라이버 파일을 JDBC 싱크 커넥터 JAR 파일이 포함된 디렉토리에 추출합니다.
4. JDBC 싱크 커넥터가 설치된 경로에 드라이버 JAR 파일을 추가합니다.
5. JDBC 싱크 커넥터를 설치하는 경로가 [Kafka Connect `plugin.path`](https://kafka.apache.org/documentation/#connectconfigs)의 일부인지 확인합니다.
6. Kafka Connect 프로세스를 다시 시작하여 새 JAR 파일을 선택합니다.

### 3-2.  Debezium JDBC 커넥터 구성

커넥터 속성은 다음에서 참고합니다.

* [JCBC 커넥터 일반 특성](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-connector-properties-generic)
* [JDBC 커넥터 연결 특성](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-connector-properties-connection)
* [JDBC 커넥터 런타임 특성](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-connector-properties-runtime)
* [JDBC 커넥터 확장 가능 특성](https://debezium.io/documentation/reference/stable/connectors/jdbc.html#jdbc-connector-properties-extendable)

```concurnas
curl --location 'localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data '{ 
    "name": "debezium-mysql-sink-connector",  
    "config": {
        // 제네릭 속성
        "connector.class": "io.debezium.connector.jdbc.JdbcSinkConnector",  
        "tasks.max": "1",
        "topics": "mysql_topic.hongdb.kafka_connect",                // 사용할 항목 목록으로, 쉼표로 구분
        //"topics.regex":"mysql_topic.hongdb.*", // topics와 함꼐 사용하지 못함
        "heartbeat.interval.ms": "3000",                                                            
        "autoReconnect":"true",           
        // JDBC 커넥터 연결 특성
        "connection.url":"jdbc:mysql://localhost:3306/hong",   
        "connection.username": "hong",  
        "connection.password": "hong1234" ,
        "connection.pool.min_size": 5,
        "connection.pool.max_size": 32,
        // JDBC 커넥터 런타임 특성
        "delete.enabled": "true",         // 커넥터가 이벤트를 처리할지 또는 삭제 표시 이벤트를 처리할지 여부를 지정
        "database.time_zone": "UTC",      // 임시 필드 유형을 작성할 때 사용되는 시간대를 지정
        "insert.mode": "upsert",          // 데이터베이스에 이벤트를 삽입하는 데 사용되는 전략을 지정
        "primary.key.mode": "record_key", // 기본 키 열을 확인하는 데 사용되는 방법을 지정
        "quote.identifiers":"true",
        "schema.evolution": "basic",      // 커넥터가 대상 테이블 스키마를 전개시키는 방법을 지정
        "table.name.format": "kafka_connect",
        // "table.name.format": "${topic}",  // 이벤트의 주제 이름에 따라 대상 테이블 이름의 형식을 지정하는 방법을 결정하는 문자열을 지정
        //  JDBC 커넥터 확장 가능 특성
        "auto.evolve": "true",            // Target DB 열이없을 경우, 자동으로 만들지 여부 지정                           
        "auto.create":"true",             // Target DB에 해당 테이블이 없을 경우, 자동으로 만들지 여부 지정                           
        "value.converter.schemas.enable":"true",              
        "value.converter":"org.apache.kafka.connect.json.JsonConverter",       
        "pk.mode" :"kafka" // Primary Key (PK) 모드를 설정하는 데 사용
  }
} 
```

* 추가 속성 으로 싱크 커넥터에 다음 구성을 추가하여 데이터를 변환하고 해당 구성을 다시 적용할 수 있습니다

```
"transforms": "unwrap, rename",
"transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
"transforms.rename.type": "org.apache.kafka.connect.transforms.RegexRouter",
"transforms.rename.regex": ".*\\.(.*)",
"transforms.rename.replacement": "$1",
```

### 3-3. 결과

#### 3-3-1. 생성 확인&#x20;

````
localhost:8083/connectors

```json
[
    "debezium-mysql-01",
    "debezium-mysql-sink-connector"
]
```

````

#### 3-3-2. 테스트

* Source DB 에서 INSERT

```xquery
INSERT INTO hongdb.kafka_connect (name) VALUES ('홍길동 나라');
INSERT INTO hongdb.kafka_connect (name) VALUES ('김길동 나라');
commit;
```

* Target DB  확인

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption><p>수핸전</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption><p>수행후</p></figcaption></figure>

***

**참고:** [**https://debezium.io/documentation/reference/stable/connectors/jdbc.html**](https://debezium.io/documentation/reference/stable/connectors/jdbc.html)

참고: [Kafka Connect, Debezium로 PostgreSQL CDC 구성하기](https://thekoguryo.github.io/oci/chapter17/oci-oss-cdc-postgresql-debezium/) &#x20;

