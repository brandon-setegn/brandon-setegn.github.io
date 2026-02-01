---
title:  "Spec-Driven Development with OpenSpec: A Kafka Example"
date:   2025-01-31 20:00:00 -0500
categories: [openspec, kafka, streaming, python]
mermaid: true
tags: [openspec, spec-driven development, kafka, python, streaming, ai]
image:
  path: /assets/img/openspec-header.png
---

When I originally started working on this post I thought it was going to be about Kafka and how to get started with it.  But over the last few weeks I started to do to more _spec-driven development_, mainly with OpenSpec.  Having seen the how incredibly powerful spec-driven can be, I had to pivot to this new topic.

[OpenSpec](https://intent-driven.dev/knowledge/openspec/) is a lightweight  approach to spec-driven development that treats a single, unified specification as the source of truth for a system. By defining **intent before implementation**, it improves clarity, strengthens traceability from requirements to code, and helps keep humans and AI assistants aligned throughout the development process.

In this post we'll use OpenSpec to drive a small project: a local [Apache Kafka](https://kafka.apache.org/) producer and consumer in Python.  The code will demonstrate how to stand up a local Kafka server and connect with a [Confluent Python client](https://docs.confluent.io/kafka-clients/python/current/overview.html).  The focus here is on how OpenSpec shapes the work and the benefits you get from it.

> 💡**OpenSpec keeps a living spec as the source of truth.  Instead of scattering design across tickets and wikis, you maintain specs and changes that evolve with the codebase—improving clarity, traceability, and alignment with AI assistants.**

## Source Code

> The example project is available on GitHub: [kafka-test](https://github.com/brandon-setegn/kafka-test).  All code blocks labeled "From the example" below are sourced from that repo.

## What is OpenSpec?

[OpenSpec](https://github.com/Fission-AI/OpenSpec) treats **specifications as the source of truth**.  Instead of scattering design across tickets, wikis, and ad-hoc docs, you maintain a living spec that evolves with the codebase.  The workflow is **propose → design → implement → archive**: you capture *what* should change and *why*, then implement, then archive the change so you have an audit trail.

Two folders structure the work:

- **specs/** — The current state of the system.  Each spec describes a capability (e.g. local Kafka, Python clients) with requirements and scenarios in a consistent format.
- **changes/** — Proposed deltas.  A change typically has a proposal (why and what), a design (decisions and trade-offs), and a task list.  When done, the change is archived so the history is preserved.

Benefits:

- **Clarity before coding** — You spell out requirements and scenarios up front, so implementation has a clear target.
- **Traceability** — You can trace from a scenario in a spec to the code that fulfills it, and from a task list to the files that were added or changed.
- **Better alignment** — Humans and AI assistants share the same reference (the specs and change docs), which reduces drift between intent and code.
- **Lighter than heavy design docs** — OpenSpec aims for enough structure to guide work without drowning in process.  It works with Cursor, Claude Code, and other AI coding tools.

No code lives in the specs themselves.  They describe *what* the system shall do.  The code lives in the repo and is tied to the spec via the change workflow.

## Project Overview: kafka-test

The example project (kafka-test) exists to support **local Kafka experimentation and learning**, and it was built using OpenSpec.  Clone it from [kafka-test](https://github.com/brandon-setegn/kafka-test).

High-level structure (from the example):

- **Root:** `producer.py`, `consumer.py`, `docker-compose.yml`, `requirements.txt`, `.env.example`, `README.md`
- **openspec/:** `project.md`, `specs/` (current capabilities), `changes/archive/` (completed changes)

The project runs a Kafka broker locally via Docker Compose and provides Python producer and consumer clients that send and receive JSON messages.  There is no Confluent Cloud or Schema Registry in this example—everything runs against a local broker.

## OpenSpec in Action

The following sections use only artifacts from the kafka-test repo.  Each is labeled so you know it comes from that repo.

### Project context (kafka-test: openspec/project.md)

The project's OpenSpec **project context** defines purpose, conventions, and domain so that every change stays aligned.

**From the example** (kafka-test: `openspec/project.md`):

Purpose: a testing and experimentation project for Apache Kafka—producer/consumer functionality, configurations, patterns, and reusable examples. Domain terms include **Kafka Topics** (named channels for messages), **Producers** (publish to topics), **Consumers** (read from topics), **Consumer Groups**, **Partitions**, and **Brokers**. Conventions cover code style (e.g. PEP 8 for Python), architecture (separate producer and consumer modules, config for connection settings, error handling), testing, and Git workflow. Constraints include: broker must be accessible, serialization format and connection failures must be considered.

### Specs as requirements

Specs are written as requirements with **WHEN/THEN/AND** scenarios.  Two specs in the example describe the current system.

**From the example** (kafka-test: `openspec/specs/local-kafka/spec.md`):

The **Local Kafka** spec says the project SHALL provide a Docker Compose configuration to run a Kafka broker locally.  Example scenarios:

- **Start local Kafka:** WHEN a developer runs `docker-compose up` THEN Zookeeper and Kafka start, Kafka is accessible on port 9092, and services are healthy.
- **Stop:** WHEN a developer runs `docker-compose down` THEN services stop and containers are removed.
- **Connect:** WHEN a client connects to `localhost:9092` THEN the connection succeeds and the client can produce and consume messages.
- **Persist:** WHEN services are stopped and restarted THEN topics and messages are preserved via Docker volumes.

**From the example** (kafka-test: `openspec/specs/python-kafka-clients/spec.md`):

The **Python Kafka Clients** spec says the project SHALL provide a Python producer and consumer for the local broker, with dependency management and JSON serialization.  It includes scenarios such as: producer connects to localhost:9092 and sends messages to a topic with delivery confirmation, producer serializes Python dicts to JSON, consumer connects and subscribes and reads messages and deserializes JSON and uses a consumer group, and both raise appropriate errors when the broker is unreachable.  Another requirement: `pip install -r requirements.txt` installs the needed packages (e.g. confluent-kafka), and Python 3.8+ is supported.

### A change from idea to code

The Python clients were added via a single change.  Here we walk through that change using only the example’s archived artifacts.

**Proposal** (kafka-test: `openspec/changes/archive/2026-01-30-add-python-kafka-clients/proposal.md`):

- **Why:** Enable programmatic interaction with the local Kafka broker for testing, experimentation, and development. Python is popular and has good library support.
- **What changes:** Python project structure and `requirements.txt`, a Kafka producer client and a consumer client, example usage and docs, and configuration to use localhost:9092.
- **Impact:** New spec `python-kafka-clients`, new files `producer.py`, `consumer.py`, `requirements.txt`, and dependencies Python 3.x and a Kafka client library (e.g. confluent-kafka).  No breaking changes.

**Design** (kafka-test: `openspec/changes/archive/2026-01-30-add-python-kafka-clients/design.md`):

- **Library:** Use `confluent-kafka` (performance, documentation, maintenance).  Alternatives considered were kafka-python and aiokafka.
- **Serialization:** JSON for simplicity and readability.
- **Layout:** Separate `producer.py` and `consumer.py` for clear separation of concerns.
- **Configuration:** Environment variables or hardcoded defaults (localhost:9092) to match Docker Compose.
- **Non-goals:** No Schema Registry, no production-grade monitoring, no custom serialization in this change.

**Tasks** (kafka-test: `openspec/changes/archive/2026-01-30-add-python-kafka-clients/tasks.md`):

The task list maps directly to what was implemented: project setup (requirements.txt, Python version, README venv instructions), producer implementation (producer.py with create_producer, produce_message, JSON serialization, localhost:9092, error handling, `if __name__ == '__main__'`), consumer implementation (consumer.py with create_consumer, consume_messages, JSON deserialization, consumer group, error handling, main block), documentation (README setup, usage, prerequisites), and validation (connect, produce, consume, end-to-end, error when broker down).  Each of these tasks corresponds to the code and docs you see in kafka-test.

## Code from the Example

> The following snippets are from the [kafka-test](https://github.com/brandon-setegn/kafka-test) repo.  Full files and configuration are there.
{: .prompt-info }

The snippets below highlight the concepts: how the producer is created and sends serialized messages, and how the consumer reads and deserializes them.

### Producer: creation and sending (kafka-test: producer.py)

The producer is created with the Confluent client and a config dict.  You then serialize messages and call `produce`, then `poll` and `flush` to wait for delivery.

**From the example** (kafka-test: `producer.py`) — creating the producer:

```python
    config = {
        'bootstrap.servers': bootstrap_servers,
    }
    ...
    producer = Producer(config)
    return producer
```

**From the example** (kafka-test: `producer.py`) — serializing and sending a message:

```python
        if isinstance(value, dict) or not isinstance(value, (str, bytes)):
            value_json = json.dumps(value)
            value_bytes = value_json.encode('utf-8')
        ...
        producer.produce(topic, value=value_bytes, key=key_bytes, callback=delivery_callback)
        producer.poll(0)
        producer.flush()
```

The client calls your `delivery_callback(err, msg)` when each message is delivered or fails, so you can confirm or handle errors.

### Consumer: reading and deserializing (kafka-test: consumer.py)

The consumer subscribes to topics, polls in a loop, and deserializes each message from bytes to a Python dict.

**From the example** (kafka-test: `consumer.py`) — subscribe, poll, and deserialize:

```python
            msg = consumer.poll(timeout=timeout)
            ...
            value_str = msg_value.decode('utf-8')
            value_dict = json.loads(value_str)
            ...
            yield (msg.topic(), msg.partition(), msg.offset(), timestamp_ms, value_dict)
```

So the flow is: subscribe once, then repeatedly poll.  For each message, decode the value from UTF-8 and parse JSON to get a dict your code can use.

## Prerequisites and Running the Example

> You do **not** need a Confluent Cloud account.  Everything in this example runs against a local broker.
{: .prompt-info }

You need:

- Docker and Docker Compose
- Python 3.8 or higher (and optionally a virtual environment)

Steps (aligned with the example’s README):

1. **Start Kafka:** From the project directory after cloning kafka-test, run `docker-compose up -d`.  The broker will be at `localhost:9092`.
2. **Python setup:** Create a venv if you like, then `pip install -r requirements.txt`.
3. **Produce:** Run `python producer.py`.  It sends a JSON message to the default topic (`test-topic` unless `KAFKA_TOPIC` is set).
4. **Consume:** In another terminal, run `python consumer.py`.  It subscribes to the same topic, consumes up to 10 messages (per the example’s main), and prints them with partition, offset, and timestamp.

Optional: copy `.env.example` to `.env` and set `ZOOKEEPER_PORT` or `KAFKA_PORT` if you need different ports.

## Wrapping Up

This post focused on **OpenSpec** and how it drives a small Kafka example.  The specs gave a single source of truth for what “local Kafka” and “Python clients” mean.  The change (proposal, design, tasks) made the path from idea to code explicit and traceable, and the archived change keeps an audit trail.  The implementation still shows Kafka and the Confluent Python client in action—producing and consuming JSON on a local broker—but the narrative emphasized clarity, traceability, and alignment that OpenSpec provides.
 -
Key takeaways:

- OpenSpec keeps specs as the source of truth and uses a propose → design → implement → archive workflow.
- The two-folder layout (specs/ and changes/) keeps current state and proposed deltas clear.
- The kafka-test repo was built and documented using OpenSpec.  All code labeled “From the example” is sourced from that repo.
- You can run the example locally with Docker Compose and Python.  No cloud account is required.

## Next Steps

- Explore [OpenSpec and spec-driven workflows](https://intent-driven.dev/knowledge/openspec/) for your own projects.
- Use the [Confluent Python client documentation](https://docs.confluent.io/kafka-clients/python/current/overview.html) to go deeper on Kafka producers and consumers.
- Extend the example (e.g. more topics, different serialization) and document new behavior in the OpenSpec specs and a new change.

## Source Code and OpenSpec

The example project, including all code and OpenSpec artifacts (project context, specs, and archived changes), is available on GitHub: [kafka-test](https://github.com/brandon-setegn/kafka-test).
