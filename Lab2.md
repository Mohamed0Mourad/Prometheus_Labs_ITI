## 1-How do I trigger a Prometheus alert?

### **A. Define Alerting Rules in Prometheus**
Alert rules are defined in a Prometheus rule file (typically `*.rules.yml`) and loaded via `prometheus.yml`.

#### **Example Rule File (`alert.rules.yml`)**  
```yaml
groups:
- name: example-alerts
  rules:
  - alert: HighCpuUsage
    expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100 > 80
    for: 5m  # Alert only if condition persists for 5 minutes
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU usage is {{ $value }}% (above 80%) for 5 minutes."
```

- **`expr`**: The PromQL query that triggers the alert when the condition is met.  
- **`for`**: Wait for the condition to persist before firing (avoids flapping).  
- **`labels`**: Add metadata (e.g., severity).  
- **`annotations`**: Human-readable details for notifications.

---

### **B. Load Rules in `prometheus.yml`**
```yaml
rule_files:
  - "alert.rules.yml"
```
---

### **C. Configure Alertmanager**
Alertmanager handles routing, deduplication, and notifications (email, Slack, PagerDuty, etc.).

#### **Example `alertmanager.yml`**
```yaml
route:
  receiver: slack-notifications
  group_by: [alertname, severity]
  group_wait: 30s
  group_interval: 5m

receivers:
- name: slack-notifications
  slack_configs:
  - api_url: "https://hooks.slack.com/services/XXXXX"
    channel: "#alerts"
    send_resolved: true
```

Start Alertmanager:
```bash
./alertmanager --config.file=alertmanager.yml
```

### 2-What is the difference between node exporter and mysql exporter?

### ** Node Exporter**  
**Purpose**: Collects **hardware and OS-level metrics** from a machine (Linux/Windows).  
**Use Case**: Monitor server health (CPU, RAM, disk, network, etc.).  

**Key Metrics Collected**:  
- CPU usage (`node_cpu_seconds_total`)  
- Memory usage (`node_memory_MemAvailable_bytes`)  
- Disk I/O (`node_disk_read_bytes_total`)  
- Network traffic (`node_network_receive_bytes_total`)  
- Load average (`node_load1`)
- 
### ** MySQL Exporter**  
**Purpose**: Collects **database-specific metrics** from MySQL/MariaDB.  
**Use Case**: Monitor query performance, connections, replication lag, etc.  

**Key Metrics Collected**:  
- Query throughput (`mysql_global_status_questions`)  
- Connection count (`mysql_global_status_threads_connected`)  
- Replication status (`mysql_slave_status_seconds_behind_master`)  
- InnoDB buffer pool stats (`mysql_innodb_buffer_pool_read_requests`)  

### **Key Differences**  
| Feature               | Node Exporter                     | MySQL Exporter                     |  
|-----------------------|-----------------------------------|------------------------------------|  
| **Target**            | OS/Hardware                      | MySQL Database                    |  
| **Metrics**           | CPU, RAM, Disk, Network          | Queries, Connections, Locks, etc. |  
| **Port**              | `9100` (default)                 | `9104` (default)                  |  
| **Dependency**        | Runs on the host                 | Needs MySQL credentials           |  
| **Example Query**     | `node_memory_free_bytes`         | `mysql_global_status_uptime`      |  

---

### **When to Use Which?**  
- Use **Node Exporter** to monitor server health (e.g., "Is the server out of RAM?").  
- Use **MySQL Exporter** to monitor database performance (e.g., "Is replication lagging?").  

### 3 What is the maximum retention period for saving data in Prometheus, and how can it be increased?
By default, Prometheus stores time-series data for 15 days. However, you can configure this based on your storage requirements.
Configure at Startup:
Modify Prometheus's startup command (prometheus.yml is not used for retention settings):
```yml
# Set retention to 90 days (example)
prometheus \
  --config.file="/etc/prometheus/prometheus.yml" \
  --storage.tsdb.path="/var/lib/prometheus/data" \
  --storage.tsdb.retention.time=90d
```

### 4-What are the different PromQL data types available in the Prometheus Expression language?

### **PromQL Data Types in Prometheus**  
## **1. Instant Vector (Most Common)**  
- **Definition**: A set of time series with a single data point **at the latest timestamp** (instant snapshot).  
- **Use Case**: Current values of metrics (e.g., CPU usage now).  
- **Example**:  
  ```promql
  node_cpu_seconds_total{mode="idle"}  # Returns idle CPU time per core (latest value)
  ```

## **2. Range Vector**  
- **Definition**: A set of time series with **multiple data points over a specified time window**.  
- **Use Case**: Calculations over time (e.g., rate of requests in the last 5 minutes).  
- **Example**:  
  ```promql
  http_requests_total[5m]  # Returns all HTTP requests over the last 5 minutes
  ```
- **Key Functions**:  
  - `rate()`, `irate()` (for counters)  
  - `avg_over_time()`, `max_over_time()` (for gauges)  

## **3. Scalar**  
- **Definition**: A single numeric value (no labels or timestamps).  
- **Use Case**: Arithmetic operations or comparisons.  
- **Example**:  
  ```promql
  42  # Literal scalar
  count(node_cpu_seconds_total) > 10  # Returns 0 (false) or 1 (true)
  ```

## **4. String**  
- **Definition**: A simple text value (rarely used in queries; mainly for annotations).  
- **Use Case**: Label manipulation or debug output.  
- **Example**:  
  ```promql
  "This is a string"  # Only used in rare cases (e.g., `label_replace()`)
  ```

### **Key Differences Summary**  
| Type          | Example                          | Used With                          |  
|---------------|----------------------------------|------------------------------------|  
| **Instant Vector** | `up{job="node"}`              | Alerts, dashboards (current state) |  
| **Range Vector**   | `http_requests_total[5m]`     | `rate()`, `avg_over_time()`        |  
| **Scalar**         | `3.14` or `count(...)`        | Math operations (`+`, `-`, `*`)    |  
| **String**         | `"debug"`                     | Rare (annotations, `label_replace`)|  

### 5-How Can the average request duration over the last 5 minutes from a histogram be calculated?
### **Simple Explanation: Calculate Average Request Duration (Last 5 mins) from a Histogram**  

- A histogram metric (e.g., `http_request_duration_seconds`) tracks:  
  - **`_count`** → Total number of requests.  
  - **`_sum`** → Sum of all request durations (e.g., 100 requests took 50 seconds total).  

#### **PromQL Query (Last 5 Minutes)**  
```promql
  rate(http_request_duration_seconds_sum[5m])  
/  
  rate(http_request_duration_seconds_count[5m])
```  

### 6-What is Thanos Prometheus?

### **Thanos: The Scalable Prometheus**  

**Thanos** is an open-source project that extends **Prometheus** to solve its limitations in **long-term storage, high availability, and global querying**—without breaking existing setups.  

---

## **Key Problems Thanos Solves**  
1. **Limited Retention**  
   - Prometheus stores data only for **weeks/months** (local disk).  
   - Thanos uses **object storage** (S3, GCS, etc.) for **years of retention**.  

2. **No Global View**  
   - Prometheus is single-server; Thanos **aggregates data** from multiple Prometheus instances.  

3. **High Availability (HA)**  
   - Thanos provides **deduplication** and **redundancy** for metrics.  

4. **Downsampling**  
   - Compresses old data to save space while keeping trends visible.  

---

## **Core Features**  
| Component          | Role                                                                 |
|--------------------|----------------------------------------------------------------------|
| **Sidecar**        | Attaches to Prometheus, uploads data to object storage.              |
| **Store Gateway**  | Lets you query historical data from object storage.                  |
| **Query (Querier)**| Merges data from Prometheus + object storage for a **global view**.  |
| **Compactor**      | Downsamples old data (e.g., 5m resolution for data older than 2w).  |
| **Ruler**          | Handles alerts/recording rules across multiple Prometheus instances. |

---
## **When to Use Thanos?**  
✅ Need **years of metrics** (not just 15 days).  
✅ Have **multiple Prometheus servers** (e.g., per region).  
✅ Want a **single dashboard** for all data.  

---

### 7 What is promtool and how i can use it ? 
**`promtool`** is a command-line utility bundled with Prometheus for **validating configuration files, testing rules, and debugging metrics**.    

#### **1. Validate `prometheus.yml`**  
Check for syntax errors:  
```sh
promtool check config prometheus.yml
```
#### **2. Test Alerting/Recording Rules**  
Verify your `.rules` files:  
```sh
promtool check rules alert.rules.yml
```
#### **3. Query Prometheus from CLI**  
Fetch metrics without a browser:  
```sh
promtool query instant http://localhost:9090 'up{job="node"}'
```
#### **4. Debug TSDB (Time-Series Database)**  
Inspect local storage:  
```sh
promtool tsdb analyze /path/to/data
```
#### **5. Verify Metric Labels**  
Check for naming conventions:  
```sh
promtool check metrics /path/to/metrics.txt
```

Run `promtool --help` for more options! 🛠️

### 8-What types of Monitoring can be done via Grafana?

Grafana is a powerful **visualization and observability** tool that supports monitoring across multiple data sources. Here’s a brief breakdown of what you can monitor:
---

### **1. Infrastructure Monitoring**  
- **Servers**: CPU, RAM, Disk, Network (via Prometheus, InfluxDB, Telegraf).  
- **Containers & Kubernetes**: Pods, Nodes, Deployments (using Prometheus, Datadog, or Grafana Loki).  
- **Cloud Services**: AWS CloudWatch, Azure Monitor, GCP Stackdriver.  


### **2. Application Performance Monitoring (APM)**  
- **Latency, Errors, Requests** (using Jaeger, Tempo, Elastic APM).  
- **Database Performance**: Query times, connections (MySQL, PostgreSQL, MongoDB).  
- **Microservices**: Trace requests across services (OpenTelemetry, Zipkin).  

**Example Query (PromQL)**:  
```promql
rate(http_requests_total[5m])
```
### **3. Log Monitoring & Analysis**  
- **Centralized Logs**: Filter and visualize logs (Grafana Loki, ELK Stack).  
- **Error Tracking**: Correlate logs with metrics (e.g., Loki + Prometheus).  

### **4. Network Monitoring**  
- **Bandwidth, Latency, Packet Loss** (via SNMP, Prometheus, or InfluxDB).  
- **Firewall & Load Balancers** (Palo Alto, F5, Nginx).  


### **5. Real-Time & Synthetic Monitoring**  
- **Uptime Checks**: HTTP, ICMP, DNS (Grafana Synthetic Monitoring, Blackbox Exporter).  
- **User Experience (RUM)**: Frontend performance (Google Lighthouse, Sentry).  
**Example Alert**:  
```
Website downtime > 2 minutes.
```
### **6. Business & Custom Metrics**  
- **Revenue, User Signups, Conversion Rates** (SQL databases, Google Analytics).  
- **IoT/Sensor Data** (InfluxDB, MQTT).  


### 9-Can we see different Servers CPU comparison in Grafana?
Yes! You Can Compare CPU Usage Across Multiple Servers in Grafana
Grafana makes it easy to visualize and compare CPU metrics from different servers in a single dashboard


