<a href="https://www.hyphen.earth/"><img src="https://github.com/Hyphen-Global-AG/hyphen-climate-framework/blob/main/.github/Hyphen_logo_final_hor.png?raw=true" alt="Hyphen Global AG" width="400"/></a>

# Hyphen-Chainlink-Grafana-Widgets

Widgets for Monitoring Chainlink Nodes with Prometheus / Grafana

## 1. Introduction
At the moment, Chainlink node does not expose information about it's OCR P2P nodes over Prometheus metrics.
This information is available in Chainlink Database, table `p2p_peers`.
There is [SQL Exporter](https://github.com/free/sql_exporter) available for Prometheus which can be used to get this data from the Database.
Grafana dashboard is available to visualize an information about P2P peers.

## 2. SQL Exporter in Kubernetes
SQL Exporter requires YAML configuration file, which can be deployed as a Kubernetes secret.
Here is the configuration for 2 metrics:
1. `chainlink_p2p_peers_total` - Total number of OCR P2P Peers
2. `chainlink_p2p_peer_address` - IP address information for each P2P peer
```
global:
  scrape_timeout: 10s
  scrape_timeout_offset: 500ms
  min_interval: 0s
  max_connections: 1
  max_idle_connections: 1
target:
  data_source_name: 'postgres://<username>:<password>@<server>:<port>/<database>'
  collectors:
    - chainlink
collectors:
  - collector_name: chainlink
    metrics:
      - metric_name: chainlink_p2p_peers_total
        type: gauge
        help: 'Total number of OCR P2P Peers.'
        values: [count]
        query: |
          SELECT count(distinct id)
          FROM p2p_peers
          WHERE id != peer_id
      - metric_name: chainlink_p2p_peer_address
        type: counter
        help: 'OCR P2P Peer IP address'
        key_labels:
          - peer
          - addr
        values: [count]
        query: |
          SELECT id as peer, addr, 1 as count
          FROM p2p_peers
          WHERE id != peer_id
```

Kubernetes `Deployment` and `Service` objects are used to deploy the Exporter itself and expose it's port for Prometheus scraping.
Kubernetes CRD `ServiceMonitor` is used for Prometheus configuration.
Please refer [k8_manifests](k8_manifests) folder for the working manifests.
Edit all variables marked by `<>` in both files and apply changes:

```
cd k8_manifests
kubectl apply -f .
```
After couple minutes custom metrics should be available in Prometheus.

## 3. Grafana dashboard
Import JSON file from [grafana_dashboards](grafana_dashboards) directory.
Please make sure Prometheus is a default data source in Grafana or edit `DataSource` variable in the imported dashboard.
