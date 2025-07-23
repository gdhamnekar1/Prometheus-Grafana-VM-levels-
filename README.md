# Prometheus-Grafana-VM-levels-

To use **Prometheus** and **Grafana** on your Azure VM (with configuration `Standard_B2s_v2`), you'll follow these main steps:

---

### ✅ Prerequisites:

1. **Linux-based VM** (e.g., Ubuntu 20.04/22.04)
2. **VM has outbound internet access**
3. **VM is updated**
4. **Inbound NSG rules** open for:

   * SSH (`22`)
   * Grafana UI (`3000`)
   * Prometheus (`9090`)
5. You have **sudo/root access**

---

## 🔧 Step-by-step Procedure

---

### 1️⃣ Update and Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install wget curl vim -y
```

---

### 2️⃣ Install Prometheus

#### 📦 Download and Extract Prometheus

```bash
cd /opt
sudo wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
sudo tar -xvf prometheus-2.52.0.linux-amd64.tar.gz
sudo mv prometheus-2.52.0.linux-amd64 prometheus
```

#### 🛠 Setup Prometheus Configuration

```bash
cd /opt/prometheus
sudo vim prometheus.yml
```

Make sure it has something like:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

#### ▶ Start Prometheus

```bash
./prometheus --config.file=prometheus.yml
```

> Prometheus will be running on: **http\://<vm-ip>:9090**

---

### 3️⃣ Install Grafana

#### 📦 Add Grafana Repo and Install

```bash
sudo apt install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install grafana -y
```

#### ▶ Start and Enable Grafana

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

> Grafana will be running on: **http\://<vm-ip>:3000**

Default credentials:

* **Username:** `admin`
* **Password:** `admin` (you’ll be prompted to change it)

---

### 4️⃣ Configure Grafana to Use Prometheus

1. Access Grafana via `http://<your-vm-ip>:3000`
2. Go to **Settings > Data Sources > Add data source**
3. Choose **Prometheus**
4. Set:

   * **URL**: `http://localhost:9090`
5. Click **Save & Test**

---

### 5️⃣ (Optional) Add Dashboards

* Go to **Create > Import**
* Use community dashboard IDs like:

  * Linux Server: `13978`
  * Node Exporter Full: `1860`

---

### 6️⃣ (Optional) Install Node Exporter for System Metrics

```bash
cd /opt
sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.8.0/node_exporter-1.8.0.linux-amd64.tar.gz
sudo tar -xvzf node_exporter-1.8.0.linux-amd64.tar.gz
sudo mv node_exporter-1.8.0.linux-amd64 node_exporter
```

Run it:

```bash
/opt/node_exporter/node_exporter &
```

Then add this to Prometheus config:

```yaml
- job_name: 'node_exporter'
  static_configs:
    - targets: ['localhost:9100']
```

Restart Prometheus and you’ll get system-level metrics.

---

### ✅ Final Checklist

| Component     | Port | URL                            | Notes                 |
| ------------- | ---- | ------------------------------ | --------------------- |
| Prometheus    | 9090 | http\://\<VM\_IP>:9090         | Monitoring system     |
| Grafana       | 3000 | http\://\<VM\_IP>:3000         | Dashboard UI          |
| Node Exporter | 9100 | http\://\<VM\_IP>:9100/metrics | System metrics for VM |

---

Would you like a **systemd setup** for Prometheus and Node Exporter for persistent service management?
