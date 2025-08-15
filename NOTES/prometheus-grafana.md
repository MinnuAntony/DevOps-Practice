Perfect — here’s a **short, simple, native (tar) setup** for Prometheus + Grafana on your EC2.
Assuming **Ubuntu/Debian**. (On Amazon Linux/RHEL: replace `apt`/`.deb` with `yum`/`.rpm` equivalents.)

---

## 0) Prep

```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin prometheus
sudo useradd --no-create-home --shell /usr/sbin/nologin node_exporter
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

**Why:** create dedicated users + dirs (safer, cleaner).

---

## 1) Install Prometheus (tar binaries)

```bash
cd /tmp
export PROM_VER="2.53.0"   # set to a current version if needed
wget https://github.com/prometheus/prometheus/releases/download/v${PROM_VER}/prometheus-${PROM_VER}.linux-amd64.tar.gz
tar -xzf prometheus-${PROM_VER}.linux-amd64.tar.gz
cd prometheus-${PROM_VER}.linux-amd64
sudo cp prometheus promtool /usr/local/bin/
sudo cp -r consoles console_libraries /etc/prometheus/
```

**Why:** puts binaries on PATH; ships web consoles.

---

## 2) Install Node Exporter (tar binary)

```bash
cd /tmp
export NODE_VER="1.8.2"
wget https://github.com/prometheus/node_exporter/releases/download/v${NODE_VER}/node_exporter-${NODE_VER}.linux-amd64.tar.gz
tar -xzf node_exporter-${NODE_VER}.linux-amd64.tar.gz
sudo cp node_exporter-${NODE_VER}.linux-amd64/node_exporter /usr/local/bin/
```

**Why:** exposes EC2 CPU/RAM/disk metrics on port 9100.

---

## 3) Minimal Prometheus config (scrape Node Exporter)

```bash
sudo tee /etc/prometheus/prometheus.yml >/dev/null <<'YAML'
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: "node"
    static_configs:
      - targets: ["localhost:9100"]
YAML
sudo chown -R prometheus:prometheus /etc/prometheus
```

**Why:** tells Prometheus to pull metrics from your EC2.

---

## 4) Systemd services (auto-start, restart on failure)

**Prometheus**

```bash
sudo tee /etc/systemd/system/prometheus.service >/dev/null <<'EOF'
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

**Node Exporter**

```bash
sudo tee /etc/systemd/system/node_exporter.service >/dev/null <<'EOF'
[Unit]
Description=Node Exporter
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
ExecStart=/usr/local/bin/node_exporter
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

**Start both**

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter prometheus
```

**Why:** runs them as services; keeps them alive.

---

## 5) Install Grafana (quick)

**Ubuntu/Debian (.deb)**

```bash
cd /tmp
export G_VER="11.2.0"   # adjust if needed
wget https://dl.grafana.com/oss/release/grafana_${G_VER}_amd64.deb
sudo dpkg -i grafana_${G_VER}_amd64.deb
sudo systemctl enable --now grafana-server
```

*(On Amazon Linux/RHEL, use the `.rpm` and `yum install -y grafana-<ver>.x86_64.rpm`.)*
**Why:** Grafana as a managed service, minimal fuss.

---

## 6) Quick verify

```bash
curl -s localhost:9100/metrics | head   # node exporter up
curl -s localhost:9090/-/ready          # prometheus ready -> "HTTP/1.1 200 OK"
systemctl status grafana-server         # grafana running
```

Open in browser:

* Prometheus → `http://<EC2-IP>:9090`
* Grafana → `http://<EC2-IP>:3000` (login: `admin` / `admin`)

  * Add data source: **Prometheus**, URL: `http://localhost:9090`
  * Import dashboard ID **1860** (Node Exporter Full)

---

### What’s happening (super short)

* **node\_exporter** exposes EC2 host metrics → `localhost:9100`
* **prometheus** **pulls** those metrics and stores them → `:9090`
* **grafana** reads from Prometheus and visualizes → `:3000`

That’s it. If you want, next I can add **simple alerting** (high CPU) in 3–4 lines.
