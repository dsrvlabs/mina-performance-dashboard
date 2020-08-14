# Coda Performance Dashboard

> This page is still under development. 
> Also, this repository is temporary and is subject to change.

Coda Performance Dashboard is a performance monitoring tool for Coda Protocol. It provides two functions, collecting and visualizing Block Producers' and Snarkers' performance data.

## Plan (milestone)
- Milestone 1 (alpha) : until July 24 ![Progress](https://progress-bar.dev/100/?title=completed)
  - Prepare Code infrastructure (node) [![Issue Status](https://img.shields.io/badge/closed-lightgrey.svg)](https://github.com/dsrvlabs/coda-performance-dashboard)
  - Initial Prometheus and Grafana setup [![Issue Status](https://img.shields.io/badge/closed-lightgrey.svg)](https://github.com/dsrvlabs/coda-performance-dashboard)
  - Showing raw metrics [![Issue Status](https://img.shields.io/badge/closed-lightgrey.svg)](https://github.com/dsrvlabs/coda-performance-dashboard)
  - Basic integration for internal release [![Issue Status](https://img.shields.io/badge/closed-lightgrey.svg)](https://github.com/dsrvlabs/coda-performance-dashboard)
- Milestone 2 (beta) : until Aug 21
  - Enhanced Prometheus and Grafana setup [#2](https://github.com/dsrvlabs/coda-performance-dashboard/issues/2) ![Progress](https://progress-bar.dev/20/?title=progress) 
  - Showing user-friendly derived metrics [#3](https://github.com/dsrvlabs/coda-performance-dashboard/issues/3)
  - Improve UI [#4](https://github.com/dsrvlabs/coda-performance-dashboard/issues/4)
  - Advanced Integration for public release [#5](https://github.com/dsrvlabs/coda-performance-dashboard/issues/5)
  - Documentation (draft) [#6](https://github.com/dsrvlabs/coda-performance-dashboard/issues/6)
- Milestone 3 (MVP) : until Sep 4
  - Adaptions based on testing
  - Documentation (Final)
  - Code refactoring and integration for open source


## Features
- Block Producer Performance Dashboard
- Snarker Performance Dashboard

<img width="1495" alt="Dashboard" src="https://user-images.githubusercontent.com/897510/88521994-b7454800-d030-11ea-9307-f7d0010f9727.png">

## Install Guide

The Coda Performance Dashboard displays information of nodes through Prometheus and Grafana.

### Prerequisites

- Coda Protocol Block Producer and Snarker node(s)
  - Reference : https://codaprotocol.com/docs/getting-started


### Install Docker
> Prometheus and Grafana provides service through Docker.

```
apt install docker.io
```

### Configure Prometheus
> Save Prometheum configuration file and run it with Docker.

Create work folder and save the Prometheum configuration file.
```
mkdir /var/prometheus-coda
cd /var/prometheus-coda
vi /var/prometheus-coda/prometheus-coda.yml
```

Paste the contents of the file below. At this time, change <code>NODE_IP_ADDRESS</code> to your own IP address.
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
      - targets: ['NODE_IP_ADDRESS:6060'] ## Replace NODE_IP_ADDRESS to IP address of your server
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
<img width="982" alt="prometheus" src="https://user-images.githubusercontent.com/897510/88522011-bf9d8300-d030-11ea-8004-d9cef424469f.png">


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
- You can login to Grafana with username `admin` and default password `admin`. After you connect to Grafana for the first time, you will see a screen to set new password for your account.

Add data source
- <code>Configuration > Data Sources > Add data source > Prometheus</code>
- Type it below
  - Name : <code>Prometheus-Coda</code>
  - HTTP > URL : <code>http://CODA_NODE_IP_ADDRESS:19090</code>
- Click <code>[Save & Test]</code>

Add a dashboard from the template
- Go to <code>Create > Import JSON</code>
- Upload this file : https://github.com/dsrvlabs/coda-performance-dashboard/blob/master/grafana-json-model.json

Now, you can see the dashboard
- <code>Dashboard > Home > Coda:BlockProducer Performance Dashboard</code>

