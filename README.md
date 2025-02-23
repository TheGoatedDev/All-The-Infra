# All-The-Infra

A comprehensive development infrastructure setup using Docker Compose, providing databases, message queues, mail testing, and monitoring solutions.

## Services Overview

This infrastructure setup includes multiple services organized into different categories:
- Databases (PostgreSQL, MySQL, Redis, MongoDB, InfluxDB)
- Message Queues (RedPanda, RabbitMQ)
- Mail Testing (MailHog)
- Monitoring (Grafana, Mimir, Loki, Tempo, OpenTelemetry)

## Port Mappings

### Database Services
| Service | Port | Purpose |
|---------|------|---------|
| PostgreSQL | 5432 | PostgreSQL database connection |
| MySQL | 3306 | MySQL database connection |
| Redis | 6379 | Redis cache connection |
| MongoDB | 27017 | MongoDB database connection |
| InfluxDB | 8086 | InfluxDB time series database connection |

### Message Queue Services
| Service | Port | Purpose |
|---------|------|---------|
| RedPanda | 9092 | Kafka-compatible messaging |
| RabbitMQ | 5672 | AMQP messaging |
| RabbitMQ Management | 15672 | RabbitMQ web management interface |


### Mail Services
| Service | Port | Purpose |
|---------|------|---------|
| MailHog | 1025 | SMTP server |
| MailHog Web | 8025 | Web interface for mail testing |

### Monitoring Services
| Service | Port | Purpose |
|---------|------|---------|
| Mimir | 9009 | Metrics storage and querying |
| Loki | 3100 | Log aggregation |
| Tempo | 3200 | Distributed tracing backend |
| Tempo OTLP | 4317 | OpenTelemetry gRPC receiver |
| Tempo OTLP HTTP | 4318 | OpenTelemetry HTTP receiver |
| Grafana | 3000 | Monitoring dashboard UI |
| OpenTelemetry Collector | 4317 | OTLP gRPC receiver (traces, metrics, logs) |
| OpenTelemetry Collector | 4318 | OTLP HTTP receiver (traces, metrics, logs) |
| OpenTelemetry Collector | 8888 | Prometheus metrics endpoint |
| OpenTelemetry Collector | 8889 | Debug endpoint |
| OpenTelemetry Collector | 13133 | Health check endpoint |

## Network Configuration

The infrastructure is organized into separate networks for isolation:
- `database-network`: For database services
- `queue-network`: For message queue services
- `mail-network`: For mail testing services
- `monitoring-network`: For monitoring services

## Getting Started

1. Ensure you have Docker and Docker Compose installed
2. Clone this repository
3. Start all services:
   ```bash
   docker compose up -d
   ```
4. Start specific service category:
   ```bash
   # For databases only
   docker compose -f docker-compose.yml -f docker-compose.databases.yml up -d
   
   # For message queues only
   docker compose -f docker-compose.yml -f docker-compose.queues.yml up -d
   
   # For mail services only
   docker compose -f docker-compose.yml -f docker-compose.mail.yml up -d
   
   # For monitoring only
   docker compose -f docker-compose.yml -f docker-compose.monitoring.yml up -d
   ```

## Health Checks

All services are configured with appropriate health checks to ensure they are running correctly. You can check the status of all services using:

```bash
docker compose ps
```

## Resource Limits

Each service has been configured with appropriate memory limits to ensure stable operation:
- Database services: 512MB - 1GB
- Message queues: 512MB - 1GB
- Mail services: 256MB
- Monitoring services: 256MB - 1GB 