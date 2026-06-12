# 📊 AWS Monitoring Stack

> Complete observability stack deployed to EKS using kube-prometheus-stack Helm chart. Custom Grafana dashboards for AWS EC2, Kubernetes cluster health, and application SLOs. AlertManager routes critical alerts to Slack. CloudWatch Metrics Exporter bridges AWS-native metrics into Prometheus.

[![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?style=flat-square&logo=prometheus)](https://prometheus.io)
[![Grafana](https://img.shields.io/badge/Grafana-Dashboards-F46800?style=flat-square&logo=grafana)](https://grafana.com)
[![Helm](https://img.shields.io/badge/Helm-kube--prometheus--stack-0F1689?style=flat-square&logo=helm)](https://helm.sh)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## 🏗️ Architecture

```
AWS Services (EC2, RDS, Lambda, ALB)
        │  CloudWatch Metrics
        ▼
┌──────────────────────────┐
│  CloudWatch Exporter     │  ← Bridges AWS metrics into Prometheus
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐     ┌─────────────────────────────┐
│  Prometheus              │─────▶│  Grafana                    │
│  (time-series DB)        │     │  - EC2 Dashboard             │
│  - Scrapes all pods      │     │  - Kubernetes Dashboard      │
│  - 30-day retention      │     │  - Application SLO Dashboard │
└──────────┬───────────────┘     └─────────────────────────────┘
           │
           ▼
┌──────────────────────────┐     ┌─────────────────────────────┐
│  AlertManager            │─────▶│  Slack Channel              │
│  - Routing rules         │     │  #alerts-critical            │
│  - Silence management    │     │  #alerts-warning             │
│  - PagerDuty integration │     └─────────────────────────────┘
└──────────────────────────┘
```

## 📁 Repository Structure

```
aws-monitoring-stack/
├── helm/
│   ├── prometheus-values.yaml    # kube-prometheus-stack config
│   ├── alertmanager-config.yaml  # Routing rules + Slack webhook
│   └── cloudwatch-exporter.yaml  # AWS metrics bridge config
├── dashboards/
│   ├── ec2-overview.json         # EC2 CPU, memory, disk, network
│   ├── kubernetes-cluster.json   # Node health, pod status, PVC
│   ├── application-slo.json      # Latency p50/p95/p99, error rate
│   └── cost-overview.json        # AWS cost metrics via CW Exporter
├── alerts/
│   ├── infrastructure.yaml       # Node down, disk full, OOM rules
│   ├── application.yaml          # High error rate, latency breach
│   └── slo.yaml                  # Error budget burn rate alerts
├── scripts/
│   └── slo_calculator.py         # SLI/SLO/error budget calculator
└── README.md
```

## 🚀 Quick Start

```bash
# 1. Add Helm repos
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# 2. Install kube-prometheus-stack
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace \
  -f helm/prometheus-values.yaml

# 3. Install CloudWatch Exporter
helm install cloudwatch-exporter prometheus-community/prometheus-cloudwatch-exporter \
  --namespace monitoring \
  -f helm/cloudwatch-exporter.yaml

# 4. Apply AlertManager config
kubectl apply -f helm/alertmanager-config.yaml -n monitoring

# 5. Import Grafana dashboards
# Access Grafana: kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring
# Import JSON files from dashboards/ directory
```

## 📈 Dashboards Included

| Dashboard | Key Metrics |
|-----------|-------------|
| **EC2 Overview** | CPU, memory, disk I/O, network throughput, instance count |
| **Kubernetes Cluster** | Node health, pod restarts, resource requests vs limits, PVC usage |
| **Application SLO** | Request rate, error rate, p50/p95/p99 latency, error budget |
| **AWS Cost** | Daily spend by service, trending, budget alerts |

## 🔔 Alert Rules

```yaml
# Example: High Error Rate Alert
- alert: HighErrorRate
  expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "Error rate above 5% for {{ $labels.service }}"
```

## 🛠️ Tech Stack

`Prometheus` `Grafana` `AlertManager` `kube-prometheus-stack` `CloudWatch Exporter` `Helm` `AWS EKS` `Slack` `Python`

---

*Part of [Sreekar KV's cloud portfolio](https://sreekar424.github.io)*
