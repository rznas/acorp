# Change Data Capture and Streaming Techniques for Real-Time DW

## Overview

This reference details change data capture (CDC) and streaming techniques required for intra-day and real-time data warehouse architectures. Traditional batch-oriented CDC (covered in SKILL 10) assumes daily extracts. Real-time CDC requires continuous or micro-batch data capture with different technology and trade-offs.

---

## CDC Requirements for Real-Time/Intra-Day

<realtime_cdc_requirements>

Traditional daily batch CDC can use:
- Audit columns queried once per day
- Timed extracts at end of day
- Full diff compare overnight

Real-time/intra-day CDC requires:
- **Continuous or frequent capture** (seconds to minutes, not hours)
- **Event-driven notification** (push, not poll)
- **Low overhead on source system** (cannot impact OLTP performance)
- **Guaranteed delivery** (no lost events during failures)
- **Ordered delivery** (events processed in correct sequence)
- **Replay capability** (recover from ETL failures)

</realtime_cdc_requirements>

---

## CDC Method 1: Message Queue Monitoring (Recommended)

### Architecture

Source applications publish transactions to enterprise message queue/event stream. ETL subscribes and consumes events.

<message_queue_architecture>

```
[Source Application] --publish--> [Message Queue] --subscribe--> [ETL Consumer]
                                        |
                                        v
                                [Event Topics/Queues]
                                - customer.created
                                - order.placed
                                - payment.processed
                                - inventory.updated
```

**Message queue technologies**:
- Apache Kafka (distributed streaming platform)
- Amazon Kinesis (AWS managed streaming)
- Azure Event Hubs (Azure managed streaming)
- Google Pub/Sub (GCP managed messaging)
- RabbitMQ (traditional message broker)
- IBM MQ / JMS (enterprise message queue)

</message_queue_architecture>

### Implementation Pattern

**Source application publishes events**:

```java
// Source application code (example: order service)
@Transactional
public void createOrder(Order order) {
    // 1. Persist to source database
    orderRepository.save(order);

    // 2. Publish event to message queue
    OrderCreatedEvent event = new OrderCreatedEvent(
        order.getId(),
        order.getCustomerId(),
        order.getTotalAmount(),
        order.getTimestamp()
    );

    messageProducer.send("orders.created", event);
}
```

**ETL consumer subscribes and processes**:

```python
# ETL consumer code
from kafka import KafkaConsumer
import json

consumer = KafkaConsumer(
    'orders.created',
    bootstrap_servers=['kafka:9092'],
    value_deserializer=lambda m: json.loads(m.decode('utf-8')),
    group_id='dw-etl-consumer',
    auto_offset_commit=False  # Manual commit after processing
)

for message in consumer:
    event = message.value

    # 1. Column quality screens
    if not validate_order_event(event):
        log_error_event(event, "Column validation failed")
        continue

    # 2. Lookup surrogate keys (with provisional handling)
    customer_key = lookup_customer_key(event['customer_id'])
    if customer_key is None:
        customer_key = get_provisional_customer_key(event['customer_id'])

    # 3. Load to real-time partition
    load_to_realtime_partition(event, customer_key)

    # 4. Commit offset (acknowledge processed)
    consumer.commit()
```

### Advantages

- **Low latency**: Events available immediately after source transaction
- **Low overhead**: Source application already publishing events (no polling)
- **Event-driven**: ETL reacts to events, no wasted polling
- **Scalability**: Message queues handle high volume (millions/sec with Kafka)
- **Replay capability**: Kafka retains events for configurable retention period
- **Ordered delivery**: Partition keys ensure order within entity (e.g., all events for customer X in order)
- **Guaranteed delivery**: At-least-once or exactly-once semantics available

### Challenges

- **Requires event-driven architecture**: Source applications must publish events
- **No replay if connection lost without retention**: Configure adequate retention (e.g., 7 days)
- **Schema evolution**: Event schema changes require ETL compatibility handling
- **Requires message queue infrastructure**: Operational overhead

### Configuration Recommendations

**Kafka configuration for DW ETL**:

```properties
# Topic retention: Keep events for recovery (7 days)
retention.ms=604800000

# Partition strategy: Partition by entity ID for ordering
partitions=12
partition.key=customer_id  # or order_id, etc.

# Consumer group: Multiple ETL consumers for parallelism
group.id=dw-etl-consumer

# Offset management: Manual commit after successful load
enable.auto.commit=false

# Delivery semantics: At-least-once (idempotent load required)
# or exactly-once (requires transactions)
```

### When to Use

- Source applications support event-driven architecture
- High-volume, high-velocity data (millions of events/day)
- Latency requirement: Seconds to minutes
- Operational capability to run message queue infrastructure

---

## CDC Method 2: Database Log Scraping/Streaming

### Architecture

Capture changes from database transaction log (redo log, write-ahead log, binlog).

<log_scraping_architecture>

```
[Source Database] --> [Transaction Log] --> [Log Reader] --> [ETL Pipeline]
                           |
                           v
                    [Redo/WAL/Binlog]
```

**Technologies**:
- Oracle GoldenGate (Oracle)
- SQL Server Change Data Capture (SQL Server)
- MySQL binlog replication (MySQL)
- PostgreSQL logical replication (PostgreSQL)
- Debezium (open-source CDC connector for Kafka)
- AWS Database Migration Service (DMS) CDC
- Attunity/Qlik Replicate

</log_scraping_architecture>

### Implementation Pattern: Debezium + Kafka

**Debezium connector configuration**:

```json
{
  "name": "orders-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "database.hostname": "mysql-source",
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "secret",
    "database.server.id": "184054",
    "database.server.name": "orders-db",
    "table.whitelist": "sales.orders,sales.order_lines",
    "database.history.kafka.bootstrap.servers": "kafka:9092",
    "database.history.kafka.topic": "schema-changes.orders"
  }
}
```

**Event format** (Debezium change event):

```json
{
  "before": null,
  "after": {
    "order_id": 12345,
    "customer_id": 789,
    "order_date": "2024-12-15",
    "total_amount": 299.99,
    "status": "PLACED"
  },
  "source": {
    "version": "1.9.0",
    "connector": "mysql",
    "name": "orders-db",
    "ts_ms": 1702648800000,
    "snapshot": "false",
    "db": "sales",
    "table": "orders",
    "server_id": 223344,
    "gtid": null,
    "file": "mysql-bin.000003",
    "pos": 154,
    "row": 0
  },
  "op": "c",  // c=create, u=update, d=delete
  "ts_ms": 1702648800105
}
```

**ETL consumer**:

```python
from kafka import KafkaConsumer
import json

consumer = KafkaConsumer(
    'orders-db.sales.orders',
    bootstrap_servers=['kafka:9092'],
    value_deserializer=lambda m: json.loads(m.decode('utf-8'))
)

for message in consumer:
    event = message.value
    operation = event['op']

    if operation == 'c':  # Create
        row_data = event['after']
        insert_to_realtime_partition(row_data)

    elif operation == 'u':  # Update
        old_data = event['before']
        new_data = event['after']
        handle_update(old_data, new_data)

    elif operation == 'd':  # Delete
        row_data = event['before']
        handle_delete(row_data)
```

### Advantages

- **Captures all changes**: Regardless of application path (triggers, scripts, manual SQL)
- **No application changes required**: Works with legacy systems
- **Low latency**: Changes available immediately after commit
- **Complete change history**: INSERT, UPDATE, DELETE all captured

### Challenges

- **Complex setup**: Requires database configuration, permissions, log retention
- **Database-specific**: Different tools/techniques per RDBMS
- **Log may be purged**: During crises, DBAs may purge logs (losing events)
- **Vendor lock-in**: Commercial tools (GoldenGate, Attunity) expensive
- **Schema changes**: DDL changes require reconfiguration

### Database-Specific Configuration

**Oracle GoldenGate**:
```sql
-- Enable supplemental logging
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
ALTER TABLE sales.orders ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;

-- GoldenGate extract configuration
EXTRACT EXT_ORDERS
USERID ggadmin, PASSWORD secret
EXTTRAIL ./dirdat/lt
TABLE sales.orders;
TABLE sales.order_lines;
```

**SQL Server CDC**:
```sql
-- Enable CDC on database
EXEC sys.sp_cdc_enable_db;

-- Enable CDC on table
EXEC sys.sp_cdc_enable_table
    @source_schema = 'sales',
    @source_name = 'orders',
    @role_name = NULL,
    @supports_net_changes = 1;

-- Query changes
SELECT * FROM cdc.sales_orders_CT
WHERE __$start_lsn > @last_lsn;
```

**MySQL binlog**:
```ini
# my.cnf configuration
[mysqld]
log-bin=mysql-bin
binlog_format=ROW
binlog_row_image=FULL
server-id=1

# Replicate specific databases
binlog-do-db=sales
```

**PostgreSQL logical replication**:
```sql
-- Create publication
CREATE PUBLICATION orders_pub FOR TABLE sales.orders, sales.order_lines;

-- Create replication slot
SELECT * FROM pg_create_logical_replication_slot('dw_etl_slot', 'pgoutput');

-- Subscribe to changes (from ETL consumer)
```

### When to Use

- Cannot modify source applications to publish events
- Need to capture changes from all paths (applications, scripts, manual)
- Latency requirement: Seconds to minutes
- Operational capability to manage log-based CDC tools

---

## CDC Method 3: Database Triggers

### Architecture

Database triggers fire on INSERT/UPDATE/DELETE and write change records to staging table.

<trigger_architecture>

```
[Source Table] --trigger--> [CDC Staging Table] --poll--> [ETL]
    orders                      orders_cdc
```

</trigger_architecture>

### Implementation Pattern

**Create CDC staging table**:

```sql
CREATE TABLE orders_cdc (
    cdc_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    cdc_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    operation VARCHAR(10),  -- INSERT, UPDATE, DELETE
    order_id INTEGER,
    customer_id INTEGER,
    order_date DATE,
    total_amount DECIMAL(10,2),
    -- Mirror all columns from source table
    old_total_amount DECIMAL(10,2)  -- For UPDATE operations
);

CREATE INDEX idx_orders_cdc_ts ON orders_cdc(cdc_timestamp);
```

**Create triggers**:

```sql
-- INSERT trigger
CREATE TRIGGER orders_insert_trigger
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    INSERT INTO orders_cdc (operation, order_id, customer_id, order_date, total_amount)
    VALUES ('INSERT', NEW.order_id, NEW.customer_id, NEW.order_date, NEW.total_amount);
END;

-- UPDATE trigger
CREATE TRIGGER orders_update_trigger
AFTER UPDATE ON orders
FOR EACH ROW
BEGIN
    INSERT INTO orders_cdc (operation, order_id, customer_id, order_date,
                            total_amount, old_total_amount)
    VALUES ('UPDATE', NEW.order_id, NEW.customer_id, NEW.order_date,
            NEW.total_amount, OLD.total_amount);
END;

-- DELETE trigger
CREATE TRIGGER orders_delete_trigger
AFTER DELETE ON orders
FOR EACH ROW
BEGIN
    INSERT INTO orders_cdc (operation, order_id, customer_id, order_date, total_amount)
    VALUES ('DELETE', OLD.order_id, OLD.customer_id, OLD.order_date, OLD.total_amount);
END;
```

**ETL polling**:

```python
import time

last_cdc_id = get_last_processed_cdc_id()

while True:
    # Poll every 60 seconds
    time.sleep(60)

    # Fetch new changes
    changes = query("""
        SELECT * FROM orders_cdc
        WHERE cdc_id > %s
        ORDER BY cdc_id
        LIMIT 10000
    """, last_cdc_id)

    for change in changes:
        if change['operation'] == 'INSERT':
            insert_to_realtime_partition(change)
        elif change['operation'] == 'UPDATE':
            update_realtime_partition(change)
        elif change['operation'] == 'DELETE':
            handle_delete(change)

        last_cdc_id = change['cdc_id']

    save_checkpoint(last_cdc_id)
```

### Advantages

- **Guaranteed capture**: Every change captured
- **Simple to implement**: Standard SQL triggers
- **Low latency**: Changes captured immediately
- **Works across applications**: Any path that modifies table

### Challenges

- **Performance impact on OLTP**: Triggers add overhead to every transaction
- **Trigger management**: Can be disabled, dropped, or fail silently
- **CDC table growth**: Requires regular purging
- **Limited to single database**: Cannot capture across federated sources

### Performance Considerations

**Minimize trigger overhead**:

```sql
-- BAD: Complex logic in trigger
CREATE TRIGGER orders_insert_trigger
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    INSERT INTO orders_cdc ...;

    -- DON'T DO THIS IN TRIGGER:
    UPDATE customer_stats SET order_count = order_count + 1;
    SELECT * FROM product WHERE product_id = NEW.product_id;
END;

-- GOOD: Minimal logic
CREATE TRIGGER orders_insert_trigger
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    INSERT INTO orders_cdc (operation, order_id, ...)
    VALUES ('INSERT', NEW.order_id, ...);
    -- Just capture the change, nothing else
END;
```

**Purge CDC staging table**:

```sql
-- Run periodically (hourly or daily)
DELETE FROM orders_cdc
WHERE cdc_timestamp < DATE_SUB(NOW(), INTERVAL 7 DAY);
```

### When to Use

- Small to medium volume tables
- Source DBA approves trigger usage
- Cannot use message queues or log scraping
- Latency requirement: Minutes

---

## CDC Method 4: Audit Column Polling (Frequent)

### Architecture

Source table has last_modified_date column. ETL polls frequently (every 1-5 minutes).

<audit_column_polling>

```
[Source Table]                    [ETL Polling]
  orders
  - order_id                      Every 1-5 minutes:
  - customer_id                   SELECT * FROM orders
  - last_modified_date            WHERE last_modified_date > :last_poll_time
```

</audit_column_polling>

### Implementation Pattern

**Source table with audit columns**:

```sql
CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER,
    total_amount DECIMAL(10,2),
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_modified_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE INDEX idx_orders_modified ON orders(last_modified_date);
```

**ETL polling script**:

```python
import time
from datetime import datetime, timedelta

def poll_changes():
    last_poll_time = get_last_poll_time()

    while True:
        current_poll_time = datetime.now()

        # Query changes since last poll
        changes = query("""
            SELECT * FROM orders
            WHERE last_modified_date > %s
              AND last_modified_date <= %s
            ORDER BY last_modified_date
        """, last_poll_time, current_poll_time)

        for row in changes:
            process_change(row)

        # Update checkpoint
        save_last_poll_time(current_poll_time)
        last_poll_time = current_poll_time

        # Wait before next poll (5 minutes)
        time.sleep(300)
```

### Advantages

- **Simple**: No infrastructure required
- **Reliable**: If audit columns maintained by database triggers

### Challenges

- **Source system load**: Frequent polling queries impact OLTP
- **Cannot detect deletes**: Deleted rows not in table to query
- **Audit column reliability**: Must be maintained by ALL paths (triggers, not application)
- **Clock skew**: If using SYSDATE, time zones and clock differences cause issues

### Enhanced Pattern: Watermark Table

Track high-water mark to avoid rescanning:

```sql
-- Watermark tracking table
CREATE TABLE cdc_watermarks (
    source_table VARCHAR(100) PRIMARY KEY,
    last_modified_date TIMESTAMP,
    last_poll_time TIMESTAMP
);

-- Query pattern
SELECT * FROM orders
WHERE last_modified_date > (
    SELECT last_modified_date
    FROM cdc_watermarks
    WHERE source_table = 'orders'
)
ORDER BY last_modified_date;

-- Update watermark
UPDATE cdc_watermarks
SET last_modified_date = :max_last_modified,
    last_poll_time = CURRENT_TIMESTAMP
WHERE source_table = 'orders';
```

### When to Use

- Low to medium volume tables
- Source has reliable audit columns (maintained by triggers)
- Cannot use other CDC methods
- Latency requirement: 5-15 minutes acceptable

---

## Streaming ETL Pipeline Architecture

### Micro-Batch Processing Pattern

**Stream processing with micro-batches**:

```
[Message Queue] --> [Streaming Engine] --> [Micro-Batch] --> [Real-Time Partition]
    Kafka            Kafka Streams         Every 1-5 min      DW fact table
                     Apache Flink
                     Spark Streaming
```

**Apache Kafka Streams example**:

```java
StreamsBuilder builder = new StreamsBuilder();

// Input stream from Kafka topic
KStream<String, OrderEvent> orders = builder.stream("orders.created");

// Transform: Column quality screens
KStream<String, OrderEvent> validated = orders.filter(
    (key, order) -> validateOrder(order)
);

// Transform: Enrich with dimension lookups
KStream<String, OrderFact> enriched = validated.map(
    (key, order) -> {
        int customerKey = lookupCustomerKey(order.getCustomerId());
        int productKey = lookupProductKey(order.getProductId());
        return new OrderFact(order, customerKey, productKey);
    }
);

// Output: Write to database (micro-batch)
enriched.toTable("orders_fact_realtime", Produced.with(...));
```

**Apache Flink example**:

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// Read from Kafka
FlinkKafkaConsumer<OrderEvent> consumer = new FlinkKafkaConsumer<>(
    "orders.created",
    new OrderEventSchema(),
    properties
);

DataStream<OrderEvent> orders = env.addSource(consumer);

// Transform
DataStream<OrderFact> facts = orders
    .filter(order -> validateOrder(order))
    .map(order -> enrichWithDimensions(order));

// Window: Micro-batch every 5 minutes
facts.windowAll(TumblingProcessingTimeWindows.of(Time.minutes(5)))
     .apply(new BatchLoader());

env.execute();
```

### Lambda Architecture Pattern

Combine batch and streaming for accuracy + speed:

```
[Source Data]
    |
    +---> [Batch Layer] ------> [Batch View (Accurate)]
    |         Nightly             Static fact table
    |
    +---> [Speed Layer] ------> [Real-Time View (Fast)]
              Streaming            Real-time partition
                                        |
                                        v
                                  [Serving Layer]
                                  Union both views
```

**Implementation**:
- **Batch layer**: Traditional ETL (full quality screens, conforming)
- **Speed layer**: Streaming ETL (minimal quality, provisional)
- **Serving layer**: Query combines both, prefers batch when available

---

## Handling CDC Edge Cases

### Late-Arriving Events

**Problem**: Events arrive out of order (network delay, retry, etc.)

**Solution**: Window-based processing with allowed lateness

```java
// Flink example: Allow 1 hour lateness
DataStream<OrderEvent> orders = ...;

orders.windowAll(TumblingEventTimeWindows.of(Time.hours(1)))
      .allowedLateness(Time.hours(1))
      .apply(new BatchLoader());
```

### Duplicate Events

**Problem**: At-least-once delivery causes duplicate events

**Solution**: Idempotent loading with deduplication

```sql
-- Use UPSERT (INSERT ... ON CONFLICT UPDATE)
INSERT INTO orders_fact_realtime (order_key, customer_key, amount)
VALUES (:order_key, :customer_key, :amount)
ON CONFLICT (order_key) DO UPDATE
SET customer_key = EXCLUDED.customer_key,
    amount = EXCLUDED.amount;
```

### Schema Evolution

**Problem**: Event schema changes (new fields, renamed fields)

**Solution**: Schema registry + backward compatibility

```json
// Use Avro schema with Confluent Schema Registry
{
  "type": "record",
  "name": "OrderEvent",
  "namespace": "com.company.events",
  "fields": [
    {"name": "order_id", "type": "int"},
    {"name": "customer_id", "type": "int"},
    {"name": "total_amount", "type": "double"},
    {"name": "currency_code", "type": ["null", "string"], "default": null}
  ]
}
```

---

## Summary

Real-time CDC requires event-driven or log-based techniques. Message queue monitoring (Kafka, Kinesis) is recommended for high-volume, event-driven architectures. Database log scraping (Debezium, GoldenGate) works when source applications cannot be modified. Database triggers provide guaranteed capture with OLTP performance impact. Frequent audit column polling is simple but creates source system load. Streaming ETL frameworks (Kafka Streams, Flink, Spark Streaming) process events with micro-batch windows. Lambda architecture combines batch accuracy with streaming speed.
