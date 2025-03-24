
## 1.1 Installing prometheus as a service steps

> 1. add `Prometheus` user with no shell:

```
sudo useradd --no-create-home --shell /bin/false prometheus
```
> 2. create `Prometheus` configuration libraries directories:

```
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```
> 3. give the ownership to `Prometheus` to its library directory:
```
sudo chown prometheus:prometheus /var/lib/prometheus
```
> 4. download `Prometheus` instalation directory:

```
cd /tmp/

wget https://github.com/prometheus/prometheus/releases/download/v2.35.0/prometheus-2.35.0.linux-amd64.tar.gz
```
> 5. extraxt files:
```
tar -xvf prometheus-2.35.0.linux-amd64.tar.gz

cd prometheus-2.35.0.linux-amd64

ls
```
> 6. copy files to their quilivant configuration directory:
```
sudo mv console* /etc/prometheus

sudo mv prometheus.yml /etc/prometheus

sudo chown -R prometheus:prometheus /etc/prometheus
```
> 7. copy binary files like `promtool` and `Prometheus`:
```
sudo mv prometheus /usr/local/bin/
sudo mv promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```
> 8. finally create the service file to be able to restart the `Prometheus` service:
```
sudo vi /etc/systemd/system/prometheus.service
```
- Add This to the file
```
[Unit]
Description=Prometheus
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

> 9. finally start and enable `Prometheus` service:
```
sudo systemctl daemon-reload

sudo systemctl start prometheus

sudo systemctl status prometheus

sudo systemctl enable prometheus
```
The `Prometheus` listens on HTTP port `9090` by default.
![Prometheus Running](https://github.com/Mohamed0Mourad/Prometheus_Labs_ITI/blob/main/prometheus%20running.jpg)

# 2 - Install Node Exporter as a Service

we need to do some steps to to install `Node_Exporter`.

> 1. add `Node_Exporter` user with no shell:

```
sudo useradd --no-create-home --shell /bin/false node_exporter
```
> 2. download `Node_Exporter` instalation directory:

```
cd /tmp/

wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
```
> 3. extraxt files:
```
tar -xvf node_exporter-1.3.1.linux-amd64.tar.gz

cd node_exporter-1.3.1.linux-amd64

ls
```
> 4. copy binary files like node_exporter and give it permission:
```
sudo mv node_exporter /usr/local/bin/

sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```
> 5. finally create the service file to be able to restart the `Node_Exporter` service:
```
sudo vi /etc/systemd/system/node_exporter.service
```
- Add this to the file.
```
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

> 6. finally start and enable `Node_Exporter` service:
```
sudo systemctl daemon-reload

sudo systemctl start node_exporter

sudo systemctl status node_exporter

sudo systemctl enable node_exporter
```
The `Node_Exporter` listens on HTTP port `9100` by default.

go to `/etc/prometheus/prometheus.yml` to add another item in the scrabers section.

