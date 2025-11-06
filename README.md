# Website-Availability-Monitoring-using-Prometheus-Blackbox-Exporter-Grafana

Monitor uptime and response times of any public website (like google.com or yourcompany.com).


Tools: Prometheus + Blackbox Exporter + Grafana (using Docker)
Goal: Monitor uptime, response time, and HTTP status codes for websites like google.com, github.com, etc.


Step 1: Create a new project folder
mkdir prom-blackbox
cd prom-blackbox


Step 2: Create a dedicated Prometheus config file

Add this job:

  global:
  scrape_interval: 10s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "blackbox"
    metrics_path: /probe
    params:
      module: [http_2xx]   # HTTP 200 OK check
    static_configs:
      - targets:
        - https://google.com
        - https://github.com
        - https://example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: host.docker.internal:9115  # Blackbox exporter endpoint



Step 3: Run the Blackbox Exporter container
docker run -d \
  --name blackbox \
  -p 9115:9115 \
  prom/blackbox-exporter



Step 4: Run Prometheus with the new config file
docker run -d \
  --name prometheus-blackbox \
  -p 9091:9090 \
  -v $(pwd)/prometheus-blackbox.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus


Note: We used port 9091 instead of 9090 so it won’t clash with other Prometheus projects you might be running.


Step 5: Access Prometheus web UI

Go to:

http://localhost:9091


Then navigate to:

Status → Targets


You should see:

blackbox job with targets: google.com, github.com, example.com
and their status: UP


Step 6: Run Grafana (if not already running)
docker run -d \
  --name grafana-blackbox \
  -p 3001:3000 \
  grafana/grafana


  Visit:

http://localhost:3001
Default credentials: admin / admin


Step 7: Connect Prometheus to Grafana

In Grafana → Connections → Data Sources → Add Data Source

Choose Prometheus

URL: http://host.docker.internal:9091

Save & Test ✅



Step 8: Import Blackbox dashboard

Go to Dashboards → Import

Enter ID: 9965 (Blackbox Exporter Overview)

Choose your Prometheus data source and import

You’ll see:

Uptime status (green = UP)

Response time

DNS & SSL metrics

HTTP code summary





