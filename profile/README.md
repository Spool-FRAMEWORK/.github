# Spool Framework

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Java](https://img.shields.io/badge/Java-21%2B-orange.svg)](https://openjdk.org/)
[![Maven](https://img.shields.io/badge/build-Maven-C71A36.svg)](https://maven.apache.org/)

**Spool** is a **modular, event-driven data pipeline framework for Java**. It handles the full lifecycle from raw source extraction to a queryable data mart — with built-in observability, pluggable adapters, and no vendor lock-in.

> **[Spool-FRAMEWORK-Labs](https://github.com/Spool-FRAMEWORK-Labs)** is the examples & use cases organization for the Spool Framework — a collection of real-world, production-style pipelines built with Spool, covering data sources like public APIs, government datasets, healthcare records, and financial filings.

---

## How it works

```
External Sources  (HTTP · DB · File · Queue)
        │
        ▼
     Crawler    fetch → normalize → split → persist
        │
      Inbox     (S3 · FileSystem · custom)
        │
     Janitor    housekeeping: republish stuck · quarantine failed · remove expired
        │
    Event Bus   (Kafka · InMemory)
        │
     Ingester   validate → batch → flush
        │
    Data Lake   (S3 · FileSystem)
        │
     Mounter    aggregate → promote
        │
    Data Mart   (silver · gold)


     Watchdog   heartbeat monitoring  ←── Crawler
```

Each module communicates through an event bus. Every envelope state transition (`CAPTURED → INGESTED → PERSISTED`) is an event — fully auditable and reproducible.

---

## Modules

| Module | Description |
|---|---|
| [**crawler**](https://github.com/Spool-FRAMEWORK/crawler) | Polls external sources, normalizes the data, and stores each record in the inbox. |
| [**janitor**](https://github.com/Spool-FRAMEWORK/janitor) | Keeps the inbox healthy — republishes stuck envelopes, quarantines failures, removes expired ones. |
| [**ingester**](https://github.com/Spool-FRAMEWORK/ingester) | Consumes inbox events, validates payloads, and flushes batches to the data lake. |
| [**validator**](https://github.com/Spool-FRAMEWORK/validator) | Pluggable business-rule validators discovered at runtime via `@Validate` and ServiceLoader. |
| [**mounter**](https://github.com/Spool-FRAMEWORK/mounter) | Reads closed partitions from the data lake, applies aggregations, and writes results to a data mart. |
| [**watchdog**](https://github.com/Spool-FRAMEWORK/watchdog) | HTTP service that receives heartbeats from crawlers and exposes a `/health` endpoint. |
| [**core**](https://github.com/Spool-FRAMEWORK/core) | Shared abstractions — `SpoolNode`, `Envelope`, `EventBus`, ports and model. |
| [**infrastructure**](https://github.com/Spool-FRAMEWORK/infrastructure) | Ready-to-use adapters: HTTP source, S3, Kafka, filesystem, normalizers. Plugin SPI engine. |
| [**dsl**](https://github.com/Spool-FRAMEWORK/dsl) | Declare your entire pipeline in a single `Spool.yaml` and boot it with one line of code. |
| [**runtime**](https://github.com/Spool-FRAMEWORK/runtime) | Bootstrap class that wires DSL + OpenTelemetry configuration into a production-ready application. |

---

## Quick Start

### From YAML

Create `src/main/resources/Spool.yaml`:

```yaml
infrastructure:
  eventBus:
    type: "inMemory"
  inbox:
    filesystem:
      path: "/var/spool/inbox"
  dataLake:
    filesystem:
      path: "/var/spool/datalake"

modules:

  - crawler:
      id: "my-crawler"
      source:
        id: "my-source"
        format: JSON_ARRAY
        poll:
          http:
            url: "https://api.example.com/data"
      schedule:
        everyMilliseconds: 60000

  - janitor:
      id: "my-janitor"
      everyMilliseconds: 5000

  - ingester:
      id: "my-ingester"
      flush:
        size: 100
        every: 30
        unit: SECONDS
```

Boot the pipeline:

```java
SpoolRuntime.builder()
        .OpenTelemetryConfiguration(otelConfig)
        .withNodeFromDSL("/Spool.yaml")
        .build()
        .start();
```

### From code

```java
Crawler crawler = CrawlerBuilderFactory.poll(new HTTPPollSource("https://api.example.com/data", "my-api"))
        .source()
            .ports(CrawlerPorts.builder().inbox(inboxWriter).bus(eventBus).build())
            .schedule(PollingConfiguration.every(Duration.ofMinutes(1)))
            .and()
        .createWith(StandardNormalizer.JSON_ARRAY);

Janitor janitor = JanitorBuilderFactory.polling()
        .from(inboxQuery)
        .with(inboxUpdater)
        .removeWith(envelopeRemover)
        .on(eventPublisher)
        .subscribeWith(eventSubscriber)
        .every(Duration.ofSeconds(5))
        .create();

Ingester ingester = IngesterBuilderFactory.reactive()
        .readWith(envelopeResolver)
        .flushPolicy(FlushPolicy.whenReaches(100).orEvery(Duration.ofSeconds(30)))
        .writingTo(dataLakeWriter)
        .subscribingWith(eventSubscriber)
        .create();

SpoolNode.create()
        .register(crawler)
        .register(janitor)
        .register(ingester)
        .start();
```

---

## Why Spool

- **One interface to extend anything** — implement `PollSource` to crawl any system; `@SpoolPlugin` to add any adapter
- **Event-sourced audit trail** — every state change is an immutable event on the bus
- **Observability out of the box** — OpenTelemetry traces, Prometheus metrics, Loki logs, and Grafana dashboards included
- **Technology-agnostic** — swap Kafka for any bus, S3 for any store, without touching pipeline logic
- **Declare or code** — full YAML DSL for operations, fluent builder API for fine-grained control

---

## Background

Spool was developed as a final-year thesis project (*Trabajo de Fin de Título*) at the Universidad de Las Palmas de Gran Canaria, within the Bachelor's degree in Computer Science Engineering.

---

> Built with Java 21 · Apache License 2.0
