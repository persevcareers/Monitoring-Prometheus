# Prometheus Monitoring Overview

## Metric Collection
- **Pull Model**: Prometheus retrieves metrics over HTTP using a pull model.
- **Pushgateway**: Option to push metrics to Prometheus, useful for short-lived Kubernetes jobs & Cronjobs.
  
## Metric Endpoint
- Systems expose metrics on an `/metrics` endpoint for Prometheus to pull.
  
## PromQL
- Flexible query language for querying metrics in the Prometheus dashboard and visualizing them in Prometheus UI and Grafana.
  
## Prometheus Exporters
- Libraries that convert third-party app metrics to Prometheus format.
- Example: Prometheus node exporter for exposing Linux system-level metrics.
  
## TSDB (Time-Series Database)
- Prometheus utilizes TSDB for efficient data storage.
- By default, data is stored locally with options for integrating remote storage to avoid single points of failure.

