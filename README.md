# Kafka

Apache Kafka is a distributed event streaming platform that follows a publish-subscribe model.

Key components:
Producers: Publish messages to Kafka topics.
Brokers: Kafka cluster nodes that store and manage topics. Muscle for cluster
Topics & Partitions: Data is divided into partitions for scalability.
Consumers & Consumer Groups: Consume messages from topics using consumer group with consumer id.
ZooKeeper (or KRaft): Manages metadata and leader election. Brain for cluster
Replication: Messages are copied across multiple brokers. It happens at each partition spread across brokers
Leader: Handles all read/write requests.
Followers: Replicate data from the leader and can take over if the leader fails.
Rebalance: When a new consumer joins, leaves, or fails, Kafka triggers a rebalance. A group coordinator assigns partitions to consumers. Consumers use sticky, cooperative, or eager rebalance strategies to optimize efficiency.
rack aware cluster
Kafka Controller: is a special broker that manages cluster-wide administrative operations, such as partition leadership, topic creation, and broker failures.

Configure: replication factor, ISR (In-Sync Replicas), batch.size, linger.ms, Retention Policies,Log Segments,acks, static Group Membership (group.instance.id), Cooperative Sticky Rebalance Strategy (partition.assignment.strategy, session.timeout.ms, heartbeat.interval.ms, unclean.leader.election.enable=false, log.retention.ms, log.segment.bytes , max.in.flight.requests.per.connection=1, compression.type, enable.idempotence=true, consumer fetch.min.bytes, consumer fetch.max.wait.ms, kafka streams max.poll.records, xactly-once processing (EOS) with transactions (transactional.id), enable.auto.commit=false, use commitSync, message.max.bytes on brokers and fetch.max.bytes on consumer

Alerting: Set up alerts for latency, under-replicated partitions, consumer lag.

Challenges in scaling Kafka in a DevOps environment?
partitions needs to be moved to newly added brokers
while scaling in terms of multiple ecs volume, existinsg partitions needs to be moved to newly added broker
while scaling partition, existing data needs to be moved to be newly added partitions.
If parition and consumers are not in sync then frequent rebalancing gets trigerred in.
Rebalancing overhead when adding partitions.
Handling out-of-order messages during failover.
Managing consumer lag in high-throughput scenarios.

Security:
Authentication: SSL/TLS, SASL (Kerberos, SCRAM).
Authorization: Role-based ACLs via Kafka's built-in ACL mechanism.
Encryption: SSL for data in transit.

Patterns:
Kafka as a Message Queue
Kafka for Multi-Tenant Architecture
Kafka for Event-Driven Architecture
- Microservices publish events to Kafka topics.
- Multiple consumers (services) process events asynchronously.
- Event replay is possible due to Kafka's message retention.
- Schema validation can be enforced using Apache Avro and Schema Registry.
- E-commerce: Order service publishes "Order Placed", which triggers inventory, payment, and shipping services.
- Banking: Transactions are processed as events, ensuring event-driven fraud detection.
Kafka as a Log Aggregation System
- Application logs are published to Kafka topics.
- Log processors consume logs and store them in Elasticsearch, S3, or HDFS.
- Real-time monitoring & alerting is enabled via tools like Elasticsearch + Kibana or Prometheus + Loki.
- Cloud platforms use Kafka to aggregate logs from thousands of microservices.
Kafka for Real-Time Data Processing
- Producers publish raw data (e.g., user clicks, sensor data).
- Kafka Streams or Flink processes the data (aggregations, filtering).
- Processed data is written back to Kafka or stored in databases (Elasticsearch, Redis, PostgreSQL).
- Use Case:
- Fraud detection: Process transactions in real-time to flag suspicious behavior.
- IoT analytics: Process sensor data for anomaly detection.
Kafka for Change Data Capture (CDC)
- Kafka can track database changes using Debezium or Kafka Connect:
- Debezium listens to database logs (MySQL, PostgreSQL, MongoDB).
- Kafka topics store CDC events (insert, update, delete).
- Consumers update downstream systems (Data Warehouse, Elasticsearch).
- Use Case:
- Synchronizing MySQL to Elasticsearch for real-time search.
- Streaming database changes to a Data Lake (Snowflake, BigQuery).
Kafka as a Metrics Pipeline
- Applications send metrics to Kafka topics (CPU usage, request counts).
- Kafka consumers process and store metrics in Prometheus, InfluxDB, or Graphite.
- Dashboards visualize real-time trends (Grafana, Kibana).
- Alerts trigger on anomalies (e.g., spike in error rates).
- Use Case:
- Application performance monitoring (APM): Real-time latency tracking.
Kafka for Replicating Data Across Data Centers
- Use MirrorMaker 2.0 to replicate topics between Kafka clusters.
- Optimize compression (compression.type=snappy) for efficient transfer.
- Use geo-aware consumers to avoid unnecessary cross-region traffic.
- Ensure idempotency if messages are reprocessed.
Kafka for Microservices Communication
- Microservices use Kafka as an event bus instead of HTTP APIs.
- Producers emit events, and multiple consumers process them independently.
- Message versioning & schema evolution is managed using Avro + Schema Registry.
- Dead Letter Queues (DLQ) handle failed events.
- Use Case:
- Decoupling services in an e-commerce system (order, payment, shipping).
- Event-driven notifications (user signup triggers welcome email & analytics).






