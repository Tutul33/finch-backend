
# Monitoring_Setup.md

## Overview

This document describes the setup of a monitoring and logging stack using **Prometheus**, **Grafana**, and **Loki** in a Kubernetes cluster. It includes:
- Deployment instructions for Prometheus, Loki, Grafana
- Configuration for metrics and log collection
- Access and interpretation of dashboards
- Alert configuration and management

---

## 1. Components and Architecture

```
+------------------+         +------------------+        +------------------+
|  Applications    |  --->   |  Prometheus      |  --->  |  Grafana         |
|  (Frontend, etc) | metrics|  (Metrics DB)     | data   |  (Dashboards)    |
+------------------+         +------------------+        +------------------+
        |                              ^
        | logs                         |
        v                              |
+------------------+         +------------------+
|  Loki + Promtail |  ---->  |  Loki (Log Store) |
|  (Log Collector) | logs   |                  |
+------------------+         +------------------+
```

---

## 2. Installing Monitoring Components

### 2.1. **Prometheus & Grafana**
Deploy using the **kube-prometheus-stack** Helm chart (by the Prometheus community):

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```

This installs:
- Prometheus
- Alertmanager
- Grafana
- Node-exporter
- kube-state-metrics

### 2.2. **Loki + Promtail**
Deploy **Loki** (log aggregation) and **Promtail** (log collection) using Helm:

```sh
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install Loki
helm install loki grafana/loki-stack \
  --namespace capstone --create-namespace \
  --set promtail.enabled=false

# Deploy Promtail using your DaemonSet manifest
kubectl apply -f promtail-daemonset.yaml
```

Promtail reads logs from `/var/log/pods/...` and sends them to Loki.

---

## 3. Metrics Collection

### 3.1. **Application Metrics**
Applications expose metrics on `/metrics`. Exporters used:
- `redis-exporter`
- `postgres-exporter`
- `http_requests_total`, `http_request_duration_seconds`, etc. from custom apps

Metrics are collected via `ServiceMonitor` resources, e.g.:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: redis-exporter
  namespace: capstone
spec:
  selector:
    matchLabels:
      app: redis-exporter
  namespaceSelector:
    matchNames: [capstone]
  endpoints:
    - port: metrics
      interval: 15s
```

### 3.2. **System Metrics**
Collected by:
- **Node Exporter** (CPU, memory, disk)
- **kube-state-metrics** (pod status, deployments, etc.)

No extra configuration is needed if using `kube-prometheus-stack`.

---

## 4. Logs Collection with Promtail

Promtail collects logs from host paths:
- `/var/log/containers`
- `/var/log/pods`

Configured in `promtail-config` ConfigMap with `kubernetes_sd_configs`. Example relabeling:

```yaml
scrape_configs:
  - job_name: kubernetes-pods
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_label_name]
        target_label: job
      - source_labels: [__meta_kubernetes_namespace]
        target_label: namespace
```

Logs are pushed to Loki at:
```
http://loki.capstone.svc.cluster.local:3100/loki/api/v1/push
```

---

## 5. Accessing Dashboards

### Grafana UI
Access Grafana using:
```sh
kubectl port-forward svc/prometheus-stack-grafana -n monitoring 3000:80
```
Then open: http://localhost:3000

Default login:
- **Username:** `admin`
- **Password:** Retrieved via:
  ```sh
  kubectl get secret --namespace monitoring prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
  ```

### Dashboards

| Dashboard | Purpose |
|----------|---------|
| **Application Dashboard** | HTTP request and error rates |
| **Kubernetes Cluster Dashboard** | Node and pod resource usage |
| **Application Logs** | View logs from frontend/backend via Loki |
| **Redis & PostgreSQL Metrics** | DB metrics like memory, connections |

Import the dashboards in Grafana > Dashboards > Import. Paste the JSON definitions.

---

## 6. Interpreting Dashboards

### Key Metrics

- **Frontend Request Rate**: Traffic volume to frontend
- **Backend Error Rate**: 5xx errors; indicates issues in APIs
- **CPU/Memory Usage**: Watch for throttling or OOMKills
- **Redis Memory Usage**: Ensure Redis is not memory bound
- **PostgreSQL Connections**: Track max connections usage

### Logs Panel

Use filters like:
```logql
{app="finch-backend", level="error"}
```
to filter errors. Logs are timestamped and searchable by pod, container, namespace.

---

## 7. Alerting Setup

### Alert Rules

Alert rules are defined in `PrometheusRule` CRDs. Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: capstone-alert-rules
  namespace: capstone
spec:
  groups:
    - name: pod-health.rules
      rules:
        - alert: PodCrashLooping
          expr: rate(kube_pod_container_status_restarts_total[5m]) > 0.1
          for: 2m
          labels:
            severity: warning
          annotations:
            summary: "Pod is restarting too frequently"
```

Apply rules:
```sh
kubectl apply -f prometheus-rules.yaml
```

### Example Alerts

| Alert | Condition |
|-------|-----------|
| **PodCrashLooping** | Pod restarts > 0.1 times per second |
| **PodNotReady** | Pod readiness condition is false |
| **RedisDown** | Redis exporter is unreachable |
| **HighBackendErrorRate** | HTTP 5xx rate > 1 req/sec |
| **PostgresDown** | PostgreSQL exporter is down |

### Alertmanager

- Alerts from Prometheus are sent to Alertmanager
- You can configure email, Slack, or other integrations via Alertmanager config

---

## 8. Maintenance & Tips

- Use `kubectl top` for quick CPU/mem snapshots
- Regularly prune old data in Loki (retention config)
- Secure Grafana with OAuth or SSO in production
- Use recording rules for expensive queries

---

## Appendix

### Ports Used

| Component | Port |
|----------|------|
| Prometheus | 9090 |
| Grafana | 3000 |
| Loki | 3100 |
| Redis Exporter | 9121 |
| PostgreSQL Exporter | 9187 |

---

## Conclusion

You now have a full observability stack with Prometheus for metrics, Loki for logs, and Grafana for visualization. This setup enables proactive monitoring, troubleshooting, and performance analysis for your Kubernetes-based applications.
