# Blocky Role

This role deploys Blocky, a DNS server with ad-blocking capabilities for your homelab.

## Features

- Fast DNS resolution with caching
- Ad and tracker blocking using public blocklists
- Prometheus metrics endpoint
- Customizable upstream DNS servers
- REST API for management

## Requirements

- Kubernetes cluster with kubectl access
- kubernetes.core Ansible collection

## Role Variables

See `defaults/main.yaml` for all available variables.

Key variables:
- `blocky_namespace`: Namespace for deployment (default: homelab)
- `blocky_upstream_dns`: List of upstream DNS servers
- `blocky_service_type`: Kubernetes service type (default: LoadBalancer)
- `blocky_blocklists`: List of blocklist URLs

## Example Playbook

```yaml
- hosts: k3s_servers[0]
  vars:
    kubeconfig_path: /path/to/kubeconfig
  roles:
    - blocky
```

## Tags

- dns
- blocky

## Access

After deployment:
```bash
kubectl port-forward -n homelab svc/blocky 4000:4000
# Open http://localhost:4000

# Get LoadBalancer IP for DNS
kubectl get svc -n homelab blocky
```

## Usage

Configure your devices or DHCP server to use the Blocky LoadBalancer IP as DNS server.
