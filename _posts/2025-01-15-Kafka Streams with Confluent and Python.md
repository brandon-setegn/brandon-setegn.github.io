---
title:  "Spec-Driven Development with OpenSpec: A Kafka Example"
date:   2025-01-15 20:00:00 -0500
categories: [kafka, streaming, python, openspec]
mermaid: true
tags: [openspec, spec-driven development, kafka, python, streaming, data engineering]
image:
  path: /assets/img/kafka-streams-confluent.png
---

[OpenSpec](https://intent-driven.dev/knowledge/openspec/) is a lightweight approach to spec-driven development that keeps a single, unified specification as the source of truth for your system. It improves clarity before coding, traceability from requirement to implementation, and alignment between humans and AI assistants. In this post we'll use OpenSpec to drive a small project: a local [Apache Kafka](https://kafka.apache.org/) producer and consumer in Python. The code still demonstrates Kafka and the [Confluent Python client](https://docs.confluent.io/kafka-clients/python/current/overview.html); the focus here is on how OpenSpec shapes the work and the benefits you get from it.

> **Source code and OpenSpec:** The example project lives in this repo at [kafka-openspec-example](kafka-openspec-example/). All code blocks labeled "From the example" are verbatim from that folder.

## What is OpenSpec?

OpenSpec treats **specifications as the source of truth**. Instead of scattering design across tickets, wikis, and ad-hoc docs, you maintain a living spec that evolves with the codebase. The workflow is **propose → design → implement → archive**: you capture *what* should change and *why*, then implement, then archive the change so you have an audit trail.

Two folders structure the work:

- **specs/** — The current state of the system. Each spec describes a capability (e.g. local Kafka, Python clients) with requirements and scenarios in a consistent format.
- **changes/** — Proposed deltas. A change typically has a proposal (why and what), a design (decisions and trade-offs), and a task list. When done, the change is archived so the history is preserved.

Benefits:

- **Clarity before coding** — You spell out requirements and scenarios up front, so implementation has a clear target.
- **Traceability** — You can trace from a scenario in a spec to the code that fulfills it, and from a task list to the files that were added or changed.
- **Better alignment** — Humans and AI assistants share the same reference (the specs and change docs), which reduces drift between intent and code.
- **Lighter than heavy design docs** — OpenSpec aims for enough structure to guide work without drowning in process. It works with Cursor, Claude Code, and other AI coding tools.

No code lives in the specs themselves; they describe *what* the system shall do. The code lives in the repo and is tied to the spec via the change workflow.

## Project Overview: kafka-openspec-example

The example project exists to support **local Kafka experimentation and learning**, and it was built using OpenSpec. You can find it in this repository under the folder **kafka-openspec-example**.

High-level structure (from the example):

- **Root:** `producer.py`, `consumer.py`, `docker-compose.yml`, `requirements.txt`, `.env.example`, `README.md`
- **openspec/:** `project.md`, `specs/` (current capabilities), `changes/archive/` (completed changes)

The project runs a Kafka broker locally via Docker Compose and provides Python producer and consumer clients that send and receive JSON messages. There is no Confluent Cloud or Schema Registry in this example—everything runs against a local broker.

## OpenSpec in Action

The following sections use only artifacts from the kafka-openspec-example project. Each is labeled so you know it comes from the repo.

### Project context (from the example: openspec/project.md)

The project's OpenSpec **project context** defines purpose, conventions, and domain so that every change stays aligned.

**From the example** (`kafka-openspec-example/openspec/project.md`):

Purpose: a testing and experimentation project for Apache Kafka—producer/consumer functionality, configurations, patterns, and reusable examples. Domain terms include **Kafka Topics** (named channels for messages), **Producers** (publish to topics), **Consumers** (read from topics), **Consumer Groups**, **Partitions**, and **Brokers**. Conventions cover code style (e.g. PEP 8 for Python), architecture (separate producer and consumer modules, config for connection settings, error handling), testing, and Git workflow. Constraints include: broker must be accessible, serialization format and connection failures must be considered.

### Specs as requirements

Specs are written as requirements with **WHEN/THEN/AND** scenarios. Two specs in the example describe the current system.

**From the example** (`kafka-openspec-example/openspec/specs/local-kafka/spec.md`):

The **Local Kafka** spec says the project SHALL provide a Docker Compose configuration to run a Kafka broker locally. Example scenarios:

- **Start local Kafka:** WHEN a developer runs `docker-compose up` THEN Zookeeper and Kafka start, Kafka is accessible on port 9092, and services are healthy.
- **Stop:** WHEN a developer runs `docker-compose down` THEN services stop and containers are removed.
- **Connect:** WHEN a client connects to `localhost:9092` THEN the connection succeeds and the client can produce and consume messages.
- **Persist:** WHEN services are stopped and restarted THEN topics and messages are preserved via Docker volumes.

**From the example** (`kafka-openspec-example/openspec/specs/python-kafka-clients/spec.md`):

The **Python Kafka Clients** spec says the project SHALL provide a Python producer and consumer for the local broker, with dependency management and JSON serialization. It includes scenarios such as: producer connects to localhost:9092 and sends messages to a topic with delivery confirmation; producer serializes Python dicts to JSON; consumer connects, subscribes, reads messages, deserializes JSON, and uses a consumer group; both raise appropriate errors when the broker is unreachable. Another requirement: `pip install -r requirements.txt` installs the needed packages (e.g. confluent-kafka), and Python 3.8+ is supported.

### A change from idea to code

The Python clients were added via a single change. Here we walk through that change using only the example’s archived artifacts.

**Proposal** (from the example: `openspec/changes/archive/2026-01-30-add-python-kafka-clients/proposal.md`):

- **Why:** Enable programmatic interaction with the local Kafka broker for testing, experimentation, and development. Python is popular and has good library support.
- **What changes:** Python project structure and `requirements.txt`; a Kafka producer client and a consumer client; example usage and docs; configuration to use localhost:9092.
- **Impact:** New spec `python-kafka-clients`; new files `producer.py`, `consumer.py`, `requirements.txt`; dependencies Python 3.x and a Kafka client library (e.g. confluent-kafka). No breaking changes.

**Design** (from the example: `openspec/changes/archive/2026-01-30-add-python-kafka-clients/design.md`):

- **Library:** Use `confluent-kafka` (performance, documentation, maintenance; alternatives considered: kafka-python, aiokafka).
- **Serialization:** JSON for simplicity and readability.
- **Layout:** Separate `producer.py` and `consumer.py` for clear separation of concerns.
- **Configuration:** Environment variables or hardcoded defaults (localhost:9092) to match Docker Compose.
- **Non-goals:** No Schema Registry, no production-grade monitoring, no custom serialization in this change.

**Tasks** (from the example: `openspec/changes/archive/2026-01-30-add-python-kafka-clients/tasks.md`):

The task list maps directly to what was implemented: project setup (requirements.txt, Python version, README venv instructions); producer implementation (producer.py with create_producer, produce_message, JSON serialization, localhost:9092, error handling, `if __name__ == '__main__'`); consumer implementation (consumer.py with create_consumer, consume_messages, JSON deserialization, consumer group, error handling, main block); documentation (README setup, usage, prerequisites); and validation (connect, produce, consume, end-to-end, error when broker down). Each of these tasks corresponds to the code and docs you see in the repo.

## Code from the Example

Every code block below is a direct quote from the kafka-openspec-example project. Labels use the path under that folder.

### Local Kafka (from the example: docker-compose.yml)

**From the example** (`kafka-openspec-example/docker-compose.yml`):

```yaml
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "${ZOOKEEPER_PORT:-2181}:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    volumes:
      - zookeeper-data:/var/lib/zookeeper/data
      - zookeeper-logs:/var/lib/zookeeper/log
    healthcheck:
      test: ["CMD", "nc", "-z", "localhost", "2181"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - kafka-network

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    hostname: kafka
    container_name: kafka
    depends_on:
      zookeeper:
        condition: service_healthy
    ports:
      - "${KAFKA_PORT:-9092}:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
    volumes:
      - kafka-data:/var/lib/kafka/data
    healthcheck:
      test: ["CMD", "kafka-broker-api-versions", "--bootstrap-server", "localhost:9092"]
      interval: 10s
      timeout: 10s
      retries: 5
    networks:
      - kafka-network

volumes:
  zookeeper-data:
  zookeeper-logs:
  kafka-data:

networks:
  kafka-network:
    driver: bridge
```

This satisfies the Local Kafka spec: `docker-compose up` brings up Zookeeper and Kafka; the broker is on port 9092; healthchecks and volumes provide resilience and persistence.

### Producer (from the example: producer.py)

The producer uses the Confluent Kafka Python client to send JSON messages to a topic. Broker address comes from the environment (default localhost:9092).

**From the example** (`kafka-openspec-example/producer.py`) — create_producer and produce_message:

```python
def create_producer(bootstrap_servers=None):
    """
    Create and return a Kafka producer instance.
    
    Args:
        bootstrap_servers: Kafka broker address (default: localhost:9092)
    
    Returns:
        Producer instance configured to connect to the broker
    """
    if bootstrap_servers is None:
        bootstrap_servers = os.getenv('KAFKA_BOOTSTRAP_SERVERS', 'localhost:9092')
    
    config = {
        'bootstrap.servers': bootstrap_servers,
    }
    
    try:
        producer = Producer(config)
        return producer
    except Exception as e:
        raise KafkaException(f"Failed to create producer: {e}")


def produce_message(producer, topic, value, key=None):
    """
    Produce a message to a Kafka topic.
    
    Args:
        producer: Kafka Producer instance
        topic: Topic name to send message to
        value: Message value (dict or JSON-serializable object)
        key: Optional message key (string)
    
    Returns:
        None (raises exception on failure)
    """
    try:
        # Serialize value to JSON if it's a dict or other JSON-serializable object
        if isinstance(value, dict) or not isinstance(value, (str, bytes)):
            value_json = json.dumps(value)
            value_bytes = value_json.encode('utf-8')
        elif isinstance(value, str):
            value_bytes = value.encode('utf-8')
        else:
            value_bytes = value
        
        # Encode key if provided
        key_bytes = key.encode('utf-8') if key and isinstance(key, str) else key
        
        # Produce message
        producer.produce(topic, value=value_bytes, key=key_bytes, callback=delivery_callback)
        
        # Wait for message to be delivered (flush)
        producer.poll(0)
        producer.flush()
        
    except Exception as e:
        raise KafkaException(f"Failed to produce message: {e}")
```

**From the example** (`kafka-openspec-example/producer.py`) — delivery_callback and main:

```python
def delivery_callback(err, msg):
    """
    Callback function to handle message delivery status.
    
    Args:
        err: Error object if delivery failed, None otherwise
        msg: Message object
    """
    if err is not None:
        print(f'Message delivery failed: {err}')
        raise KafkaException(f"Message delivery failed: {err}")
    else:
        ts_type, ts_ms = msg.timestamp()
        ts_str = f' (timestamp: {ts_ms})' if ts_type and ts_ms else ''
        print(f'Message delivered to {msg.topic()} [{msg.partition()}] at offset {msg.offset()}{ts_str}')


def main():
    """Example usage of the producer."""
    try:
        # Create producer
        producer = create_producer()
        print("Producer created successfully")
        
        # Example: Send a message (Kafka attaches CreateTime timestamp automatically)
        topic = os.getenv('KAFKA_TOPIC', 'test-topic')
        message = {'message': 'Hello, Kafka!'}
        
        print(f"Producing message to topic '{topic}': {message}")
        produce_message(producer, topic, message)
        print("Message produced successfully")
        
    except KafkaException as e:
        print(f"Kafka error: {e}", file=sys.stderr)
        sys.exit(1)
    except Exception as e:
        print(f"Unexpected error: {e}", file=sys.stderr)
        sys.exit(1)


if __name__ == '__main__':
    main()
```

### Consumer (from the example: consumer.py)

The consumer subscribes to a topic, reads messages, and deserializes JSON. It uses a consumer group and supports a configurable max message count.

**From the example** (`kafka-openspec-example/consumer.py`) — create_consumer:

```python
def create_consumer(bootstrap_servers=None, group_id=None):
    """
    Create and return a Kafka consumer instance.
    
    Args:
        bootstrap_servers: Kafka broker address (default: localhost:9092)
        group_id: Consumer group ID (default: python-consumer-group)
    
    Returns:
        Consumer instance configured to connect to the broker
    """
    if bootstrap_servers is None:
        bootstrap_servers = os.getenv('KAFKA_BOOTSTRAP_SERVERS', 'localhost:9092')
    
    if group_id is None:
        group_id = os.getenv('KAFKA_CONSUMER_GROUP', 'python-consumer-group')
    
    config = {
        'bootstrap.servers': bootstrap_servers,
        'group.id': group_id,
        'auto.offset.reset': 'earliest',  # Start from beginning if no offset exists
    }
    
    try:
        consumer = Consumer(config)
        return consumer
    except Exception as e:
        raise KafkaException(f"Failed to create consumer: {e}")
```

**From the example** (`kafka-openspec-example/consumer.py`) — consume_messages:

```python
def consume_messages(consumer, topics, timeout=1.0, max_messages=None):
    """
    Consume messages from Kafka topics.
    
    Args:
        consumer: Kafka Consumer instance
        topics: List of topic names to subscribe to, or single topic name
        timeout: Timeout in seconds for polling (default: 1.0)
        max_messages: Maximum number of messages to consume (None for unlimited)
    
    Yields:
        Tuple of (topic, partition, offset, timestamp_ms, deserialized_message_dict).
        timestamp_ms is from Kafka (CreateTime or LogAppendTime), or None if not available.
    """
    # Convert single topic to list
    if isinstance(topics, str):
        topics = [topics]
    
    try:
        # Subscribe to topics
        consumer.subscribe(topics)
        print(f"Subscribed to topics: {topics}")
        
        message_count = 0
        
        while True:
            # Check if we've reached max messages
            if max_messages is not None and message_count >= max_messages:
                break
            
            # Poll for messages
            msg = consumer.poll(timeout=timeout)
            
            if msg is None:
                continue
            
            if msg.error():
                if msg.error().code() == KafkaError._PARTITION_EOF:
                    # End of partition event (not an error)
                    continue
                else:
                    raise KafkaException(f"Consumer error: {msg.error()}")
            
            # Deserialize message value from JSON
            # Handle tombstone messages (None value)
            msg_value = msg.value()
            if msg_value is None:
                # Tombstone message (null value)
                value_dict = None
            else:
                try:
                    value_str = msg_value.decode('utf-8')
                    value_dict = json.loads(value_str)
                except (UnicodeDecodeError, json.JSONDecodeError, AttributeError) as e:
                    # If not JSON or decode fails, return as string
                    try:
                        value_dict = {'raw_value': msg_value.decode('utf-8', errors='replace')}
                    except AttributeError:
                        # Fallback if value is not bytes-like
                        value_dict = {'raw_value': str(msg_value)}
            
            message_count += 1
            
            # Kafka timestamp: (timestamp_type, timestamp_ms); type 1=CreateTime, 2=LogAppendTime
            ts_type, ts_ms = msg.timestamp()
            timestamp_ms = ts_ms if ts_type else None
            
            yield (msg.topic(), msg.partition(), msg.offset(), timestamp_ms, value_dict)
            
    except KeyboardInterrupt:
        print("\nConsumer interrupted by user")
    except KafkaException as e:
        raise
    finally:
        consumer.close()
```

**From the example** (`kafka-openspec-example/consumer.py`) — main:

```python
def main():
    """Example usage of the consumer."""
    try:
        consumer = create_consumer()
        print("Consumer created successfully")
        
        topic = os.getenv('KAFKA_TOPIC', 'test-topic')
        print(f"Consuming messages from topic '{topic}'...")
        print("Press Ctrl+C to stop\n")
        
        for topic_name, partition, offset, timestamp_ms, message in consume_messages(consumer, topic, max_messages=10):
            ts_str = ""
            if timestamp_ms is not None:
                dt = datetime.fromtimestamp(timestamp_ms / 1000.0)
                ts_str = f" (produced: {dt.strftime('%b %d, %Y at %I:%M:%S %p')})"
            print(f"Received message from {topic_name}[{partition}] at offset {offset}{ts_str}:")
            print(f"  {json.dumps(message, indent=2)}")
            print()
        
        print("Finished consuming messages")
        
    except KafkaException as e:
        print(f"Kafka error: {e}", file=sys.stderr)
        sys.exit(1)
    except Exception as e:
        print(f"Unexpected error: {e}", file=sys.stderr)
        sys.exit(1)


if __name__ == '__main__':
    main()
```

### Config and dependencies (from the example)

**From the example** (`kafka-openspec-example/requirements.txt`):

```
# Python Kafka client library
# Requires Python 3.8 or higher
confluent-kafka>=2.3.0
```

**From the example** (`kafka-openspec-example/.env.example`):

```
# Docker Compose Environment Variables
# Copy this file to .env and customize as needed

# Zookeeper port (default: 2181)
ZOOKEEPER_PORT=2181

# Kafka broker port (default: 9092)
KAFKA_PORT=9092
```

The Python code also respects `KAFKA_BOOTSTRAP_SERVERS`, `KAFKA_TOPIC`, and `KAFKA_CONSUMER_GROUP` as documented in the example’s README; those are not in `.env.example` because they have defaults in the scripts.

## Prerequisites and Running the Example

You do **not** need a Confluent Cloud account. You need:

- Docker and Docker Compose
- Python 3.8 or higher (and optionally a virtual environment)

Steps (aligned with the example’s README):

1. **Start Kafka:** From the `kafka-openspec-example` directory run `docker-compose up -d`. The broker will be at `localhost:9092`.
2. **Python setup:** Create a venv if you like, then `pip install -r requirements.txt`.
3. **Produce:** Run `python producer.py`. It sends a JSON message to the default topic (`test-topic` unless `KAFKA_TOPIC` is set).
4. **Consume:** In another terminal, run `python consumer.py`. It subscribes to the same topic, consumes up to 10 messages (per the example’s main), and prints them with partition, offset, and timestamp.

Optional: copy `.env.example` to `.env` and set `ZOOKEEPER_PORT` or `KAFKA_PORT` if you need different ports.

## Wrapping Up

This post focused on **OpenSpec** and how it drives a small Kafka example. The specs gave a single source of truth for what “local Kafka” and “Python clients” mean; the change (proposal, design, tasks) made the path from idea to code explicit and traceable; and the archived change keeps an audit trail. The implementation still shows Kafka and the Confluent Python client in action—producing and consuming JSON on a local broker—but the narrative emphasized clarity, traceability, and alignment that OpenSpec provides.

Key takeaways:

- OpenSpec keeps specs as the source of truth and uses a propose → design → implement → archive workflow.
- The two-folder layout (specs/ and changes/) keeps current state and proposed deltas clear.
- The kafka-openspec-example project was built and documented using OpenSpec; all code labeled “From the example” is verbatim from that project.
- You can run the example locally with Docker Compose and Python; no cloud account is required.

## Next Steps

- Explore [OpenSpec and spec-driven workflows](https://intent-driven.dev/knowledge/openspec/) for your own projects.
- Use the [Confluent Python client documentation](https://docs.confluent.io/kafka-clients/python/current/overview.html) to go deeper on Kafka producers and consumers.
- Extend the example (e.g. more topics, different serialization) and document new behavior in the OpenSpec specs and a new change.

## Source Code and OpenSpec

The example project, including all code and OpenSpec artifacts (project context, specs, and archived changes), is in this repository at **[kafka-openspec-example](kafka-openspec-example/)**.
