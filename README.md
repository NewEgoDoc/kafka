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
      - "29092:29092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: "zookeeper:2181"
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0

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
