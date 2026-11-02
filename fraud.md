# Fraud producer, consumer, topic/s and event/s
## Topic configs
kubectl exec -it kafka-0 -- kafka-topics \
  --bootstrap-server kafka-service:9092 \
  --create \
  --topic digital-payments.fraud.score.events.v1 \
  --partitions 100 \
  --replication-factor 3 \
  --config retention.ms= 220924800000 \
  --config cleanup.policy= delete \
  --config min.insync.replicas= 2 \
  --config compression.type= lz4 \
  --config max.message.bytes= 1048576

  ### Consumer configs

kubectl exec -it kafka-0 -- kafka-console-consumer \
  --bootstrap-server kafka-service:9092 \
  --topic digital-payments.payment.events.v1 \
  --property parse.key=true \
  --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
  --property value.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
  --group fraud-scoring-service \
  --property max.poll.records= 100 \
  --property session.timeout.ms=10000  \ 
  --property heartbeat.interval.ms=3000  \
  --property auto.offset.reset=earliest  \ 
  --property enable.auto.commit=true  \
  --property auto.commit.interval.ms=1000  \
  --property max.poll.interval.ms=30000  \
  --property fetch.min.bytes=1  \  
  --property fetch.max.wait.ms=10  \  
  --property max.partition.fetch.bytes=1048576  \ 
  --property partition.assignment.strategy=org.apache.kafka.clients.consumer.CooperativeStickyAssignor 


### Producer configs

kubectl exec -it kafka-0 -- kafka-console-producer \
  --bootstrap-server kafka-service:9092 \
  --topic digital-payments.fraud.score.events.v1 \
  --property parse.key=true \
  --property key.separator=: \
  --property key.serializer=org.apache.kafka.common.serialization.StringSerializer \
  --property value.serializer=org.apache.kafka.common.serialization.StringSerializer \
  --property acks=all \
  --property retries=2147483647  \
  --property max.in.flight.requests.per.connection=5  \
  --property enable.idempotence=true  \
  --property compression.type= lz4  \
  --property linger.ms=1  \
  --property batch.size=16384  \
  --property delivery.timeout.ms=10000  \
  --property request.timeout.ms=3000 