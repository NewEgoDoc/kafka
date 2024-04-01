# kafka

> kafka zookeeper 가동

./bin/zookeeper-server-start.sh ./config/zookeeper.properties

> kafka server 기동

./bin/kafka-server-start.sh ./config/server.properties

> topic 생성 및 partition 생성
```bash
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic quickstart-events --partitions 1

./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe quickstart-events

./bin/kafka-console-producer.sh --broker-list localhost:9092 -topic quickstart-events
```

> topic consumer 1

```bash
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events --from-beginning
```
> topic consumer 2

```bash
./bin/kafka-console-consumer.sh --bootstrap-serverlocalhost:9092 --topic quickstart-events --from-beginning
```

위에는 내부로 
```docker
version: "3.8"

services:
  maria:
    image: mariadb
    container_name: maria
    restart: always
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: "idb1004#"
      MYSQL_DATABASE: mydb
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_SERVER_ID: 1
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 5
      ZOOKEEPER_SYNC_LIMIT: 2
    ports:
      - "22181:2181"

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - "29092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: "zookeeper:2181"
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0

  kafka-connect:
    image: confluentinc/cp-kafka-connect:latest
    depends_on:
      - kafka
    ports:
      - "8083:8083"
    environment:
      CONNECT_BOOTSTRAP_SERVERS: "kafka:9092"
      CONNECT_REST_ADVERTISED_HOST_NAME: "localhost"
      CONNECT_GROUP_ID: "connect-cluster"
      CONNECT_CONFIG_STORAGE_TOPIC: "connect-configs"
      CONNECT_OFFSET_STORAGE_TOPIC: "connect-offsets"
      CONNECT_STATUS_STORAGE_TOPIC: "connect-status"
      CONNECT_KEY_CONVERTER: "org.apache.kafka.connect.json.JsonConverter"
      CONNECT_VALUE_CONVERTER: "org.apache.kafka.connect.json.JsonConverter"
      CONNECT_INTERNAL_KEY_CONVERTER: "org.apache.kafka.connect.json.JsonConverter"
      CONNECT_INTERNAL_VALUE_CONVERTER: "org.apache.kafka.connect.json.JsonConverter"
      CONNECT_REST_PORT: 8083
      CONNECT_PLUGIN_PATH: "/usr/share/java,/etc/kafka-connect/jars"
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: 1

  kafka-ui:
    image: provectuslabs/kafka-ui
    depends_on:
      - kafka
    container_name: kafka-ui
    ports:
      - "9090:8080"
    restart: always
    environment:
      - KAFKA_CLUSTERS_0_NAME=local
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka:9092
      - KAFKA_CLUSTERS_0_ZOOKEEPER=zookeeper:2181

```

위의 도커 컴포즈는 해당 내용에서 mariadb와 connector를 연결 시키기위해 같이 올렸다.

kafka-connect container 안의 /usr/share/java/kafka에  kafka-connect-jdbc-10.7.6.jar mariadb-java-client-3.0.7.jar를 넣어줘야 mariadb를 연결시킬수 있고

```curl
curl --location '127.0.0.1:8083/connectors' \
--header 'Content-Type: application/json' \
--data '{
    "name" : "my-source-connect",
    "config" : {
        "connector.class" : "io.confluent.connect.jdbc.JdbcSourceConnector",
        "connection.url":"jdbc:mariadb://maria:3306/mydb",
        "connection.user":"root",
        "connection.password":"idb1004#",
        "mode": "incrementing",
        "incrementing.column.name" : "id",
        "table.whitelist":"mydb.users",
        "topic.prefix" : "my_topic_",
        "tasks.max" : "1"
    }
}'
```
mydb.users 이렇게 디비를 명시해주지 않으면 최근 버전에서는 performance_schema.users 와 헷갈려 오류가 난다

Kcaft용 Kafka로 kafka ui까지 실행
```yml
version: '3'

networks:
  kafka_network:

volumes:
  Kafka00:
    driver: local
  Kafka01:
    driver: local
  Kafka02:
    driver: local

services:
  Kafka00Service:
    image: bitnami/kafka:3.5.1-debian-11-r44
    restart: unless-stopped
    container_name: Kafka00Container
    ports:
      - '10000:9094'
    environment:
      - KAFKA_CFG_BROKER_ID=0
      - KAFKA_CFG_NODE_ID=0
      - KAFKA_KRAFT_CLUSTER_ID=HsDBs9l6UUmQq7Y5E6bNlw
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@Kafka00Service:9093,1@Kafka01Service:9093,2@Kafka02Service:9093
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://Kafka00Service:9092,EXTERNAL://127.0.0.1:10000
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
      - KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
    networks:
      - kafka_network
    volumes:
      - Kafka00:/bitnami/kafka

  Kafka01Service:
    image: bitnami/kafka:3.5.1-debian-11-r44
    restart: unless-stopped
    container_name: Kafka01Container
    ports:
      - '10001:9094'
    environment:
      - KAFKA_CFG_BROKER_ID=1
      - KAFKA_CFG_NODE_ID=1
      - KAFKA_KRAFT_CLUSTER_ID=HsDBs9l6UUmQq7Y5E6bNlw
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@Kafka00Service:9093,1@Kafka01Service:9093,2@Kafka02Service:9093
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://Kafka01Service:9092,EXTERNAL://127.0.0.1:10001
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
      - KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
    networks:
      - kafka_network
    volumes:
      - Kafka01:/bitnami/kafka

  Kafka02Service:
    image: bitnami/kafka:3.5.1-debian-11-r44
    restart: unless-stopped
    container_name: Kafka02Container
    ports:
      - '10002:9094'
    environment:
      - KAFKA_CFG_BROKER_ID=2
      - KAFKA_CFG_NODE_ID=2
      - KAFKA_KRAFT_CLUSTER_ID=HsDBs9l6UUmQq7Y5E6bNlw
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@Kafka00Service:9093,1@Kafka01Service:9093,2@Kafka02Service:9093
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://Kafka02Service:9092,EXTERNAL://127.0.0.1:10002
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
      - KAFKA_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - KAFKA_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
      - KAFKA_CFG_PROCESS_ROLES=controller,broker
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
    networks:
      - kafka_network
    volumes:
      - Kafka02:/bitnami/kafka

  KafkaWebUiService:
    image: provectuslabs/kafka-ui:latest
    restart: always
    container_name: KafkaWebUiContainer
    ports:
      - '8085:8080' # 호스트의 8085 포트를 컨테이너의 8080 포트에 바인딩
    environment:
      - KAFKA_CLUSTERS_0_NAME=Local-Kraft-Cluster
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=Kafka00Service:9092,Kafka01Service:9092,Kafka02Service:9092
      - DYNAMIC_CONFIG_ENABLED=true
      - KAFKA_CLUSTERS_0_AUDIT_TOPICAUDITENABLED=true
      - KAFKA_CLUSTERS_0_AUDIT_CONSOLEAUDITENABLED=true
    depends_on:
      - Kafka00Service
      - Kafka01Service
      - Kafka02Service
    networks:
      - kafka_network
```
