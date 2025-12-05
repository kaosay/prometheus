# prometheus
easy and powerfull monitor

```
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alert_rules.yml:/etc/prometheus/alert_rules.yml
    ports:
      - "19090:9090"

    mem_limit: 500m
    cpus: 0.5

    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:9090/-/ready"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s

  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "19093:9093"

    mem_limit: 200m
    cpus: 0.3

    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:9093/-/ready"]
      interval: 30s
      timeout: 10s
      retries: 3

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - ./grafana-data:/var/lib/grafana

    mem_limit: 400m
    cpus: 0.5

    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:3000/api/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 20s

  node_exporter:
    image: prom/node-exporter:latest
    container_name: node_exporter
    pid: host
    network_mode: host
    restart: unless-stopped

    # Node Exporter 默认监听 9100，这里改为 19100
    command:
      - '--web.listen-address=:19100'
    mem_limit: 100m
    cpus: 0.2

    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:19100/metrics"]
      interval: 30s
      timeout: 10s
      retries: 3

        #volumes:
        #  grafana-data:
```
