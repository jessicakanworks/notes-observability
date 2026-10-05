# notes-observability

Monitoring for [notes-api](../notes-api) with **Prometheus, Alertmanager and Grafana**, deployable two ways: a Docker Compose stack with node-exporter, cAdvisor and blackbox probes, or `kube-prometheus-stack` on Kubernetes with a ServiceMonitor and PrometheusRule. Alerting is wired for the thing that matters: the API health endpoint failing.

1. **Docker Compose** (`compose/`) — Prometheus, Alertmanager, Grafana, node-exporter, cAdvisor, blackbox-exporter.
2. **Kubernetes** (`k8s/`) — `kube-prometheus-stack` via Helm with a values file, a ServiceMonitor for the API and a PrometheusRule.

One alert rule I actually want paged on: **the API health endpoint failing for 2 minutes**.

```bash
cd compose && cp .env.example .env && docker compose up -d
# Grafana http://localhost:3000 (admin / .env password) → dashboards auto-provisioned
# Prometheus http://localhost:9090/alerts
```
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kps prometheus-community/kube-prometheus-stack -n monitoring --create-namespace -f k8s/values.yaml
kubectl apply -f k8s/servicemonitor-api.yaml -f k8s/prometheusrule-notes.yaml
```

## Dashboards
- Node (CPU, memory, disk, network) — imported Grafana ID 1860
- Containers (cAdvisor) — Grafana ID 14282
- Notes API — request rate, error rate, p95 latency, uptime probe (`grafana/dashboards/notes-api.json`)
