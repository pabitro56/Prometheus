
- [1. Promethuse HA with thanos](#1-promethuse-ha-with-thanos)
  - [1.1. Pre-requisites for Thanos HA](#11-pre-requisites-for-thanos-ha)
    - [1.1.1. Multiple Prometheus Servers: At least two Prometheus instances scraping the same metrics.](#111-multiple-prometheus-servers-at-least-two-prometheus-instances-scraping-the-same-metrics)
  - [1.2. Components Overview](#12-components-overview)
  - [1.3. Optional](#13-optional)
- [2. Installation Process](#2-installation-process)
  - [2.1. Step 1: Download \& Install Prometheus](#21-step-1-download--install-prometheus)
    - [2.1.1. A. Download the latest version of Prometheus from the official Prometheus GitHub releases:](#211-a-download-the-latest-version-of-prometheus-from-the-official-prometheus-github-releases)
    - [2.1.2. B. Extract the downloaded file:](#212-b-extract-the-downloaded-file)
  - [2.2. Step 2: Move Prometheus Binaries to System Path](#22-step-2-move-prometheus-binaries-to-system-path)
    - [2.2.1. A. Move Prometheus and promtool binaries to a system-wide location (usually /usr/local/bin):](#221-a-move-prometheus-and-promtool-binaries-to-a-system-wide-location-usually-usrlocalbin)
  - [2.3. Step 3: Create Prometheus User and Directories](#23-step-3-create-prometheus-user-and-directories)
    - [2.3.1. A. Create a dedicated Prometheus user to run the service:](#231-a-create-a-dedicated-prometheus-user-to-run-the-service)
    - [2.3.2. B. Create the directories for Prometheus configuration and data storage:](#232-b-create-the-directories-for-prometheus-configuration-and-data-storage)
    - [2.3.3. C. Set the appropriate ownership for the directories:](#233-c-set-the-appropriate-ownership-for-the-directories)
  - [2.4. Step 4: Set Up Prometheus Configuration File](#24-step-4-set-up-prometheus-configuration-file)
  - [2.5. A. Move the configuration file to the proper location:](#25-a-move-the-configuration-file-to-the-proper-location)
  - [2.6. B. Set ownership:](#26-b-set-ownership)
  - [2.7. C. Edit the prometheus.yml file to customize your scrape jobs and rules:](#27-c-edit-the-prometheusyml-file-to-customize-your-scrape-jobs-and-rules)
  - [2.8. Step 5: Create a Systemd Service](#28-step-5-create-a-systemd-service)
    - [2.8.1. A. Create a systemd service unit file for Prometheus:](#281-a-create-a-systemd-service-unit-file-for-prometheus)
    - [2.8.2. B. Add the following configuration to the service file:](#282-b-add-the-following-configuration-to-the-service-file)
    - [2.8.3. C. Reload systemd to apply changes:](#283-c-reload-systemd-to-apply-changes)
    - [2.8.4. D. Start the Prometheus service:](#284-d-start-the-prometheus-service)
    - [2.8.5. E. Enable Prometheus to start on boot:](#285-e-enable-prometheus-to-start-on-boot)
  - [2.9. Step 6: Access Prometheus UI](#29-step-6-access-prometheus-ui)
    - [2.9.1. Open your browser and navigate to](#291-open-your-browser-and-navigate-to)
- [3. Thanos Sidecar for Prometheus1](#3-thanos-sidecar-for-prometheus1)
    - [3.0.1. Thanos Sidecar for Prometheus1 (192.168.72.10:9090)](#301-thanos-sidecar-for-prometheus1-19216872109090)
    - [3.0.2. Add the following configuration:](#302-add-the-following-configuration)
    - [3.0.3. Start the Thanos sidecar:](#303-start-the-thanos-sidecar)
    - [3.0.4. Run Thanos Sidecar for Prometheus1:](#304-run-thanos-sidecar-for-prometheus1)
    - [3.0.5. Thanos Sidecar for Prometheus2 (192.168.72.11:9095)](#305-thanos-sidecar-for-prometheus2-19216872119095)
      - [3.0.5.1. Run Thanos Sidecar for Prometheus2:](#3051-run-thanos-sidecar-for-prometheus2)
- [4. Deploy Thanos Query for Aggregation (192.168.72.14)](#4-deploy-thanos-query-for-aggregation-1921687214)
  - [4.1. A. Download the Thanos binaries:](#41-a-download-the-thanos-binaries)
  - [4.2. B. Extract the binaries:](#42-b-extract-the-binaries)
  - [4.3. C. Move Thanos binary to a system-wide location:](#43-c-move-thanos-binary-to-a-system-wide-location)
  - [4.4. D. Create a user for Thanos:](#44-d-create-a-user-for-thanos)
  - [4.5. E. Create directories for Thanos configuration and data storage:](#45-e-create-directories-for-thanos-configuration-and-data-storage)
  - [4.6. F. Run Thanos Query to aggregate data from both Prometheus instances:](#46-f-run-thanos-query-to-aggregate-data-from-both-prometheus-instances)
- [5. Access web browser](#5-access-web-browser)
- [6. Alert Manager](#6-alert-manager)
- [7. Alertmanager configuration](#7-alertmanager-configuration)
- [8. How to Prevent Duplicate Alerts](#8-how-to-prevent-duplicate-alerts)
  - [8.1. Configure External Labels in Prometheus](#81-configure-external-labels-in-prometheus)
    - [8.1.1. For Prometheus1 (192.168.72.10):](#811-for-prometheus1-1921687210)
    - [8.1.2. For Prometheus2 (192.168.72.11):](#812-for-prometheus2-1921687211)
- [9. How This Works](#9-how-this-works)

# 1. Promethuse HA with thanos
## 1.1. Pre-requisites for Thanos HA


### 1.1.1. Multiple Prometheus Servers: At least two Prometheus instances scraping the same metrics.

**Thanos Binaries:** Download Thanos binaries on all relevant servers.

**Prometheus Data Directory:** Ensure correct tsdb.path for Thanos Sidecar.

**Thanos Sidecar:** Deploy on both Prometheus servers.

**Network Configuration:** Ensure communication between Prometheus, Thanos Sidecars, and Thanos Query.

**Thanos Query Deployment:** Deploy Thanos Query on a separate or Prometheus server.

**Optional Storage:** Configure object storage for long-term storage if needed.

**Version Compatibility:** Ensure your Prometheus version is compatible with thanos.

## 1.2. Components Overview

**Prometheus:** Metrics scraping and local storage.

**Thanos Sidecar:** Runs alongside each Prometheus instance. It uploads data to object storage and allows for queries across Prometheus replicas.

**Thanos Query:** Allows querying of data from multiple Prometheus instances and the object storage, merging results from replicas.

## 1.3. Optional

**Thanos Store:** Serves data from long-term object storage (e.g., S3, GCS).

**Thanos Compactor:** Handles compaction of blocks and downsampling for efficient storage over time.

**Thanos Ruler:** Runs Prometheus-like alerting and recording rules on top of Thanos’ data.


# 2. Installation Process
## 2.1. Step 1: Download & Install Prometheus

### 2.1.1. A. Download the latest version of Prometheus from the official Prometheus GitHub releases:
```
cd /tmp 
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.47.0/prometheus-2.47.0.linux-amd64.tar.gz
```


### 2.1.2. B. Extract the downloaded file:

`tar -xvzf prometheus-2.47.0.linux-amd64.tar.gz cd prometheus-2.47.0.linux-amd64`


## 2.2. Step 2: Move Prometheus Binaries to System Path

### 2.2.1. A. Move Prometheus and promtool binaries to a system-wide location (usually /usr/local/bin):

`mv prometheus /usr/local/bin/ sudo mv promtool /usr/local/bin/`


## 2.3. Step 3: Create Prometheus User and Directories

### 2.3.1. A. Create a dedicated Prometheus user to run the service:

`useradd --no-create-home --shell /bin/false prometheus`

### 2.3.2. B. Create the directories for Prometheus configuration and data storage:

`mkdir /etc/prometheus sudo mkdir /var/lib/prometheus`


### 2.3.3. C. Set the appropriate ownership for the directories:
```
chown prometheus:prometheus /etc/prometheus 
chown prometheus:prometheus /var/lib/prometheus
```

## 2.4. Step 4: Set Up Prometheus Configuration File

## 2.5. A. Move the configuration file to the proper location:

`mv prometheus.yml /etc/prometheus/`

## 2.6. B. Set ownership:

`chown prometheus:prometheus /etc/prometheus/prometheus.yml`


## 2.7. C. Edit the prometheus.yml file to customize your scrape jobs and rules:
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

```


## 2.8. Step 5: Create a Systemd Service

### 2.8.1. A. Create a systemd service unit file for Prometheus:

`vim /etc/systemd/system/prometheus.service`

### 2.8.2. B. Add the following configuration to the service file:

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

### 2.8.3. C. Reload systemd to apply changes:

`systemctl daemon-reload`

### 2.8.4. D. Start the Prometheus service:
`systemctl start prometheus`

### 2.8.5. E. Enable Prometheus to start on boot:
`systemctl enable prometheus`



## 2.9. Step 6: Access Prometheus UI

###  2.9.1. Open your browser and navigate to
```
http://192.168.72.10:9090

http://192.168.72.11:9090
```

# 3. Thanos Sidecar for Prometheus1
### 3.0.1. Thanos Sidecar for Prometheus1 (192.168.72.10:9090)

```
Download Thanos from the official releases page.
Start the Thanos sidecar on both Prometheus servers:
Create a systemd service for Thanos sidecar:
vim /etc/systemd/system/thanos-sidecar.service
```

### 3.0.2. Add the following configuration:
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


### 3.0.3. Start the Thanos sidecar:
```
systemctl daemon-reload 
systemctl start thanos-sidecar
systemctl enable thanos-sidecar
```


### 3.0.4. Run Thanos Sidecar for Prometheus1:

`/thanos sidecar --tsdb.path /var/lib/prometheus  --prometheus.url http://192.168.72.10:9090 --grpc-address 0.0.0.0:10901 --http-address 0.0.0.0:10902 &`

### 3.0.5. Thanos Sidecar for Prometheus2 (192.168.72.11:9095)

#### 3.0.5.1. Run Thanos Sidecar for Prometheus2:

`./thanos sidecar --tsdb.path /var/lib/prometheus  --prometheus.url http://192.168.72.11:9090 --grpc-address 0.0.0.0:10901 --http-address 0.0.0.0:10902 &`


# 4. Deploy Thanos Query for Aggregation (192.168.72.14)

Download Thanos Query from the official releases page.

We can execute any of prometheus server or any other server.

## 4.1. A. Download the Thanos binaries:
```
cd /tmp 
curl -LO https://github.com/thanos-io/thanos/releases/download/v0.32.0/thanos-0.32.0.linux-amd64.tar.gz
```


## 4.2. B. Extract the binaries:

`tar -xvzf thanos-0.32.0.linux-amd64.tar.gz cd thanos-0.32.0.linux-amd64`





## 4.3. C. Move Thanos binary to a system-wide location:

`mv thanos /usr/local/bin/`

## 4.4. D. Create a user for Thanos:

`useradd --no-create-home --shell /bin/false thanos`

## 4.5. E. Create directories for Thanos configuration and data storage:
```
mkdir /etc/thanos 
mkdir /var/lib/thanos 
chown thanos:thanos /etc/thanos /var/lib/thanos
```
## 4.6. F. Run Thanos Query to aggregate data from both Prometheus instances:

`./thanos query --http-address 0.0.0.0:9096 --grpc-address 0.0.0.0:10903 --store 192.168.72.10:10901 --store 192.168.72.11:10901 --query.replica-label=prometheus_replica &`


# 5. Access web browser

`http://192.168.72.14:9096`



# 6. Alert Manager
>[!NOTE]
    Alertmanager on Prometheus1 (192.168.72.10:9093) is the central Alertmanager for both Prometheus instances. <br>
    Prometheus2 is configured to send alerts to the Alertmanager on Prometheus1, ensuring that alerts from both Prometheus servers are routed through the same Alertmanager.<br>
    In Prometheus Server2 (192.168.72.11), you should configure Alertmanager to point to the same Alertmanager instance that is running on Prometheus Server1 (192.168.72.10:9093). This ensures that both Prometheus servers send alerts to the same Alertmanager, which handles deduplication and alert routing.<br>
    Prometheus Server2 (192.168.72.11) - Alertmanager Configuration<br>
    In the prometheus.yml file on Prometheus2, add the following alerting configuration to point to the Alertmanager on Prometheus1 (192.168.72.10:9093):<br>

# 7. Alertmanager configuration
```
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 192.168.72.10:9093
```

By using this configuration, Alertmanager will handle deduplication of alerts and route them according to your setup.


To prevent duplicate alerts from being sent by both Prometheus instances to the Alertmanagers, you need to configure external labels in each Prometheus server's prometheus.yml. These labels help Alertmanager identify that the same alert coming from multiple Prometheus servers is actually the same, allowing it to deduplicate the alerts.

# 8. How to Prevent Duplicate Alerts

## 8.1. Configure External Labels in Prometheus

In your prometheus.yml file on both Prometheus servers, add unique external_labels for each Prometheus instance. This ensures that Alertmanager can identify the source of the alerts and deduplicate them.

### 8.1.1. For Prometheus1 (192.168.72.10):
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


### 8.1.2. For Prometheus2 (192.168.72.11):
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



# 9. How This Works

When both Prometheus1 and Prometheus2 fire the same alert, they will include their external labels in the alert.

Alertmanager will deduplicate these alerts by comparing them. Even though the alerts are coming from different Prometheus instances, Alertmanager will treat them as the same alert if their content (alert name, labels, etc.) is identical.

The external label is just used to track which Prometheus instance sent the alert, but it won't cause duplicate notifications.

By following this configuration, Alertmanager will deduplicate alerts coming from both Prometheus servers, ensuring that you don't receive the same alert twice.