# Coda Performance Dashboard

> This page is still under development
> also, this repository is temporary and can be changed.

Coda Performance Dashboard is performance monitoring for Coda Protocol. It privides two functions, Block Producer Performance Dashboard and Snarker Performance Dashboard.

## Features
- Block Producer Performance Dashboard
- Snarker Performance Dashboard


![스크린샷 2020-07-24 오후 8.08.09.png](/Users/jongkwang/Desktop/스크린샷 2020-07-24 오후 8.08.09.png)


## Install Guide

The Coda Performance Dashboard displays information from nodes through Prometheus and Grafana.

### Prerequisites

- Coda Protocol nodes of Block Producer and Snarker
  - Reference : https://codaprotocol.com/docs/getting-started


### Install Docker
> Prometheus and Grafana provides services through Docker.

```
apt install docker.io
```

### Configure Prometheus
> Save Prometheum configuration file and run it based on Docker.

Create work folder and save Prometheum configuration file
```
mkdir /var/prometheus-coda
cd /var/prometheus-coda
vi /var/prometheus-coda/prometheus-coda.yml
```

Paste the contents of the file below. At this time, change <code>NODE_IP_ADDRESS</code> accordingly.
```
global:
  scrape_interval: 15s # 15s will be enough, because block of time
  scrape_timeout: 15s
  evaluation_interval: 15s # Evaluate alerting
alerting:
  alertmanagers:
  - static_configs:
    - targets: []
    scheme: http
    timeout: 5s
scrape_configs:
  - job_name: 'coda-testnet'
    metrics_path: /metrics
    scheme: http
    static_configs:
      - targets: ['NODE_IP_ADDRESS:6060'] ## Replace IP address to your server
        labels:
          hostname: 'coda-daemon'
```

> You can also get this file(prometheus-coda.yml) here.


Run Prometheus through Docker.
```
docker run \
	-d --name=prometheus-coda \
	-p 9090:9090 \
	-v /var/prometheus-coda:/prometheus \
	prom/prometheus --config.file=/prometheus/prometheus-coda.yml
```

Check Prometheus : <code>http://PROMETHEUS_IP_ADDRESS:9090</code>
[Image]

### Start Grafana

Run Grafana through Docker

```
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

### Firewall Setting
- CODA node
  - Open inbound 19090 port from PROMETHEUM Server
- Prometheum and Grafana Server
  - Open inbound 3000(Grafana) and 9090(Prometheum) port

### Configure Grafana

Login to Grafana
- Visit : <code>http://GRAFANA_IP_ADDRESS:3000</code>
- You can login to Grafana using user `admin` and default password is also `admin`. when you connect to Grafana first time. When you login, you will see a screen to set new password for `admin`.

Add data source
- <code>Configuration > Data Sources > Add data source > Prometheus</code>
- Type it below
  - Name : <code>Prometheus-Coda</code>
  - HTTP > URL : <code>http://CODA_NODE_IP_ADDRESS:19090</code>
- Click <code>[Save & Test]</code>

Add a dashboard from template
- Move to <code>Create > Import JSON</code>
- Upload this file : [FILE]

Now, You can see your dashboard
- <code>Dashboard > Home > Coda:BlockProducer Performance Dashboard</code>

