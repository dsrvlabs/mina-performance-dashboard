# Mina Performance Dashboard

Mina Performance Dashboard is a performance monitoring tool for Mina Protocol. It provides two functions, collecting and visualizing Block Producers' and Snarkers' performance data.


## 1. Features
- Block Producer Performance Dashboard
- Snarker Performance Dashboard

### 1-1. Screenshot
![screenshot_dark_1](https://user-images.githubusercontent.com/897510/92995078-f1b06900-f53a-11ea-962d-8b78c9c6de88.png)
![screenshot_dark_2](https://user-images.githubusercontent.com/897510/92995075-efe6a580-f53a-11ea-896d-13e91c0ee355.png)
![screenshot_light_1](https://user-images.githubusercontent.com/897510/92995077-f117d280-f53a-11ea-99d6-a37754804d5f.png)
![screenshot_light_2](https://user-images.githubusercontent.com/897510/92995071-ec531e80-f53a-11ea-979f-9cedb2146f39.png)

## 2. Install Guide

The Mina Performance Dashboard displays information of nodes through Prometheus and Grafana.
Also, server information is collected using node_exporter.

You have to prepare 2 servers. Of course, you can install everything on one server. But it is not recommended.

- Mina Node
  - Install Mina node : https://minaprotocol.com/docs/getting-started
  - Install node_exporter : https://github.com/prometheus/node_exporter#using-docker
- Prometheum and Grafana Server
  - Install Prometheus : <code>2.2. Install Prometheus</code>
  - Install Grafana : <code>2.3. Install Grafana</code>

### 2.1. Preparations

#### 2.1.1. Install Docker
> Prometheus and Grafana provides service through Docker.
```
apt install docker.io
```

### 2.1.2. Firewall Setting
- CODA node
  - Open inbound <code>6060</code>(Mina Metric) port from PROMETHEUM Server
  - Open inbound <code>9100</code>(node_exporter) port from PROMETHEUM Server
- Prometheum and Grafana Server
  - Open inbound <code>3000</code>(Grafana) and <code>9090</code>(Prometheum) port


### 2.2. Install Prometheus

#### 2.2.2. Configure Prometheus
Create the Prometheum configuration file.
```
vi /tmp/prometheus-mina.yml
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
  - job_name: 'mina-testnet'
    metrics_path: /metrics
    scheme: http
    static_configs:
      - targets: ['NODE_IP_ADDRESS:6060','NODE_IP_ADDRESS:9100'] ## Replace NODE_IP_ADDRESS to IP address of your server
        labels:
          hostname: 'mina-daemon'
```

> You can also get this file [prometheus-mina.yml](https://github.com/dsrvlabs/mina-performance-dashboard/blob/master/prometheus-mina.yml) here.

#### 2.2.3. Run Prometheus through Docker

```
docker run \
    -d --name=prometheus-mina \
    -p 9090:9090 \
    -v /tmp/prometheus-mina.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```

Check Prometheus : <code>http://PROMETHEUS_IP_ADDRESS:9090</code>
<img width="982" alt="prometheus" src="https://user-images.githubusercontent.com/897510/88522011-bf9d8300-d030-11ea-8004-d9cef424469f.png">

## 2.3. Install Grafana

### 2.3.1 Install and run Grafana

Run Grafana through Docker

```
docker run \
-d \
-p 3000:3000 \
--name=grafana \
-e "GF_INSTALL_PLUGINS=fifemon-graphql-datasource" \
grafana/grafana
```

> Add plugin(fifemon-graphql-datasource) : <code>-e "GF_INSTALL_PLUGINS=fifemon-graphql-datasource"</code>  
it makes GraphQL data available to Grafana.


### 2.3.1 Configure Grafana

Login to Grafana
- Visit : <code>http://GRAFANA_IP_ADDRESS:3000</code>

> You can login to Grafana with username `admin` and default password `admin`. After you connect to Grafana for the first time, you will see a screen to set new password for your account.

Add data source
- <code>Configuration > Data Sources > Add data source > Prometheus</code>
- Type it below
  - Name : <code>Prometheus-Mina</code>
  - HTTP > URL : <code>http://PROMETHEUS_IP_ADDRESS:9090</code>
- Click <code>[Save & Test]</code>

![screenshot_light_3](https://user-images.githubusercontent.com/897510/92995787-92a22280-f541-11ea-8210-9eaddf2510d3.png)

Add a dashboard from template
- Move to <code>Create > Import JSON</code>
- Import via grafana.com : 12840
  > Grafana Dashboard URL : https://grafana.com/grafana/dashboards/12840
  - Otherwise Upload a json file as below.
    > Upload this file : https://github.com/dsrvlabs/mina-performance-dashboard/blob/master/grafana-json-model.json
- Click : `Load` button

![screenshot_light_4](https://user-images.githubusercontent.com/897510/92995828-da28ae80-f541-11ea-8a2f-0d8c60343f13.png)

Now, you can see the dashboard
- <code>Dashboard > Home > Mina:BlockProducer Performance Dashboard</code>

## 3. Reference
- Mina Protocol : https://minaprotocol.com/docs/getting-started
- Prometheus : https://prometheus.io/docs/prometheus/latest/getting_started/
- Prometheus node_exporter : https://github.com/prometheus/node_exporter
- Grafana : https://grafana.com/docs/grafana/latest/
- Grafana Dashbaord : [1 Node Exporter for Prometheus Dashboard EN](https://grafana.com/grafana/dashboards/11074)
- Grafana Dahsboard : [Mina: Block Producer Performance Dashboard](https://grafana.com/grafana/dashboards/12840)
