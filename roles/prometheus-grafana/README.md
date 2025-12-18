# Prometheus & Grafana Role

This role deploys the kube-prometheus-stack Helm chart, which includes:
- Prometheus for metrics collection and storage
- Grafana for visualization with pre-configured dashboards
- AlertManager for alert routing
- Node Exporter for node-level metrics
- Kube State Metrics for Kubernetes resource metrics

## Requirements

- Kubernetes cluster with kubectl access
- Helm 3+
- kubernetes.core Ansible collection

## Role Variables

See `defaults/main.yaml` for all available variables.

Key variables:
- `prometheus_namespace`: Namespace for deployment (default: homelab)
- `prometheus_storage_size`: Storage size for Prometheus (default: 10Gi)
- `grafana_admin_password`: Admin password for Grafana (use Ansible Vault!)
- `prometheus_retention`: Data retention period (default: 30d)

## Example Playbook

```yaml
- hosts: k3s_servers[0]
  vars:
    kubeconfig_path: /path/to/kubeconfig
  roles:
    - prometheus-grafana
```

## Tags

- monitoring
- prometheus
- grafana

## Access

After deployment:
```bash
kubectl port-forward -n homelab svc/prometheus-grafana 3000:80
# Open http://localhost:3000
```
