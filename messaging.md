Kafka Streams in Action
=======================
Bill Bejeck, Manning


# Kafka
```bash
docker exec -it <kafka-container-name> /bin/bash
kafka-topics --list --bootstrap-server localhost:9092
kafka-topics --describe --topic test-topic --bootstrap-server localhost:9092

kafka-topics --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
# opens interactive prompt
kafka-console-producer --topic test-topic --bootstrap-server localhost:9092
# open in another terminal
kafka-console-consumer --topic test-topic --from-beginning --bootstrap-server localhost:9092
```
