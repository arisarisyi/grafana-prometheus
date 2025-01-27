# Grafana and Prometheus Installation Guide

This guide provides step-by-step instructions for installing and configuring Prometheus, Node Exporter, and Grafana on a Linux system.

## Prerequisites

- Linux-based operating system
- Root or sudo access
- Internet connection for downloading packages

## Installing Prometheus

1. Download and extract Prometheus:
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.53.3/prometheus-2.53.3.linux-amd64.tar.gz
tar xvf prometheus-2.53.3.linux-amd64.tar.gz
cd prometheus-2.53.3.linux-amd64
```

2. Create system user and group:
```bash
groupadd --system prometheus
useradd --system -s /sbin/nologin -g prometheus prometheus
```

3. Move binary files and verify installation:
```bash
mv prometheus promtool /usr/local/bin/
prometheus --version
```

4. Create necessary directories and set permissions:
```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus/
mv consoles/ console_libraries/ prometheus.yml /etc/prometheus/
```

5. Configure Prometheus:
```yaml
# /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

6. Create systemd service:
```ini
# /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

7. Enable and start Prometheus:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```

## Installing Node Exporter

1. Download and extract Node Exporter:
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvf node_exporter-1.8.2.linux-amd64.tar.gz
cd node_exporter-1.8.2.linux-amd64
mv node_exporter /usr/local/bin/
```

2. Create systemd service:
```ini
# /etc/systemd/system/node-exporter.service
[Unit]
Description=Prometheus exporter for machine metrics

[Service]
Restart=always
User=prometheus
ExecStart=/usr/local/bin/node_exporter
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```

3. Enable and start Node Exporter:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node-exporter.service
```

4. Update Prometheus configuration to include Node Exporter:
```yaml
# /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node-exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

5. Restart Prometheus:
```bash
sudo systemctl restart prometheus
```

## Installing Grafana

1. Follow the official Debian installation guide:
   https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/

2. Enable and start Grafana:
```bash
sudo systemctl enable --now grafana-server.service
```

3. Optional: Change Grafana port (if 3000 is in use):
```ini
# /etc/grafana/grafana.ini
# Change ;http_port = 3000 to:
http_port = 4000
```

## Verification

1. Prometheus UI: http://your-ip:9090
2. Grafana UI: http://your-ip:3000 (or custom port)
3. Check service status:
```bash
systemctl status prometheus
systemctl status node-exporter
systemctl status grafana-server
```

## Default Ports

- Prometheus: 9090
- Node Exporter: 9100
- Grafana: 3000 (or custom port)
- Default user and password admin
