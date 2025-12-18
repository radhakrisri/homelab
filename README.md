# Homelab K3s Cluster

Ansible automation for deploying a highly available K3s Kubernetes cluster.

## Features

- **High Availability**: Deploys K3s with embedded etcd for HA (3+ server nodes)
- **Multi-OS Support**: Works with Ubuntu, AlmaLinux, Raspbian/Debian
- **Multi-Architecture**: Supports both ARM64 (Raspberry Pi) and x86_64 systems
- **Best Practices**: Follows Ansible and K3s best practices

## Prerequisites

- Ansible 2.15+ installed on your control machine
- SSH access to all target nodes
- Target nodes running Ubuntu 20.04+, AlmaLinux 8+, or Raspbian/Debian 11+
- Python 3.8+ on target nodes

## Quick Start

### 1. Install Ansible Collections

```bash
ansible-galaxy install -r requirements.yaml
```

### 2. Prepare Your Inventory

```bash
cp inventories/sample_inventory.yaml inventories/production.yaml
```

Edit `inventories/production.yaml` with your actual:
- Hostnames and IP addresses
- SSH credentials
- K3s token (generate with `openssl rand -hex 32`)

### 3. (Optional) Set Up Ansible Vault for Secrets

```bash
# Create vault password file
echo 'your-vault-password' > .vault_pass
chmod 600 .vault_pass

# Encrypt sensitive variables
ansible-vault encrypt_string 'your-k3s-token' --name 'k3s_token'
```

### 4. Deploy the Cluster

```bash
# Full deployment
ansible-playbook -i inventories/production.yaml playbooks/site.yaml

# Or step-by-step
ansible-playbook -i inventories/production.yaml playbooks/site.yaml --tags common
ansible-playbook -i inventories/production.yaml playbooks/site.yaml --tags k3s-servers
ansible-playbook -i inventories/production.yaml playbooks/site.yaml --tags k3s-agents
```

### 5. Access Your Cluster

After deployment, the kubeconfig is saved to `.agent-workarea/kubeconfig`:

```bash
export KUBECONFIG=$(pwd)/.agent-workarea/kubeconfig
kubectl get nodes
```

## Directory Structure

```
.
├── ansible.cfg              # Ansible configuration
├── requirements.yaml        # Ansible Galaxy dependencies
├── inventories/
│   └── sample_inventory.yaml  # Sample inventory (copy and customize)
├── group_vars/
│   ├── all.yaml             # Global variables
│   ├── k3s_servers.yaml     # Server-specific variables
│   └── k3s_agents.yaml      # Agent-specific variables
├── playbooks/
│   ├── site.yaml            # Main deployment playbook
│   ├── reset.yaml           # Cluster reset/uninstall
│   └── upgrade.yaml         # Cluster upgrade
└── roles/
    ├── common/              # OS preparation role
    ├── k3s_server/          # K3s server installation
    └── k3s_agent/           # K3s agent installation
```

## Playbooks

| Playbook | Description |
|----------|-------------|
| `site.yaml` | Full cluster deployment |
| `reset.yaml` | Complete cluster teardown |
| `upgrade.yaml` | Rolling upgrade to new K3s version |
| `deploy-services.yaml` | Deploy all homelab services |
| `deploy-prometheus-grafana.yaml` | Deploy monitoring stack |
| `deploy-blocky.yaml` | Deploy DNS and ad blocker |
| `deploy-vault.yaml` | Deploy secret management |
| `deploy-authentik.yaml` | Deploy SSO/SAML |
| `deploy-traefik-dashboard.yaml` | Enable Traefik dashboard |
| `uninstall-services.yaml` | Uninstall all services |

## Upgrade

To upgrade the cluster to a new K3s version:

```bash
ansible-playbook -i inventories/production.yaml playbooks/upgrade.yaml \
  -e k3s_version=v1.31.2+k3s1
```

## Reset/Uninstall

To completely remove K3s from all nodes:

```bash
ansible-playbook -i inventories/production.yaml playbooks/reset.yaml
```

## Homelab Services

After deploying the K3s cluster, you can deploy additional homelab services:

### Available Services

1. **Prometheus & Grafana** - Complete monitoring stack with dashboards
2. **Blocky** - DNS server with ad-blocking capabilities
3. **Vault** - Secret management with web UI
4. **Authentik** - SSO/SAML/OAuth2 authentication
5. **Traefik Dashboard** - Ingress controller UI

### Deploy All Services

```bash
# Deploy all services at once
ansible-playbook -i inventories/production.yaml playbooks/deploy-services.yaml

# Or include services during cluster deployment
ansible-playbook -i inventories/production.yaml playbooks/site.yaml -e deploy_services=true
```

### Deploy Individual Services

```bash
# Deploy only Prometheus & Grafana
ansible-playbook -i inventories/production.yaml playbooks/deploy-prometheus-grafana.yaml

# Deploy only Blocky DNS
ansible-playbook -i inventories/production.yaml playbooks/deploy-blocky.yaml

# Deploy only Vault
ansible-playbook -i inventories/production.yaml playbooks/deploy-vault.yaml

# Deploy only Authentik
ansible-playbook -i inventories/production.yaml playbooks/deploy-authentik.yaml

# Enable Traefik Dashboard
ansible-playbook -i inventories/production.yaml playbooks/deploy-traefik-dashboard.yaml
```

### Deploy Specific Services with Tags

```bash
# Deploy only monitoring
ansible-playbook -i inventories/production.yaml playbooks/deploy-services.yaml --tags monitoring

# Deploy only Vault
ansible-playbook -i inventories/production.yaml playbooks/deploy-services.yaml --tags vault
```

### Uninstall Services

```bash
# Uninstall all services
ansible-playbook -i inventories/production.yaml playbooks/uninstall-services.yaml

# Uninstall specific service
ansible-playbook -i inventories/production.yaml playbooks/uninstall-services.yaml --tags prometheus
```

### Access Services

After deployment, services can be accessed via port-forwarding:

```bash
# Grafana (monitoring dashboards)
kubectl port-forward -n homelab svc/prometheus-grafana 3000:80
# Open http://localhost:3000 (default: admin/admin)

# Prometheus (metrics)
kubectl port-forward -n homelab svc/prometheus-kube-prometheus-prometheus 9090:9090
# Open http://localhost:9090

# Blocky (DNS API)
kubectl port-forward -n homelab svc/blocky 4000:4000
# Open http://localhost:4000

# Vault (secrets management)
kubectl port-forward -n homelab svc/vault 8200:8200
# Open http://localhost:8200

# Authentik (SSO)
kubectl port-forward -n homelab svc/authentik-server 80:80
# Open http://localhost

# Traefik Dashboard
kubectl port-forward -n kube-system svc/traefik-dashboard 9000:9000
# Open http://localhost:9000/dashboard/
```

### Service Configuration

Service configuration is managed through role defaults and can be overridden in your inventory:

**Configuration Structure:**
- `group_vars/k3s_services.yaml` - Common settings (namespace, Helm repos, service toggles)
- `roles/<role>/defaults/main.yaml` - Service-specific default configuration

**To customize services**, override variables in your inventory file:

```yaml
# In your inventory file (e.g., inventories/production.yaml)
all:
  vars:
    # Enable/disable services (in group_vars/k3s_services.yaml)
    prometheus_enabled: true
    blocky_enabled: true
    vault_enabled: true
    authentik_enabled: true
    traefik_dashboard_enabled: true
    
    # Override service-specific settings
    # Grafana configuration (see roles/prometheus-grafana/defaults/main.yaml)
    grafana_admin_password: "{{ vault_grafana_admin_password }}"
    grafana_storage_size: "5Gi"
    
    # Blocky DNS configuration (see roles/blocky/defaults/main.yaml)
    blocky_upstream_dns:
      - 1.1.1.1
      - 8.8.8.8
    
    # Vault configuration (see roles/vault/defaults/main.yaml)
    vault_dev_mode: false  # Set to true for development only
    vault_storage_size: "10Gi"
    
    # Authentik configuration (see roles/authentik/defaults/main.yaml)
    authentik_secret_key: "{{ vault_authentik_secret_key }}"
    authentik_postgresql_password: "{{ vault_authentik_postgresql_password }}"
```

For a complete list of configurable variables, see each role's `defaults/main.yaml` file.

For production deployments, store sensitive values in Ansible Vault:

```bash
# Encrypt sensitive variables
ansible-vault encrypt_string 'your-strong-password' --name 'vault_grafana_admin_password'
ansible-vault encrypt_string 'your-secret-key' --name 'vault_authentik_secret_key'
```

## Configuration Options

### K3s Settings (in inventory or group_vars)

| Variable | Default | Description |
|----------|---------|-------------|
| `k3s_version` | `v1.31.2+k3s1` | K3s version to install |
| `k3s_token` | - | Cluster token (required, use vault) |
| `k3s_cluster_cidr` | `10.42.0.0/16` | Pod network CIDR |
| `k3s_service_cidr` | `10.43.0.0/16` | Service network CIDR |
| `k3s_server_extra_args` | - | Additional server arguments |
| `k3s_agent_extra_args` | - | Additional agent arguments |

### Node Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `k3s_node_labels` | `[]` | Kubernetes labels for the node |
| `k3s_node_taints` | `[]` | Kubernetes taints for the node |

## Supported Operating Systems

| OS | Version | Architecture |
|----|---------|--------------|
| Ubuntu | 20.04, 22.04, 24.04 | amd64, arm64 |
| AlmaLinux | 8, 9 | amd64, arm64 |
| Debian | 11, 12 | amd64, arm64 |
| Raspbian | 11, 12 | arm64 |

## Troubleshooting

### Check K3s Service Status

```bash
# On server nodes
sudo systemctl status k3s

# On agent nodes
sudo systemctl status k3s-agent

# View logs
sudo journalctl -u k3s -f
```

### Verify Cluster Health

```bash
# From any server node
sudo k3s kubectl get nodes
sudo k3s kubectl get pods -A
```

### Common Issues

1. **Nodes not joining**: Ensure all firewalls allow required ports (6443, 2379-2380, 10250, 8472)
2. **etcd issues**: For HA, ensure odd number of server nodes (3 or 5)
3. **ARM64 issues**: Ensure cgroups are enabled in `/boot/firmware/cmdline.txt`

## License

MIT

## Contributing

Contributions welcome! Please follow the guidelines in `.github/agents/devops-agent.md`.
