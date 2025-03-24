
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

2 - Install Node Exporter as a Service]
