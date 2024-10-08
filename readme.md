
# Promethuse HA with thanos
## Pre-requisites for Thanos HA


### Multiple Prometheus Servers: At least two Prometheus instances scraping the same metrics.

**Thanos Binaries:** Download Thanos binaries on all relevant servers.

**Prometheus Data Directory:** Ensure correct tsdb.path for Thanos Sidecar.

**Thanos Sidecar:** Deploy on both Prometheus servers.

**Network Configuration:** Ensure communication between Prometheus, Thanos Sidecars, and Thanos Query.

**Thanos Query Deployment:** Deploy Thanos Query on a separate or Prometheus server.

**Optional Storage:** Configure object storage for long-term storage if needed.

**Version Compatibility:** Ensure your Prometheus version is compatible with thanos.

## Components Overview

**Prometheus:** Metrics scraping and local storage.

**Thanos Sidecar:** Runs alongside each Prometheus instance. It uploads data to object storage and allows for queries across Prometheus replicas.

**Thanos Query:** Allows querying of data from multiple Prometheus instances and the object storage, merging results from replicas.

## Optional

**Thanos Store:** Serves data from long-term object storage (e.g., S3, GCS).

**Thanos Compactor:** Handles compaction of blocks and downsampling for efficient storage over time.

**Thanos Ruler:** Runs Prometheus-like alerting and recording rules on top of Thanos’ data.


# Installation Process
## Step 1: Download & Install Prometheus

### A. Download the latest version of Prometheus from the official Prometheus GitHub releases:
```
cd /tmp 
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.47.0/prometheus-2.47.0.linux-amd64.tar.gz
```


### B. Extract the downloaded file:

`tar -xvzf prometheus-2.47.0.linux-amd64.tar.gz cd prometheus-2.47.0.linux-amd64`


## Step 2: Move Prometheus Binaries to System Path

### A. Move Prometheus and promtool binaries to a system-wide location (usually /usr/local/bin):

`mv prometheus /usr/local/bin/ sudo mv promtool /usr/local/bin/`


## Step 3: Create Prometheus User and Directories

### A. Create a dedicated Prometheus user to run the service:

`useradd --no-create-home --shell /bin/false prometheus`

### B. Create the directories for Prometheus configuration and data storage:

`mkdir /etc/prometheus sudo mkdir /var/lib/prometheus`


### C. Set the appropriate ownership for the directories:
```
chown prometheus:prometheus /etc/prometheus 
chown prometheus:prometheus /var/lib/prometheus
```

## Step 4: Set Up Prometheus Configuration File

## A. Move the configuration file to the proper location:

`mv prometheus.yml /etc/prometheus/`

## B. Set ownership:

`chown prometheus:prometheus /etc/prometheus/prometheus.yml`


## C. Edit the prometheus.yml file to customize your scrape jobs and rules:
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

```


## Step 5: Create a Systemd Service

### A. Create a systemd service unit file for Prometheus:

`vim /etc/systemd/system/prometheus.service`

### B. Add the following configuration to the service file:

```
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/ \
  --web.console.templates=/usr/share/prometheus/consoles \
  --web.console.libraries=/usr/share/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

### C. Reload systemd to apply changes:

`systemctl daemon-reload`

### D. Start the Prometheus service:
`systemctl start prometheus`

### E. Enable Prometheus to start on boot:
`systemctl enable prometheus`



## Step 6: Access Prometheus UI

###  Open your browser and navigate to
```
http://192.168.72.10:9090

http://192.168.72.11:9090
```

# Thanos Sidecar for Prometheus1
### Thanos Sidecar for Prometheus1 (192.168.72.10:9090)

```
Download Thanos from the official releases page.
Start the Thanos sidecar on both Prometheus servers:
Create a systemd service for Thanos sidecar:
vim /etc/systemd/system/thanos-sidecar.service
```

### Add the following configuration:
```
[Unit]
Description=Thanos Sidecar
Wants=network-online.target
After=network-online.target

[Service]
User=thanos
Group=thanos
ExecStart=/tmp/thanos-0.32.0.linux-amd64/thanos sidecar \
  --tsdb.path /var/lib/prometheus \
  --prometheus.url http://localhost:9090 \
  --grpc-address 0.0.0.0:10901 \
  --http-address 0.0.0.0:10902

[Install]
WantedBy=multi-user.target
```


### Start the Thanos sidecar:
```
systemctl daemon-reload 
systemctl start thanos-sidecar
systemctl enable thanos-sidecar
```


### Run Thanos Sidecar for Prometheus1:

`/thanos sidecar --tsdb.path /var/lib/prometheus  --prometheus.url http://192.168.72.10:9090 --grpc-address 0.0.0.0:10901 --http-address 0.0.0.0:10902 &`

### Thanos Sidecar for Prometheus2 (192.168.72.11:9095)

#### Run Thanos Sidecar for Prometheus2:

`./thanos sidecar --tsdb.path /var/lib/prometheus  --prometheus.url http://192.168.72.11:9090 --grpc-address 0.0.0.0:10901 --http-address 0.0.0.0:10902 &`


# Deploy Thanos Query for Aggregation (192.168.72.14)

Download Thanos Query from the official releases page.

We can execute any of prometheus server or any other server.

## A. Download the Thanos binaries:
`
cd /tmp 
curl -LO https://github.com/thanos-io/thanos/releases/download/v0.32.0/thanos-0.32.0.linux-amd64.tar.gz
`


## B. Extract the binaries:

`tar -xvzf thanos-0.32.0.linux-amd64.tar.gz cd thanos-0.32.0.linux-amd64`





## C. Move Thanos binary to a system-wide location:

`mv thanos /usr/local/bin/`

## D. Create a user for Thanos:

`useradd --no-create-home --shell /bin/false thanos`

## E. Create directories for Thanos configuration and data storage:
```
mkdir /etc/thanos 
mkdir /var/lib/thanos 
chown thanos:thanos /etc/thanos /var/lib/thanos
```
## F. Run Thanos Query to aggregate data from both Prometheus instances:

`./thanos query --http-address 0.0.0.0:9096 --grpc-address 0.0.0.0:10903 --store 192.168.72.10:10901 --store 192.168.72.11:10901 --query.replica-label=prometheus_replica &`


# Access web browser

`http://192.168.72.14:9096`



# Alert Manager
>[!NOTE]
    Alertmanager on Prometheus1 (192.168.72.10:9093) is the central Alertmanager for both Prometheus instances. <br>
    Prometheus2 is configured to send alerts to the Alertmanager on Prometheus1, ensuring that alerts from both Prometheus servers are routed through the same Alertmanager.<br>
    In Prometheus Server2 (192.168.72.11), you should configure Alertmanager to point to the same Alertmanager instance that is running on Prometheus Server1 (192.168.72.10:9093). This ensures that both Prometheus servers send alerts to the same Alertmanager, which handles deduplication and alert routing.<br>
    Prometheus Server2 (192.168.72.11) - Alertmanager Configuration<br>
    In the prometheus.yml file on Prometheus2, add the following alerting configuration to point to the Alertmanager on Prometheus1 (192.168.72.10:9093):<br>

# Alertmanager configuration
```
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 192.168.72.10:9093
```

By using this configuration, Alertmanager will handle deduplication of alerts and route them according to your setup.


To prevent duplicate alerts from being sent by both Prometheus instances to the Alertmanagers, you need to configure external labels in each Prometheus server's prometheus.yml. These labels help Alertmanager identify that the same alert coming from multiple Prometheus servers is actually the same, allowing it to deduplicate the alerts.

# How to Prevent Duplicate Alerts

## Configure External Labels in Prometheus

In your prometheus.yml file on both Prometheus servers, add unique external_labels for each Prometheus instance. This ensures that Alertmanager can identify the source of the alerts and deduplicate them.

### For Prometheus1 (192.168.72.10):
```
global:
  external_labels:
    prometheus_instance: "prometheus1"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 192.168.72.10:9093  # Alertmanager on Prometheus1
          - 192.168.72.11:9093  # Alertmanager on Prometheus2

```


### For Prometheus2 (192.168.72.11):
```
global:
  external_labels:
    prometheus_instance: "prometheus2"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 192.168.72.10:9093  # Alertmanager on Prometheus1
          - 192.168.72.11:9093  # Alertmanager on Prometheus2
```



# How This Works

When both Prometheus1 and Prometheus2 fire the same alert, they will include their external labels in the alert.

Alertmanager will deduplicate these alerts by comparing them. Even though the alerts are coming from different Prometheus instances, Alertmanager will treat them as the same alert if their content (alert name, labels, etc.) is identical.

The external label is just used to track which Prometheus instance sent the alert, but it won't cause duplicate notifications.

By following this configuration, Alertmanager will deduplicate alerts coming from both Prometheus servers, ensuring that you don't receive the same alert twice.