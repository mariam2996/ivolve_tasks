<img width="1366" height="768" alt="540b9e9b-2069-40c6-b97e-756a6c03178e" src="https://github.com/user-attachments/assets/8fd0d286-3f99-4cd5-b30a-d8d9edc542ac" />
<img width="1366" height="768" alt="2ace6c34-c608-41c3-9b6c-76133101ab80" src="https://github.com/user-attachments/assets/1ef677f1-32f6-4c2e-8bcb-3272322553a7" />
# Lab 18 - Node-Wide Pod Management with DaemonSet

## Objective

Deploy a monitoring stack in Kubernetes and use a DaemonSet to run Prometheus node-exporter on every node.

The monitoring stack includes:

- Prometheus
- Alertmanager
- Grafana
- kube-state-metrics
- Prometheus node-exporter

## Monitoring Namespace

A dedicated namespace was used:

```bash
kubectl create namespace monitoring
```

## Prometheus Installation

Prometheus was installed in the `monitoring` namespace using Helm.

```bash
helm install prometheus \
  oci://ghcr.io/prometheus-community/charts/prometheus \
  -n monitoring
```

The Prometheus installation also deployed:

- Alertmanager
- kube-state-metrics
- node-exporter
- pushgateway

## Verification

Check monitoring pods:

```bash
kubectl get pods -n monitoring
```

Check the node-exporter DaemonSet:

```bash
kubectl get daemonset -n monitoring
```

Verify that a node-exporter pod is running on each Kubernetes node:

```bash
kubectl get pods -n monitoring -o wide | grep node-exporter
```

The DaemonSet showed:

```text
DESIRED: 2
CURRENT: 2
READY:   2
```

This confirmed that node-exporter was running on both cluster nodes.

## Metrics Exposure

Node addresses were checked using:

```bash
kubectl get nodes -o wide
```

The node-exporter metrics endpoint was validated on port `9100`:

```bash
curl http://172.30.1.2:9100/metrics | head
curl http://172.30.2.2:9100/metrics | head
```

Both nodes successfully exposed Prometheus metrics.

## Prometheus Verification

Prometheus readiness was verified using:

```bash
kubectl exec -n monitoring deploy/prometheus-server -- \
wget -qO- http://localhost:9090/-/ready
```

The server returned:

```text
Prometheus Server is Ready.
```

## Grafana Installation

Grafana was installed in the same monitoring namespace using Helm.

```bash
helm repo add grafana-community https://grafana-community.github.io/helm-charts
helm repo update
helm install grafana grafana-community/grafana -n monitoring
```

The Grafana pod was verified using:

```bash
kubectl get pods -n monitoring
```

## Grafana Access

The Grafana admin password was retrieved using:

```bash
kubectl get secret grafana -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 --decode
```

Grafana was exposed temporarily using:

```bash
kubectl port-forward --address 0.0.0.0 \
  -n monitoring svc/grafana 3000:80
```

## Prometheus Data Source

Prometheus was configured as the Grafana data source using:

```text
http://prometheus-server.monitoring.svc.cluster.local
```

## Grafana Dashboard

The `Node Exporter Full` dashboard was imported into Grafana.

The dashboard successfully displayed real-time node metrics including:

- CPU usage
- Memory usage
- Disk usage
- Network metrics
- System uptime

## Result

The monitoring stack was successfully deployed.

Prometheus collected metrics from node-exporter running as a DaemonSet on every Kubernetes node, and Grafana successfully visualized the collected node metrics.
