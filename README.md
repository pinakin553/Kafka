# Apache Kafka

Apache Kafka is a distributed event streaming platform that follows a publish-subscribe model.

## Key Components

- **Producers**: Publish messages to Kafka topics.
- **Brokers**: Kafka cluster nodes that store and manage topics. They serve as the muscle for the cluster.
- **Topics & Partitions**: Data is divided into partitions for scalability.
- **Consumers & Consumer Groups**: Consume messages from topics using consumer group with consumer id.
- **ZooKeeper (or KRaft)**: Manages metadata and leader election. Acts as the brain for the cluster.
- **Replication**: Messages are copied across multiple brokers, with replication happening at each partition spread across brokers.
- **Leader**: Handles all read/write requests.
- **Followers**: Replicate data from the leader and can take over if the leader fails.
- **Rebalance**: When a new consumer joins, leaves, or fails, Kafka triggers a rebalance. A group coordinator assigns partitions to consumers. Consumers use sticky, cooperative, or eager rebalance strategies to optimize efficiency.
- **Rack Aware Cluster**: Ensures high availability and fault tolerance by spreading replicas across different racks.
- **Kafka Controller**: A special broker that manages cluster-wide administrative operations, such as partition leadership, topic creation, and broker failures.

## Configuration

- **Replication Factor**
- **ISR (In-Sync Replicas)**
- **Batch Size (`batch.size`)**
- **Linger Time (`linger.ms`)**
- **Retention Policies**
- **Log Segments**
- **Acknowledgments (`acks`)**
- **Static Group Membership (`group.instance.id`)**
- **Cooperative Sticky Rebalance Strategy (`partition.assignment.strategy`, `session.timeout.ms`, `heartbeat.interval.ms`, `unclean.leader.election.enable=false`)**
- **Log Retention (`log.retention.ms`)**
- **Log Segment Bytes (`log.segment.bytes`)**
- **Max In-Flight Requests Per Connection (`max.in.flight.requests.per.connection=1`)**
- **Compression Type (`compression.type`)**
- **Enable Idempotence (`enable.idempotence=true`)**
- **Consumer Fetch Min Bytes (`fetch.min.bytes`)**
- **Consumer Fetch Max Wait Time (`fetch.max.wait.ms`)**
- **Kafka Streams Max Poll Records (`max.poll.records`)**
- **Exactly-Once Processing (EOS) with Transactions (`transactional.id`)**
- **Auto Commit (`enable.auto.commit=false`)**
- **Commit Sync (`commitSync`)**
- **Message Max Bytes on Brokers (`message.max.bytes`)**
- **Fetch Max Bytes on Consumer (`fetch.max.bytes`)**

## Alerting

Set up alerts for:

- Latency
- Under-replicated partitions
- Consumer lag

## Challenges in Scaling Kafka in a DevOps Environment

- Partitions need to be moved to newly added brokers.
- While scaling in terms of multiple ECS volumes, existing partitions need to be moved to newly added brokers.
- While scaling partitions, existing data needs to be moved to newly added partitions.
- Frequent rebalancing triggered if partitions and consumers are not in sync.
- Rebalancing overhead when adding partitions.
- Handling out-of-order messages during failover.
- Managing consumer lag in high-throughput scenarios.

## Security

- **Authentication**: SSL/TLS, SASL (Kerberos, SCRAM).
- **Authorization**: Role-based ACLs via Kafka's built-in ACL mechanism.
- **Encryption**: SSL for data in transit.

## Patterns

### Kafka as a Message Queue

### Kafka for Multi-Tenant Architecture

### Kafka for Event-Driven Architecture

- Microservices publish events to Kafka topics.
- Multiple consumers (services) process events asynchronously.
- Event replay is possible due to Kafka's message retention.
- Schema validation can be enforced using Apache Avro and Schema Registry.
- **E-commerce**: Order service publishes "Order Placed", which triggers inventory, payment, and shipping services.
- **Banking**: Transactions are processed as events, ensuring event-driven fraud detection.

### Kafka as a Log Aggregation System

- Application logs are published to Kafka topics.
- Log processors consume logs and store them in Elasticsearch, S3, or HDFS.
- Real-time monitoring & alerting is enabled via tools like Elasticsearch + Kibana or Prometheus + Loki.
- Cloud platforms use Kafka to aggregate logs from thousands of microservices.

### Kafka for Real-Time Data Processing

- Producers publish raw data (e.g., user clicks, sensor data).
- Kafka Streams or Flink processes the data (aggregations, filtering).
- Processed data is written back to Kafka or stored in databases (Elasticsearch, Redis, PostgreSQL).
- **Use Case**:
  - **Fraud detection**: Process transactions in real-time to flag suspicious behavior.
  - **IoT analytics**: Process sensor data for anomaly detection.

### Kafka for Change Data Capture (CDC)

- Kafka can track database changes using Debezium or Kafka Connect:
  - Debezium listens to database logs (MySQL, PostgreSQL, MongoDB).
  - Kafka topics store CDC events (insert, update, delete).
  - Consumers update downstream systems (Data Warehouse, Elasticsearch).
- **Use Case**:
  - Synchronizing MySQL to Elasticsearch for real-time search.
  - Streaming database changes to a Data Lake (Snowflake, BigQuery).

### Kafka as a Metrics Pipeline

- Applications send metrics to Kafka topics (CPU usage, request counts).
- Kafka consumers process and store metrics in Prometheus, InfluxDB, or Graphite.
- Dashboards visualize real-time trends (Grafana, Kibana).
- Alerts trigger on anomalies (e.g., spike in error rates).
- **Use Case**:
  - Application performance monitoring (APM): Real-time latency tracking.

### Kafka for Replicating Data Across Data Centers

- Use MirrorMaker 2.0 to replicate topics between Kafka clusters.
- Optimize compression (`compression.type=snappy`) for efficient transfer.
- Use geo-aware consumers to avoid unnecessary cross-region traffic.
- Ensure idempotency if messages are reprocessed.

### Kafka for Microservices Communication

- Microservices use Kafka as an event bus instead of HTTP APIs.
- Producers emit events, and multiple consumers process them independently.
- Message versioning & schema evolution is managed using Avro + Schema Registry.
- Dead Letter Queues (DLQ) handle failed events.
- **Use Case**:
  - Decoupling services in an e-commerce system (order, payment, shipping).
  - Event-driven notifications (user signup triggers welcome email & analytics).
