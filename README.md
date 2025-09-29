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

## Kafka Configuration

### Producer Configuration

- **acks** – Level of acknowledgment required from brokers before a producer request is considered successful.  
  *Use Case:* `all` for max reliability, `1` for lower latency.  
- **batch.size** – Maximum size (in bytes) of a batch of messages sent in one request.  
  *Use Case:* Increase to improve throughput for high-volume producers.  
- **linger.ms** – Time to wait before sending a batch to allow more messages to accumulate.  
  *Use Case:* Small value reduces latency, higher value improves batching.  
- **compression.type** – Algorithm (e.g., gzip, snappy, lz4) used to compress messages.  
  *Use Case:* Use snappy for faster compression, gzip for better ratio.  
- **enable.idempotence=true** – Ensures that messages are delivered exactly once without duplication.  
  *Use Case:* Required for exactly-once semantics in critical applications.  
- **max.in.flight.requests.per.connection=1** – Max number of unacknowledged requests per connection.  
  *Use Case:* Set to 1 with idempotence to prevent message reordering.  
- **transactional.id** – Enables transactional guarantees for producers.  
  *Use Case:* Use for exactly-once processing across multiple partitions or topics.  
- **retries** – Number of times the producer retries sending a message if it fails.  
  *Use Case:* Set higher for unreliable networks to ensure delivery.  
- **delivery.timeout.ms** – Maximum time to wait for a message acknowledgment before considering it failed.  
  *Use Case:* Increase in high-latency networks to prevent unnecessary failures.  

### Consumer Configuration

- **group.instance.id** – Fixed identifier for a consumer to maintain identity across rebalances.  
  *Use Case:* Use for static group membership to reduce rebalances in long-running consumers.  
- **enable.auto.commit=false** – Whether the consumer automatically commits offsets.  
  *Use Case:* Disable for manual, precise offset control in critical processing.  
- **auto.offset.reset** – Action if no initial offset exists (`earliest` or `latest`).  
  *Use Case:* `earliest` for batch jobs, `latest` for real-time streaming.  
- **fetch.min.bytes** – Minimum amount of data the consumer fetches in a request.  
  *Use Case:* Increase to reduce network calls in high-throughput scenarios.  
- **fetch.max.wait.ms** – Max wait time to accumulate `fetch.min.bytes`.  
  *Use Case:* Tune for latency vs throughput trade-off.  
- **fetch.max.bytes** – Maximum data fetched per request.  
  *Use Case:* Increase if messages are large to avoid truncation.  
- **max.poll.records** – Max records returned in a single poll.  
  *Use Case:* Lower value for low-latency processing, higher for bulk processing.  
- **session.timeout.ms** – Timeout to detect consumer failures.  
  *Use Case:* Short for fast failure detection, long for network instability.  
- **heartbeat.interval.ms** – Interval at which consumer sends heartbeats to the broker.  
  *Use Case:* Short interval ensures broker knows consumer is alive.  
- **max.partition.fetch.bytes** – Max data fetched per partition in a request.  
  *Use Case:* Prevents a single large partition from starving others.  

### Broker Configuration

- **replication.factor** – Number of copies of each partition across brokers.  
  *Use Case:* Higher value for fault-tolerant production clusters.  
- **log.retention.ms** – Duration Kafka retains logs before deletion.  
  *Use Case:* Tune for storage limits and regulatory retention policies.  
- **log.segment.bytes** – Maximum size of a log segment before rolling over.  
  *Use Case:* Smaller segments for fast recovery, larger for fewer files.  
- **log.retention.bytes** – Max total size of logs per partition.  
  *Use Case:* Use with disk quotas to prevent storage overflow.  
- **message.max.bytes** – Maximum size of a single message the broker will accept.  
  *Use Case:* Increase for large messages; match consumer `fetch.max.bytes`.  
- **unclean.leader.election.enable=false** – Prevents out-of-sync replicas from becoming leader.  
  *Use Case:* Recommended for high data integrity in production.  
- **num.partitions** – Default number of partitions for a topic if not specified.  
  *Use Case:* Set based on expected parallelism and throughput.  
- **min.insync.replicas** – Minimum number of replicas that must acknowledge a write for success.  
  *Use Case:* Ensures durability in replicated topics.  
- **replica.lag.time.max.ms** – Max time a follower can lag before being considered out-of-sync.  
  *Use Case:* Prevents slow replicas from being chosen as leaders.  
- **connections.max.idle.ms** – Max idle time for connections before closing.  
  *Use Case:* Frees resources in low-traffic clusters.  
- **replica.fetch.max.bytes** – Max bytes a replica can fetch per request from the leader.  
  *Use Case:* Tune for network bandwidth and large messages.  
- **log.cleaner.enable** – Enable log compaction.  
  *Use Case:* Use for topics requiring key-based deduplication (like changelog topics).  

### Miscellaneous / Cluster

- **ISR (In-Sync Replicas)** – Replicas fully caught up with the leader.  
  *Use Case:* Ensure sufficient replicas for high availability.  
- **partition.assignment.strategy** – Strategy used for assigning partitions to consumers.  
  *Use Case:* `cooperative-sticky` minimizes partition movement during rebalances.  
- **retention.policies** – Rules defining how long Kafka retains messages.  
  *Use Case:* Configure for regulatory or business requirements.  
- **leader.imbalance.check.interval.seconds** – Interval for checking partition leader distribution across brokers.  
  *Use Case:* Helps balance load across brokers automatically.  


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
 
### Handling Out-of-Order Messages During Failover

Kafka guarantees **in-order delivery per partition**, but failover can cause messages to be delivered out-of-order if an out-of-sync replica becomes the new leader. To handle this effectively:

#### Key Concepts

- **Sequence Number (`seq`)**
  - Assigned by the **producer** per partition.
  - Ensures **idempotence**: broker ignores duplicate messages on retries.
  - Independent of broker offsets.

- **Offset**
  - Assigned by the **broker** per partition.
  - Tracks the **position of a message in the log** for consumers.
  - Consumers read messages in **offset order**.

- **ISR (In-Sync Replicas)**
  - Only replicas fully caught up with the leader can become the new leader.
  - Ensures messages aren’t lost during failover.

- **Transactional Producers (`transactional.id`)**
  - Guarantees **exactly-once semantics** across multiple partitions/topics.
  - Works with idempotent producers to maintain order even during failover.

---

#### How It Works

1. Producer sends messages with **PID + Seq #**.  
2. Broker assigns an **offset** when the message is appended to the log.  
3. If the **leader fails**:  
   - Only replicas in **ISR** can become the new leader.  
   - Out-of-sync replicas are prevented from causing message loss or out-of-order delivery (`unclean.leader.election.enable=false`).  
4. Producer retries a message after a transient error:  
   - Broker checks **PID + Seq #**.  
   - If the message was already written, it is **ignored** (no duplicate).  
5. Consumers read messages in **offset order**, preserving correct sequence.

---

#### Example: Sequence Number vs Offset

| Partition | Producer | Seq # | Broker Offset | Notes |
|-----------|----------|-------|---------------|-------|
| 2         | A        | 0     | 0             | First message appended |
| 2         | A        | 1     | 1             | Next message |
| 2         | B        | 0     | 2             | New producer starts seq=0 |
| 2         | A        | 2     | 3             | Continues sending |
| 2         | B        | 1     | 4             | Continues sending |

**Explanation:**  
- Sequence numbers are **per producer per partition** and help the broker detect duplicates.  
- Offsets are **assigned by the broker** and determine the order seen by consumers.  
- Even during failover, messages remain **deduplicated and ordered** if idempotence and transactions are used.  

---

#### Practical Config Recommendations

- **Producer Side:**  
  - `enable.idempotence=true` → Prevent duplicates per partition.  
  - `transactional.id=<unique_id>` → Exactly-once across multiple partitions/topics.  
  - `acks=all` and `min.insync.replicas >= 2` → Ensure messages are committed before acknowledgment.  

- **Broker Side:**  
  - `unclean.leader.election.enable=false` → Prevent out-of-sync leaders.  
  - Monitor **ISR** size to ensure high availability.  

- **Partitioning Strategy:**  
  - Keep related messages in the **same partition** to preserve order.  

---

### Kafka Configuration & Scenarios for Ordered, Sequential Processing

Kafka guarantees **in-order delivery per partition**. This README consolidates **producer, consumer, broker settings**, sequential processing examples, failover handling, idempotence, transactions, and common real-world scenarios.

```properties
# -----------------------------
# Producer Settings
# -----------------------------
enable.idempotence=true               # Prevent duplicates per partition
acks=all                              # Wait for all in-sync replicas to acknowledge
transactional.id=my-transaction       # Exactly-once across multiple partitions/topics
max.in.flight.requests.per.connection=1  # Maintain strict order
batch.size=16384                       # Batch multiple messages for throughput
linger.ms=5                            # Small delay to allow batching
compression.type=snappy                # Reduce network overhead
# Use a consistent key (e.g., userId) for related messages to ensure they go to the same partition

# -----------------------------
# Consumer Settings
# -----------------------------
enable.auto.commit=false              # Manual offset commits to prevent duplicates
max.poll.records=100                  # Fetch multiple messages per poll
fetch.max.wait.ms=500                 # Max wait time to fill batch
max.partition.fetch.bytes=1048576     # Max data per partition
session.timeout.ms=30000              # Detect consumer failures quickly
heartbeat.interval.ms=10000           # Heartbeat to broker to maintain session
isolation.level=read_committed        # Ensure consumer reads only committed transactional messages

# -----------------------------
# Broker / Cluster Settings
# -----------------------------
replication.factor=3                   # Number of replicas per partition for fault tolerance
num.partitions=1                       # Single partition for strict global ordering
min.insync.replicas=2                  # Minimum replicas that must acknowledge a write
log.retention.ms=604800000             # Retain messages for 7 days
log.segment.bytes=1073741824           # 1 GB per log segment
message.max.bytes=10485760             # Max message size 10 MB
unclean.leader.election.enable=false   # Prevent out-of-sync replicas from becoming leaders
# Monitor ISR to ensure high availability without losing ordering guarantees

# -----------------------------
# Producing Messages Sequentially
# -----------------------------
# Pseudocode:
# producer.init_transactions()
# producer.begin_transaction()
# for message in messages:         # e.g., 100 messages
#     producer.send(topic, key=message.key, value=message.value)
# producer.commit_transaction()    # Commit all messages atomically

# -----------------------------
# Consuming Messages Sequentially
# -----------------------------
# Pseudocode:
# while True:
#     records = consumer.poll(timeout_ms=500)
#     for record in records:
#         process(record)          # Sequential processing logic
#     consumer.commit()            # Commit offsets after processing

# -----------------------------
# PID + Sequence Number vs Broker Offset vs Consumer Read Order
# -----------------------------
# Example for Partition 0
# +-------------------+-------------------+---------------------+
# | Producer PID + Seq | Broker Offset     | Consumer Read Order |
# +-------------------+-------------------+---------------------+
# | PID=1001, Seq=0   | Offset=0          | Read first          |
# | PID=1001, Seq=1   | Offset=1          | Read second         |
# | PID=1001, Seq=2   | Offset=2          | Read third          |
# | PID=1002, Seq=0   | Offset=3          | Read fourth         |
# | PID=1002, Seq=1   | Offset=4          | Read fifth          |
# +-------------------+-------------------+---------------------+
# Sequence numbers are per producer; offsets are per broker partition.
# Consumers always read in offset order, preserving message order.

# -----------------------------
# Common Real-World Scenarios & Best Practices
# -----------------------------

# 1. Consumer sees the same message twice
# Reason: Consumer crashes or fails before committing offsets.
# Solution: Make processing idempotent, commit offsets after processing.

# 2. Messages arrive out-of-order at consumer
# Reason: Multiple partitions; consumer reads in parallel.
# Solution: Use a single partition or consistent key for related messages.

# 3. Messages lost during failover
# Reason: Unclean leader election or insufficient ISR.
# Solution: unclean.leader.election.enable=false, min.insync.replicas>=2, acks=all.

# 4. Duplicate messages due to producer retries
# Reason: Non-idempotent producer retries on transient failures.
# Solution: enable.idempotence=true, use transactions for multi-partition writes.

# 5. Consumer processes messages but app fails
# Reason: Offsets not committed after processing.
# Solution: Commit offsets **after successful processing**, use idempotent processing.

# 6. Producer faster than consumer
# Reason: Consumer fetch batch/processing limits.
# Solution: Increase consumer parallelism, adjust fetch.min.bytes, max.poll.records, and monitor consumer lag.

# 7. Out-of-order messages after rebalance
# Reason: Consumer group rebalance reassigns partitions.
# Solution: Commit offsets frequently, ensure idempotent processing logic.

# 8. Transactional producer partially succeeds
# Reason: Transaction aborts due to error.
# Solution: Consumers use isolation.level=read_committed to only read committed messages.

# -----------------------------
# Best Practices Summary
# -----------------------------
# - Use single partition or partition key for ordering.
# - Enable idempotence and/or transactions for producer.
# - Commit consumer offsets after processing.
# - Disable unclean leader election.
# - Monitor ISR to ensure high availability.
# - Batch messages and use compression for throughput.
# - Design consumers to be idempotent to handle duplicates.

----

# -----------------------------
# Kafka Internal Components
# -----------------------------
# 1. Controller
#    - Elects a single broker as the "controller"
#    - Responsible for partition leadership election, topic creation/deletion
#    - Tracks cluster metadata and manages broker coordination

# 2. Group Coordinator
#    - One broker acts as the coordinator per consumer group
#    - Manages consumer group membership, offsets, and rebalances

# 3. Offset Management
#    - __consumer_offsets (internal topic)
#      - Stores committed offsets for all consumer groups
#      - Enables consumers to resume from last committed position after crash/restart

# 4. Transactions Management
#    - __transaction_state (internal topic)
#      - Tracks transactional producer state
#      - Ensures atomic commits or aborts across partitions/topics
#    - Producers write messages within transaction boundaries to guarantee exactly-once semantics

# 5. Cluster Metadata
#    - __cluster_metadata (internal, optional in newer versions)
#      - Stores metadata about brokers, partitions, and topics
#      - Used for fast leader lookup and routing

# 6. Internal Replication & ISR
#    - Leader replica handles all writes
#    - Followers replicate messages to maintain in-sync replicas
#    - ISR ensures replicas are up-to-date; only in-sync replicas can become leaders

# -----------------------------
# Common Real-World Scenarios & Best Practices
# -----------------------------
# 1. Consumer sees the same message twice
#    - Reason: Consumer crashes or fails before committing offsets
#    - Solution: Make processing idempotent, commit offsets after processing

# 2. Messages lost during failover
#    - Reason: Unclean leader election or insufficient ISR
#    - Solution: unclean.leader.election.enable=false, min.insync.replicas>=2, acks=all

# 3. Duplicate messages due to producer retries
#    - Reason: Non-idempotent producer retries on transient failures
#    - Solution: enable.idempotence=true, use transactions for multi-partition writes

# 4. Consumer crash or processing failure
#    - Reason: Offsets not committed after processing
#    - Solution: Commit offsets only after successful processing, use idempotent processing logic

# 5. Producer faster than consumer
#    - Reason: Consumer fetch batch/processing limits
#    - Solution: Increase consumer parallelism, adjust fetch.min.bytes, max.poll.records, and monitor consumer lag

# 6. Out-of-order delivery after rebalance
#    - Reason: Consumer group rebalance reassigns partitions
#    - Solution: Commit offsets frequently, ensure idempotent processing logic

# 7. Transactional producer aborts
#    - Reason: Transaction fails
#    - Solution: Consumers use isolation.level=read_committed to only read committed messages
