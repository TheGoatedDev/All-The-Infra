# All-The-Infra

Lazy-loading development infrastructure powered by [Lazytainer](https://github.com/vmorganp/Lazytainer). Services start automatically when you connect to them and stop after 30 minutes of inactivity, keeping your system resources free.

## How It Works

A lightweight proxy ([Lazytainer](https://github.com/vmorganp/Lazytainer)) listens on all service ports. When your application connects to a port (e.g., Postgres on 5432), Lazytainer detects the traffic and starts the corresponding container. After 30 minutes with no connections, the container is stopped to free resources.

From your application's perspective, nothing changes — same ports, same connection strings. The only difference is a brief startup delay on first connection if the service isn't already running.

## Getting Started

1. Ensure Docker and Docker Compose are installed
2. Clone this repository
3. Start the proxy:
   ```bash
   docker compose up -d
   ```
4. Connect to any service as normal — it will start automatically

## Services

### Databases
| Service | Port | Connection |
|---------|------|------------|
| PostgreSQL | 5432 | `postgres://postgres:password@localhost:5432` |
| MariaDB | 3306 | `mysql://root:password@localhost:3306/development` |
| Dragonfly | 6379 | `redis://localhost:6379` (Redis-compatible) |
| MongoDB | 27017 | `mongodb://localhost:27017` |
| InfluxDB | 8086 | `http://localhost:8086` |
| ClickHouse | 8123 | `http://localhost:8123` (HTTP interface) |

### Message Queues
| Service | Port(s) | Connection |
|---------|---------|------------|
| Redpanda (Kafka) | 9092 | `localhost:9092` |
| RabbitMQ | 5672, 15672 (management UI) | `amqp://localhost:5672` |
| NATS | 4222, 8222 (monitoring), 6222 (cluster) | `nats://localhost:4222` |

### Mail
| Service | Port(s) | Connection |
|---------|---------|------------|
| Mailpit | 1025 (SMTP), 8025 (Web UI) | `smtp://localhost:1025` |

### Monitoring
| Service | Port(s) | Connection |
|---------|---------|------------|
| Grafana | 4316 | `http://localhost:4316` |
| OpenTelemetry | 4317 (gRPC), 4318 (HTTP) | `http://localhost:4317` |

### Storage
| Service | Port(s) | Connection |
|---------|---------|------------|
| SeaweedFS | 8333 (S3 API), 8888 (Filer UI) | `http://localhost:8333` |

## Cold Start Delay

When a service isn't running and you first connect, there is a startup delay:

| Service | Typical Startup Time |
|---------|---------------------|
| Dragonfly | ~1-2 seconds |
| PostgreSQL | ~2-5 seconds |
| MongoDB | ~3-8 seconds |
| MariaDB | ~5-15 seconds |
| ClickHouse | ~3-10 seconds |
| Redpanda | ~5-15 seconds |
| RabbitMQ | ~5-15 seconds |

Your application's first connection attempt will likely fail. **Ensure your application has connection retry logic** (most ORMs, connection pools, and frameworks handle this automatically).

## Checking Status

```bash
# See which services are running
docker compose ps

# Check Lazytainer proxy logs
docker compose logs lazytainer

# Manually start a service (bypass lazy loading)
docker compose start data-postgres

# Manually stop a service
docker compose stop data-postgres
```

## Configuration

### Changing the Idle Timeout

The default idle timeout is 30 minutes (1800 seconds). To change it, edit the `inactiveTimeout` values in `docker-compose.lazytainer.yml`:

```yaml
- "lazytainer.group.postgres.inactiveTimeout=3600"  # 1 hour
```

### Adding a New Service

1. Add the service definition to the appropriate compose file
2. Remove any `ports`, `restart`, and `networks` configuration
3. Add `network_mode: service:lazytainer` and `depends_on: [lazytainer]`
4. Add label `lazytainer.group=<name>`
5. In `docker-compose.lazytainer.yml`, add port mappings and group labels to the `lazytainer` service

### NATS Box (Tools)

The NATS utility container runs on-demand only:

```bash
docker compose --profile tools run --rm nats-box
```

## Resource Usage

When idle (no active services), only the two Lazytainer proxy containers run, using minimal resources (~20MB each). Services are fully stopped and consume zero CPU/memory until activated.

## Architecture

```
Your App → localhost:5432 → Lazytainer (proxy) → detects traffic → starts Postgres container
                                                                  → 30 min idle → stops Postgres
```

One Lazytainer instance manages all services. SeaweedFS uses different ports than ClickHouse so no conflicts.
