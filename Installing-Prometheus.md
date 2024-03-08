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

# Installation ,

## The Kubernetes Prometheus monitoring stack has the following components.

- **Prometheus Server**
- **Alert Manager**
- **Grafana**

## Prometheus Kubernetes Manifests:
- https://github.com/persevcareers/kubernetes-prometheus


# Create a Namespace & ClusterRole

First, we will create a Kubernetes namespace for all our monitoring components. If you don’t create a dedicated namespace, all the Prometheus Kubernetes deployment objects get deployed on the default namespace.

Execute the following command to create a new namespace named `monitoring`:

```bash
kubectl create namespace monitoring
```
## Prometheus uses Kubernetes APIs to read all the available metrics from Nodes, Pods, Deployments, etc. For this reason, we need to create an RBAC policy with read access to required API groups and bind the policy to the monitoring namespace.

In the role, given below, you can see that we have added get, list, and watch permissions to nodes, services endpoints, pods, and ingresses. The role binding is bound to the monitoring namespace. If you have any use case to retrieve metrics from any other object, you need to add that in this cluster role.
Create the role using the following command:

```bash

kubectl create -f clusterRole.yaml
```
## Create a Config Map To Externalize Prometheus Configurations
## All configurations for Prometheus are part of prometheus.yaml file and all the alert rules for Alertmanager are configured in prometheus.rules.

By externalizing Prometheus configs to a Kubernetes config map, you don’t have to build the Prometheus image whenever you need to add or remove a configuration. You need to update the config map and restart the Prometheus pods to apply the new configuration.

- **Step 1:**
Create a file called config-map.yaml and copy the file contents from this link.

- **Step 2:**
Execute the following command to create the config map in Kubernetes:

```bash
kubectl create -f config-map.yaml
```
It creates two files inside the container.

The prometheus.yaml contains all the configurations to discover pods and services running in the Kubernetes cluster dynamically. We have the following scrape jobs in our Prometheus scrape configuration:

## kubernetes-apiservers: It gets all the metrics from the API servers.
## kubernetes-nodes: It collects all the Kubernetes node metrics.
## kubernetes-pods: All the pod metrics get discovered if the pod metadata is annotated with prometheus.io/scrape and prometheus.io/port annotations.
## kubernetes-cadvisor: Collects all cAdvisor metrics.
## kubernetes-service-endpoints: All the Service endpoints are scrapped if the service metadata is annotated with prometheus.io/scrape and prometheus.io/port annotations. It can be used for black-box monitoring.
## The prometheus.rules contains all the alert rules for sending alerts to the Alertmanager.

##Create a Prometheus Deployment
- **Step 1:**
Create a file named prometheus-deployment.yaml and copy the following contents onto the file. In this configuration, we are mounting the Prometheus config map as a file inside /etc/prometheus as explained in the previous section.

This deployment uses the latest official Prometheus image from the Docker Hub. Also, we are not using any persistent storage volumes for Prometheus storage as it is a basic setup. When setting up Prometheus for production use cases, make sure you add persistent storage to the deployment.

- **Step 2:**
Create a deployment on monitoring namespace using the above file:

```bash
kubectl create -f prometheus-deployment.yaml --namespace=monitoring
```
- **Step 3:**
You can check the created deployment using the following command:

```bash
kubectl get deployments --namespace=monitoring
```

# Connecting To Prometheus Dashboard

## Exposing Prometheus as a Service [NodePort & LoadBalancer]

To access the Prometheus dashboard over an IP or a DNS name, you need to expose it as a Kubernetes service.

1. Create a file named `prometheus-service.yaml` and copy the following contents. This will expose Prometheus on all Kubernetes node IPs on port 30000.

2. Create the service using the following command:

```bash
kubectl create -f prometheus-service.yaml --namespace=monitoring
