<p align="center">
  <img src="assets/logo.svg" alt="Project Logo" width="175">
  <br>
  <i>The Distributed Logging: Grafana LGTM & SigNoz</i>
</p>

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
# Distributed Logging
## Grafana LGTM Stack & SigNoz Deployment Examples

This repository provides **production-oriented deployment examples** for modern **distributed logging, metrics, and tracing** stacks using **Grafana LGTM (Loki, Grafana, Tempo, Prometheus)** and **SigNoz**.

The primary goal of this project is to demonstrate **how to stand up a complete observability stack quickly and correctly**, while keeping the configuration **portable, reproducible, and security-aware**.

The repository is intentionally structured to support **multiple deployment models** (Docker Compose, Docker Swarm) and to act as a **reference implementation** for teams building their observability and SIEM journey.

---

## Why This Repository Exists

Modern systems are:

- Distributed
- Containerized
- Dynamically scaled
- Security-sensitive

Traditional single-node logging and monitoring approaches are no longer sufficient.

This repository focuses on:

- **Distributed logging** (Loki, Promtail)
- **Metrics & SLIs/SLOs** (Prometheus)
- **Distributed tracing** (Tempo, OpenTelemetry)
- **Security & audit visibility**
- **SIEM-ready observability foundations**

It is designed to be:

- Easy to run locally
- Safe to extend
- Suitable for demos, PoCs, and real environments

---

## Repository Structure

```text
distributed-logging/
├── docker-compose/
│   ├── grafana-lgtm/
│   │   ├── docker-compose.yml
│   │   ├── config/
│   │   │   ├── grafana-datasources.yaml
│   │   │   ├── grafana-dashboards.yaml
│   │   │   ├── prometheus.yaml
│   │   │   ├── promtail.yaml
│   │   │   ├── loki.yaml
│   │   │   ├── tempo.yaml
│   │   │   ├── otel-collector.yaml
│   │   │   └── alertmanager.yml
│   │   └── dashboards/
│   │       ├── service-overview.json
│   │       └── service-tracker.json
│   │
│   └── signoz/
│       ├── docker-compose.yml
│       ├── otel-collector-config.yaml
│       └── common/
│           ├── clickhouse/
│           ├── dashboards/
│           └── locust-scripts/
│
├── docker-swarm/
│   └── (Swarm-oriented deployments – WIP)
│
├── LICENSE
├── README.md
└── .gitignore
```

---

## Deployment Models

This repository separates **deployment logic from configuration**.

### Docker Compose

Located under:

```text
docker-compose/
```

Used for:

- Local development
- Proof of concepts (PoC)
- Demo environments
- CI-based validation

Available stacks:

- **Grafana LGTM Stack**
- **SigNoz**

Each stack is self-contained and can be started independently.

---

### Docker Swarm (Planned / WIP)

Located under:

```text
docker-swarm/
```

This directory will contain:

- Swarm-native service definitions
- Overlay networking examples
- Placement & scaling strategies
- Production-oriented defaults

---

## Grafana LGTM Stack

### Quick Start

#### Prerequisites

- Docker ≥ 24
- Docker Compose v2

| Use Case | Recommended RAM |
|--------|---------------|
| Local development / demo | 4 GB          |
| Multi-service testing | 6 GB          |
| Production-like / SIEM scenarios | 8 GB+         |

#### Start the Stack

```bash
cd docker-compose/grafana-lgtm
docker compose up -d
```

#### Access Points

| Service | URL |
|------|------|
| Grafana | http://localhost:3000 |
| Prometheus | http://localhost:9090 |
| Loki | http://localhost:3100 |
| Tempo (OTLP) | http://localhost:4318 |

Grafana is configured with:

- Anonymous access enabled
- Pre-provisioned datasources
- Auto-imported dashboards

No manual UI setup is required.

---

## Dashboards

The `dashboards/` directory contains **pre-built, production-grade dashboards**, including:

- **Service Overview**
    - Service/Instance List
    - Uptime

- **Service Tracker**
    - Per-service & per-instance metrics
    - CPU, memory, disk
    - Error rates
    - HTTP statistics
    - Logs (Loki)
    - Slow traces (Tempo)

All dashboards:

- Use **label-based filtering**
- Support **single or multiple instances**
- Are **environment-agnostic**

---

## Logging & Tracing Architecture (Grafana LGTM)

- **Application**
    - Emits structured JSON logs
    - Exposes Prometheus metrics
    - Exports traces via OpenTelemetry

- **Promtail**
    - Discovers containers dynamically
    - Enriches logs with `service`, `instance`, `service_instance_id`
    - Promtail is configured to collect logs **only from containers explicitly labeled for logging**.
      - Only services with the following Docker label will have their logs shipped to Loki:
    ```yaml
    labels:
      logging: "promtail"
    ```  

- **Loki**
    - Stores logs
    - Enables label-based queries

- **Tempo**
    - Stores traces
    - Correlates with logs via `trace_id`

This architecture enables **full signal correlation** across logs, metrics, and traces.

---

## SigNoz Stack

SigNoz is an **open-source observability platform** that provides an integrated experience for **metrics, logs, and traces** on top of OpenTelemetry.

This repository includes a **ready-to-run SigNoz setup** for comparison and learning purposes.

### Quick Start (SigNoz)

```bash
cd docker-compose/signoz
docker compose up -d
```

### What This Setup Demonstrates

- Native OpenTelemetry-first architecture
- ClickHouse-backed storage
- Unified UI for logs, metrics, and traces
- Application performance monitoring (APM)
- Built-in service maps and dependency graphs

### When to Use SigNoz vs Grafana LGTM

| Use Case | Grafana LGTM | SigNoz |
|------|------|------|
| Highly customizable dashboards | ✅ | ⚠️ |
| Native OpenTelemetry UX | ⚠️ | ✅ |
| SIEM-style log exploration | ✅ | ⚠️ |
| Single-pane-of-glass APM | ⚠️ | ✅ |
| Long-term flexibility | ✅ | ⚠️ |

---

## Security & SIEM Perspective

Distributed logging is not only about debugging.

It is foundational for:

- Audit trails
- Incident response
- Threat detection
- Forensics
- Zero Trust observability

This repository is structured to support:

- Security event enrichment
- Cross-signal correlation (logs + traces + metrics)
- Future SIEM integrations

A detailed write-up of this approach will be published as a **Medium article** based on this repository.

---

## License

Apache 2.0 License

