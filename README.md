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

```
version: '3.7'

services:
  http_json_exporter:
    image: josephspurrier/http_json_exporter:latest
    mem_limit: 512m  # Limit memory usage to 512 MB
    cpus: 0.5  # Limit CPU usage to 0.5 CPUs (50% of a single CPU core)
    restart: unless-stopped
    container_name: http_json_exporter
    ports:
      - "9119:9119"  # Expose port 9119 for metrics
    volumes:
      - ./config.yml:/etc/http_json_exporter/config.yml  # Mount config file
    #environment:
    #  - TARGET_URL=http://your_target_url_to_export_json
    #  - JSON_PATH=your_json_path  # Specify the JSON path to scrape from the target URL
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9119/metrics"]
      interval: 30s
      retries: 3
      start_period: 10s
      timeout: 10s
```

```
version: '3.8'
services:
  json-exporter:
    image: prometheuscommunity/json-exporter:latest
    container_name: json-exporter   
    cpus: 0.5
    mem_limit: 256m
    restart: unless-stopped
    ports:
      - "7979:7979"
    volumes:
      - ./json-exporter-config.yml:/config.yml:ro
    command:
      - '--config.file=/config.yml'
      - '--web.listen-address=:7979'
    # ❤️ Health check (critical for production)
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--timeout=3", "http://localhost:7979/metrics", "-O", "/dev/null"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
```
## Setting json_exporter, add config to prometheus.yml
```
  - job_name: json_api
    static_configs:
      - targets:
        - http://10.0.0.1/api/test1/
        - http://10.0.0.1/api/test2/
    metrics_path: /probe
    params:
      module: [default]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 10.0.0.1:7979   # json_exporter 所在机器:端口
```

