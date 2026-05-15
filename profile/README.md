# SPOOL — Event-Driven Data Platform Framework

> Design and implementation of an agnostic, auditable, and extensible data platform based on events.

SPOOL is a distributed data platform framework built around **event sourcing** and **CQRS** principles. It provides a modular, plugin-based architecture to ingest, validate, transform, and store data from heterogeneous sources — in a technology-agnostic and fully auditable way.

---

## Core Modules

| Module | Description |
|---|---|
| **Crawler** | Discovers, schedules and normalizes data source extraction |
| **Janitor** | Keeps Inbox clean from stuck data and removes persisted payload |
| **Ingester** | Validates and persist incoming data envelopes |
| **Validator** | Applies data quality rules and schema checks |
| **Mounter** | Tranform or aggregates partitions promoting them to higher layers |

---

## Key Features

- **Plugin-based extensibility** — extend any module via the SPI pattern without modifying core code
- **DSL configuration** — define pipelines declaratively through YAML descriptors
- **Event sourcing & audit trail** — every state change is recorded as an immutable event
- **Technology-agnostic** — works with REST APIs, message brokers, file systems, and more. You think it, you adapt it
- **Observability built-in** — distributed tracing, metrics, and structured logging out of the box with OpenTelemetry protocol
- **Multiple deployment models** — standalone JAR, Docker Compose, Kubernetes, or AWS (ECS/Fargate). It is up to you!

---

## Architecture Overview

Sources -> [Crawler] -> [Ingester] -> [Validator] -> Data Lake -> [Mounter] -> Data Mart (Gold Layer) / Silver Layer
                          |               |
                      Event Bus (Kafka / ActiveMQ / ...)

All inter-module communication flows through an event bus, ensuring loose coupling and full reproducibility.

---

## Getting Started

Documentation, examples, and deployment guides are available in each repository.

- `core` — Framework core and shared abstractions
- `infrastructure` — Official plugin implementations and SPI discovery engine
- `dsl` — YAML DSL engine

---

## Background

SPOOL was developed as a final-year thesis project (Trabajo de Fin de Título) at the Universidad de Las Palmas de Gran Canaria, within the Bachelor's degree in Computer Science Engineering.

---

> Built with Java 21
