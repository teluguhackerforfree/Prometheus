# Prometheus install monitor nginx server

🚀 FINAL PROJECT: EC2 Monitoring (Complete Flow)



🧠 What is Prometheus?

Prometheus is an open-source monitoring tool used to collect metrics from servers, applications, and services.

🔑 Key Features:
Time-series database (stores metrics)
Pull-based monitoring (scrapes data)
Powerful query language (PromQL)
Alerts using Alertmanager
📈 What is Grafana?

Grafana is used to visualize data from Prometheus and other sources.

🔑 Key Features:
Beautiful dashboards 📊
Multiple data sources
Real-time monitoring
Alerting support
⚙️ How They Work Together
🔄 Flow:
Prometheus collects metrics from targets (servers/apps)
Stores metrics in its database
Grafana connects to Prometheus
Displays data in dashboards

👉 Simple understanding:

Prometheus = Data Collector
Grafana = Visualization Tool
🏗️ Architecture Overview
Targets → Exporters → Prometheus → Grafana → Dashboard
Example:
Node Exporter → CPU, RAM, Disk metrics
Prometheus → Collects data
Grafana → Shows graphs

👉 Stack:

NGINX
Prometheus
Grafana
🧱 STEP 0: AWS EC2 Setup
Launch EC2
OS → Ubuntu 22.04
Type → t2.micro
Key → .pem
Open Ports (VERY IMPORTANT)
Port	Use
22	SSH
80	Nginx
3000	Grafana
9090	Prometheus
9100	Node Exporter
9113	Nginx Exporter
🔑 STEP 1: Connect Server
chmod 400 mykey.pem
ssh ubuntu@<EC2-IP>
🌐 STEP 2: Install Nginx
sudo apt update
sudo apt install nginx -y
sudo systemctl enable --now nginx

Test:

http://<EC2-IP>
⚙️ STEP 3: Install Node Exporter (FIXED VERSION)
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
tar xvf node_exporter-1.10.2.linux-amd64.tar.gz
sudo mv node_exporter-1.10.2.linux-amd64/node_exporter /usr/local/bin/
🔥 Make it permanent (IMPORTANT)
sudo nano /etc/systemd/system/node_exporter.service

Paste:

[Unit]
Description=Node Exporter

[Service]
User=nobody
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target

Start:

sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter

Test:

curl http://localhost:9100/metrics
⚙️ STEP 4: Enable Nginx Metrics (PROPER WAY)
sudo nano /etc/nginx/conf.d/stub_status.conf

Paste:

server {
    listen 127.0.0.1:8080;

    location /nginx_status {
        stub_status;
        allow 127.0.0.1;
        deny all;
    }
}

Restart:

sudo nginx -t
sudo systemctl reload nginx

Test:

curl http://127.0.0.1:8080/nginx_status
⚙️ STEP 5: Install Nginx Exporter (VERY IMPORTANT 🔥)
cd /tmp
wget https://github.com/nginxinc/nginx-prometheus-exporter/releases/download/v1.3.0/nginx-prometheus-exporter_1.3.0_linux_amd64.tar.gz
tar xvf nginx-prometheus-exporter_1.3.0_linux_amd64.tar.gz
sudo mv nginx-prometheus-exporter /usr/local/bin/
Create service:
sudo nano /etc/systemd/system/nginx_exporter.service

Paste:

[Unit]
Description=Nginx Exporter

[Service]
User=nobody
ExecStart=/usr/local/bin/nginx-prometheus-exporter \
  --nginx.scrape-uri=http://127.0.0.1:8080/nginx_status
Restart=always

[Install]
WantedBy=multi-user.target

Start:

sudo systemctl daemon-reload
sudo systemctl enable --now nginx_exporter

Test:

curl http://localhost:9113/metrics
⚙️ STEP 6: Install Prometheus (PRODUCTION FIXED)
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.53.0/prometheus-2.53.0.linux-amd64.tar.gz
tar xvf prometheus-2.53.0.linux-amd64.tar.gz
Move binaries
sudo mv prometheus-2.53.0.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-2.53.0.linux-amd64/promtool /usr/local/bin/
Create folders (IMPORTANT FIX 🔥)
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
sudo chown -R nobody:nogroup /var/lib/prometheus
Create config
sudo nano /etc/prometheus/prometheus.yml

Paste:

global:
  scrape_interval: 15s

scrape_configs:

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'nginx'
    static_configs:
      - targets: ['localhost:9113']




Create service (THIS FIXES YOUR ERROR 🔥)
sudo nano /etc/systemd/system/prometheus.service

Paste:

[Unit]
Description=Prometheus

[Service]
User=nobody
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus
Restart=always

[Install]
WantedBy=multi-user.target


Start Prometheus
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus

Test:

curl http://localhost:9090/-/healthy

👉 Open:

http://<EC2-IP>:9090
⚙️ STEP 7: Install Grafana
sudo apt install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt update
sudo apt install grafana -y

Start:

sudo systemctl enable --now grafana-server
🌐 STEP 8: Open Grafana
http://<EC2-IP>:3000

Login:

admin / admin
🔗 STEP 9: Connect Prometheus

👉 Grafana → Data Sources → Add Prometheus

URL:

http://localhost:9090

Click:
✅ Save & Test

📊 STEP 10: Import Dashboards

👉 Dashboards → Import

Name	ID
Node Exporter	1860
Nginx	12708
🎉 FINAL RESULT

You now have:

✅ CPU Monitoring
✅ RAM Monitoring
✅ Disk Monitoring
✅ Nginx Requests
✅ Live Dashboards

🧾 RESUME PROJECT (COPY THIS)

Project: AWS EC2 Monitoring using Prometheus & Grafana

Implemented full monitoring stack for Nginx server
Configured Node Exporter & Nginx Exporter for metrics
Visualized system and application metrics using Grafana dashboards
Used Prometheus for real-time data collection
