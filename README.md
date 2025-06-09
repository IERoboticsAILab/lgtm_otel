# Grafana + OpenTelemetry LGTM Stack

This repo sets up a **Grafana LGTM Stack** (Loki, Grafana, Tempo, Mimir) with an **OpenTelemetry Collector** using Docker Compose. It collects **host metrics** like CPU, memory, disk, etc., and visualizes them in Grafana.

---

## 🧰 Components

### 1. **Grafana LGTM Stack (`grafana/otel-lgtm`)**
- All-in-one image containing:
  - **Grafana** (UI and dashboards)
  - **Loki** (logs)
  - **Tempo** (traces)
  - **Mimir** (metrics store)
- Listens on:
  - `:3000` → Grafana UI
  - `:4317` → OTLP gRPC input (used by the collector)
  - `:4318` → OTLP HTTP (optional)

### 2. **OpenTelemetry Collector**
- Collects system/host metrics using the `hostmetrics` receiver.
- Processes and sends them to Grafana LGTM via OTLP gRPC.
- Uses a custom config file: `otel-collector-config.yaml`.

---

## 🚀 How to Run

### Prerequisites:
- Docker & Docker Compose
- (Optional) Portainer for UI management

### Start via CLI:
```bash
docker-compose up -d
````

---

## 🛠 Configuration Explained

### `docker-compose.yml`

```yaml
services:
  otel-lgtm:
    image: grafana/otel-lgtm
    ports:
      - "3000:3000"      # Grafana UI
      - "4317:4317"      # OTLP gRPC (metrics/traces/logs)
      - "4318:4318"      # OTLP HTTP (optional)
    volumes:
      - grafana_data:/var/lib/grafana

  otel-collector:
    image: otel/opentelemetry-collector
    command: ["--config=/etc/otel-collector-config.yaml"]
    volumes:
      - ./otel-collector-config.yaml:/etc/otel-collector-config.yaml
    depends_on:
      - otel-lgtm

volumes:
  grafana_data:
```

This mounts a **named volume** for Grafana persistence, and uses a host file for the collector config.

---

### `otel-collector-config.yaml`

```yaml
receivers:
  hostmetrics:
    collection_interval: 10s
    scrapers:
      cpu:
      memory:
      disk:
      filesystem:
      network:
      load:

processors:
  batch:

exporters:
  otlp:
    endpoint: otel-lgtm:4317
    tls:
      insecure: true

service:
  pipelines:
    metrics:
      receivers: [hostmetrics]
      processors: [batch]
      exporters: [otlp]
```

This config:

* Scrapes host metrics every 10s.
* Uses the `otlp` exporter to send them to the `otel-lgtm` container.

---

## 📊 Access Grafana

* URL: [http://localhost:3000](http://localhost:3000)
* Default login: `admin` / `admin`

You can create dashboards using metrics like:

* `system_cpu_time`
* `system_memory_usage`
* `system_filesystem_usage`
* etc.

---

## ✅ Persistence

All Grafana dashboards and config are stored in the `grafana_data` volume, so they survive container restarts.

---
