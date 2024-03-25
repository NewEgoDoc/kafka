# kafka

> kafka zookeeper 가동

./bin/zookeeper-server-start.sh ./config/zookeeper.properties

> kafka server 기동

./bin/kafka-server-start.sh ./config/server.properties

> topic 생성 및 partition 생성
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic quickstart-events --partitions 1
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe quickstart-events
./bin/kafka-console-producer.sh --broker-list localhost:9092 -topic quickstart-events

./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events --from-beginning
./bin/kafka-console-consumer.sh --bootstrap-serverlocalhost:9092 --topic quickstart-events --from-beginning
