# K3s Homelab Services

This document provides detailed information about the homelab services that can be deployed on your K3s cluster.

## Overview

The following services are available for deployment:

1. **Prometheus & Grafana** - Monitoring and metrics visualization
2. **Blocky** - DNS server with ad-blocking
3. **Vault** - Secret management
4. **Authentik** - SSO/SAML/OAuth2 provider
5. **Traefik Dashboard** - Ingress controller UI

## Architecture

All services are deployed in the `homelab` namespace (except Traefik which is in `kube-system`). Services use the `local-path` storage class by default, which is provided by K3s.

```
┌─────────────────────────────────────────────────┐
│              K3s Cluster                        │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │Prometheus│  │  Blocky  │  │  Vault   │     │
│  │ Grafana  │  │   DNS    │  │ Secrets  │     │
│  └──────────┘  └──────────┘  └──────────┘     │
│                                                 │
│  ┌──────────┐  ┌──────────┐                    │
│  │Authentik │  │ Traefik  │                    │
│  │   SSO    │  │Dashboard │                    │
│  └──────────┘  └──────────┘                    │
│                                                 │
└─────────────────────────────────────────────────┘
```

## Service Details

### 1. Prometheus & Grafana

**Purpose**: Complete monitoring stack for all cluster resources and services.

**Components**:
- Prometheus: Metrics collection and storage
- Grafana: Metrics visualization with dashboards
- AlertManager: Alert routing and management
- Node Exporter: Node-level metrics
- Kube State Metrics: Kubernetes resource metrics

**Default Configuration**:
```yaml
prometheus_namespace: homelab
prometheus_storage_size: 10Gi
prometheus_retention: 30d
grafana_admin_user: admin
grafana_admin_password: admin  # Change this!
grafana_storage_size: 5Gi
```

**Access**:
```bash
kubectl port-forward -n homelab svc/prometheus-grafana 3000:80
# Open http://localhost:3000
```

**Key Features**:
- Pre-configured dashboards for Kubernetes monitoring
- ServiceMonitor support for automatic service discovery
- Persistent storage for metrics and dashboards
- AlertManager for notification routing

**Customization**:
Edit `roles/prometheus-grafana/defaults/main.yaml` or override in inventory:
```yaml
prometheus_replicas: 2  # For HA
prometheus_ingress_enabled: true
prometheus_ingress_host: prometheus.yourdomain.com
grafana_ingress_enabled: true
grafana_ingress_host: grafana.yourdomain.com
```

### 2. Blocky

**Purpose**: DNS server with ad-blocking capabilities for your homelab.

**Features**:
- Fast DNS resolution with caching
- Ad and tracker blocking using public blocklists
- Prometheus metrics endpoint
- Customizable upstream DNS servers
- REST API for management

**Default Configuration**:
```yaml
blocky_namespace: homelab
blocky_upstream_dns:
  - 1.1.1.1
  - 1.0.0.1
blocky_service_type: LoadBalancer
blocky_service_port: 53
blocky_api_port: 4000
```

**Access**:
```bash
# API/Metrics
kubectl port-forward -n homelab svc/blocky 4000:4000
# Open http://localhost:4000

# Get LoadBalancer IP for DNS
kubectl get svc -n homelab blocky
```

**Usage**:
Configure your devices or DHCP server to use the Blocky LoadBalancer IP as DNS server.

**Customization**:
Edit blocklists in `roles/blocky/defaults/main.yaml`:
```yaml
blocky_blocklists:
  - https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
  - https://adaway.org/hosts.txt
  - https://your-custom-blocklist.com/hosts
```

### 3. Vault

**Purpose**: Centralized secret management for all homelab services.

**Features**:
- Secure secret storage
- Dynamic secrets generation
- Encryption as a service
- Kubernetes secrets injection
- Web UI for management
- Audit logging

**Default Configuration**:
```yaml
vault_namespace: homelab
vault_storage_size: 10Gi
vault_ui_enabled: true
vault_dev_mode: false  # Never use true in production!
vault_ha_enabled: false
```

**Access**:
```bash
kubectl port-forward -n homelab svc/vault 8200:8200
# Open http://localhost:8200
```

**Initial Setup** (Production Mode):
```bash
# Initialize Vault (only once)
kubectl exec -n homelab vault-0 -- vault operator init

# Unseal Vault (required after each restart, use 3 of 5 keys)
kubectl exec -n homelab vault-0 -- vault operator unseal <key-1>
kubectl exec -n homelab vault-0 -- vault operator unseal <key-2>
kubectl exec -n homelab vault-0 -- vault operator unseal <key-3>

# Login with root token
kubectl exec -n homelab vault-0 -- vault login <root-token>
```

**Development Mode** (NOT for production):
```yaml
vault_dev_mode: true
vault_dev_root_token: root
```

**Kubernetes Integration**:
```yaml
# Example: Inject secrets into pods
apiVersion: v1
kind: Pod
metadata:
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "myapp"
    vault.hashicorp.com/agent-inject-secret-config: "secret/myapp/config"
```

### 4. Authentik

**Purpose**: Single Sign-On (SSO) and identity provider for homelab services.

**Features**:
- SAML 2.0, OAuth2, and OpenID Connect support
- LDAP provider
- User and group management
- Multi-factor authentication (MFA)
- Flow-based authentication
- Web UI for administration

**Default Configuration**:
```yaml
authentik_namespace: homelab
authentik_secret_key: changeme  # MUST change this!
authentik_postgresql_enabled: true
authentik_redis_enabled: true
authentik_storage_size: 5Gi
```

**Access**:
```bash
kubectl port-forward -n homelab svc/authentik-server 80:80
# Open http://localhost
```

**Initial Setup**:
1. Navigate to http://localhost after port-forward
2. Follow the setup wizard
3. Create your first admin user
4. Configure authentication flows
5. Create applications and providers

**Security**:
⚠️ **IMPORTANT**: Change the secret key before deployment!
```bash
# Generate a secure secret key
openssl rand -hex 32

# Store in Ansible Vault
ansible-vault encrypt_string '<generated-key>' --name 'vault_authentik_secret_key'
```

**Integration Example**:
Configure Grafana to use Authentik for SSO:
```yaml
grafana:
  env:
    GF_AUTH_GENERIC_OAUTH_ENABLED: "true"
    GF_AUTH_GENERIC_OAUTH_NAME: "Authentik"
    GF_AUTH_GENERIC_OAUTH_CLIENT_ID: "<client-id>"
    GF_AUTH_GENERIC_OAUTH_CLIENT_SECRET: "<client-secret>"
    GF_AUTH_GENERIC_OAUTH_AUTH_URL: "http://authentik-server/application/o/authorize/"
    GF_AUTH_GENERIC_OAUTH_TOKEN_URL: "http://authentik-server/application/o/token/"
```

### 5. Traefik Dashboard

**Purpose**: Web UI for monitoring and managing the Traefik ingress controller.

**Features**:
- Real-time traffic monitoring
- Router and service configuration view
- Middleware inspection
- Certificate management
- Health checks

**Default Configuration**:
```yaml
traefik_namespace: kube-system
traefik_dashboard_port: 9000
traefik_dashboard_enabled: true
```

**Access**:
```bash
kubectl port-forward -n kube-system svc/traefik-dashboard 9000:9000
# Open http://localhost:9000/dashboard/
# Note: The trailing slash is important!
```

**Note**: K3s comes with Traefik pre-installed. This role only enables and exposes the dashboard.

## Deployment Patterns

### Independent Service Deployment

Each service can be deployed independently:

```bash
# Deploy individual services
ansible-playbook -i inventories/production.yaml playbooks/deploy-prometheus-grafana.yaml
ansible-playbook -i inventories/production.yaml playbooks/deploy-blocky.yaml
ansible-playbook -i inventories/production.yaml playbooks/deploy-vault.yaml
ansible-playbook -i inventories/production.yaml playbooks/deploy-authentik.yaml
ansible-playbook -i inventories/production.yaml playbooks/deploy-traefik-dashboard.yaml
```

### All Services at Once

```bash
# Deploy all services
ansible-playbook -i inventories/production.yaml playbooks/deploy-services.yaml
```

### Selective Deployment with Tags

```bash
# Deploy only monitoring
ansible-playbook -i inventories/production.yaml playbooks/deploy-services.yaml --tags monitoring

# Deploy only secrets management
ansible-playbook -i inventories/production.yaml playbooks/deploy-services.yaml --tags vault

# Deploy only SSO
ansible-playbook -i inventories/production.yaml playbooks/deploy-services.yaml --tags authentik
```

### Integrated Deployment

Deploy cluster and services together:

```bash
ansible-playbook -i inventories/production.yaml playbooks/site.yaml -e deploy_services=true
```

## Configuration Management

### Global Service Configuration

Service configuration follows Ansible best practices:

**Configuration Structure:**
- `group_vars/k3s_services.yaml` - Common settings (namespace, Helm repos, service toggles)
- `roles/<role>/defaults/main.yaml` - Service-specific defaults (easily overridden)

Example common settings in `group_vars/k3s_services.yaml`:

```yaml
# Namespace for all services
k3s_services_namespace: homelab

# Service deployment toggles
prometheus_enabled: true
blocky_enabled: true
vault_enabled: true

# Helm repositories (shared across services)
helm_repos:
  - name: prometheus-community
    url: https://prometheus-community.github.io/helm-charts
  - name: hashicorp
    url: https://helm.releases.hashicorp.com
```

Each role's defaults are in `roles/<role>/defaults/main.yaml`. For example, `roles/prometheus-grafana/defaults/main.yaml`:

```yaml
# Chart configuration
prometheus_chart_version: "55.5.0"
prometheus_storage_size: "10Gi"
grafana_storage_size: "5Gi"
grafana_admin_password: "{{ vault_grafana_admin_password }}"
```

### Inventory-Level Configuration

Override role defaults in your inventory file:

```yaml
all:
  vars:
    # Service-specific overrides
    grafana_admin_password: "{{ vault_grafana_admin_password }}"
    vault_storage_size: 20Gi
    blocky_upstream_dns:
      - 1.1.1.1
      - 9.9.9.9
```

To see all available variables for a service, check `roles/<role>/defaults/main.yaml`.

### Using Ansible Vault for Secrets

**Never store secrets in plain text!** Use Ansible Vault:

```bash
# Create vault password file
echo 'your-vault-password' > .vault_pass
chmod 600 .vault_pass

# Encrypt sensitive values
ansible-vault encrypt_string 'strong-password-here' --name 'vault_grafana_admin_password'
ansible-vault encrypt_string 'secret-key-32-chars' --name 'vault_authentik_secret_key'
ansible-vault encrypt_string 'db-password' --name 'vault_authentik_postgresql_password'
```

Add encrypted values to your inventory or group_vars:

```yaml
# In group_vars/all.yaml or your inventory
vault_grafana_admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          ...encrypted...

vault_authentik_secret_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          ...encrypted...
```

## Storage Management

All services use persistent storage. By default, K3s `local-path` provisioner is used.

### Storage Classes

```bash
# View available storage classes
kubectl get storageclass

# Default storage class (K3s)
NAME                   PROVISIONER             RECLAIMPOLICY
local-path (default)   rancher.io/local-path   Delete
```

### Persistent Volumes

```bash
# List persistent volumes
kubectl get pv

# List persistent volume claims
kubectl get pvc -n homelab
```

### Storage Customization

Override storage class in inventory:

```yaml
prometheus_storage_class: "your-storage-class"
vault_storage_class: "your-storage-class"
authentik_storage_class: "your-storage-class"
```

## Networking

### Service Types

Services are deployed with different service types:

- **ClusterIP** (default): Internal cluster access only
- **LoadBalancer**: External access (requires LoadBalancer controller like MetalLB)
- **NodePort**: Access via node IP and port

### Port Forwarding

Access services locally:

```bash
# Port forward examples
kubectl port-forward -n homelab svc/<service-name> <local-port>:<service-port>
```

### Ingress

Enable ingress for external access:

```yaml
# In your inventory or group_vars
grafana_ingress_enabled: true
grafana_ingress_host: grafana.yourdomain.com

prometheus_ingress_enabled: true
prometheus_ingress_host: prometheus.yourdomain.com

vault_ingress_enabled: true
vault_ingress_host: vault.yourdomain.com

authentik_ingress_enabled: true
authentik_ingress_host: auth.yourdomain.com
```

### DNS Configuration

For Blocky DNS:

1. Get the LoadBalancer IP:
   ```bash
   kubectl get svc -n homelab blocky
   ```

2. Configure your devices or DHCP server to use this IP as DNS server

3. Test DNS resolution:
   ```bash
   dig @<blocky-ip> google.com
   ```

## Monitoring and Observability

### Prometheus Metrics

All services expose Prometheus metrics:

- Prometheus: Self-monitoring
- Grafana: Usage metrics
- Blocky: DNS query metrics
- Vault: Performance metrics
- Traefik: Traffic metrics

### Grafana Dashboards

Pre-configured dashboards:

- Kubernetes Cluster Overview
- Node Exporter Full
- Persistent Volumes
- Traefik Dashboard
- Custom application dashboards

Import additional dashboards from [grafana.com](https://grafana.com/grafana/dashboards/).

### Logs

View service logs:

```bash
# Prometheus
kubectl logs -n homelab -l app.kubernetes.io/name=prometheus

# Grafana
kubectl logs -n homelab -l app.kubernetes.io/name=grafana

# Blocky
kubectl logs -n homelab -l app=blocky

# Vault
kubectl logs -n homelab -l app.kubernetes.io/name=vault

# Authentik
kubectl logs -n homelab -l app.kubernetes.io/name=authentik
```

## Security Best Practices

### 1. Change Default Passwords

Never use default passwords in production:

```yaml
grafana_admin_password: "{{ vault_grafana_admin_password }}"
authentik_secret_key: "{{ vault_authentik_secret_key }}"
authentik_postgresql_password: "{{ vault_authentik_postgresql_password }}"
```

### 2. Enable TLS/SSL

Configure ingress with TLS:

```yaml
grafana_ingress_enabled: true
grafana_ingress_tls: true
grafana_ingress_tls_secret: grafana-tls
```

### 3. Network Policies

Implement network policies to restrict traffic:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-grafana
  namespace: homelab
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: grafana
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: homelab
```

### 4. RBAC

Use Kubernetes RBAC for access control:

```bash
# Create service account
kubectl create serviceaccount monitoring-sa -n homelab

# Bind role
kubectl create clusterrolebinding monitoring-sa-binding \
  --clusterrole=view \
  --serviceaccount=homelab:monitoring-sa
```

### 5. Vault Policies

Configure Vault policies for least privilege access:

```hcl
# Policy for application access
path "secret/data/myapp/*" {
  capabilities = ["read"]
}
```

## Troubleshooting

### Service Not Starting

```bash
# Check pod status
kubectl get pods -n homelab

# Check pod events
kubectl describe pod <pod-name> -n homelab

# Check logs
kubectl logs <pod-name> -n homelab
```

### Storage Issues

```bash
# Check PVC status
kubectl get pvc -n homelab

# Check PV status
kubectl get pv

# Describe PVC for events
kubectl describe pvc <pvc-name> -n homelab
```

### Helm Issues

```bash
# List Helm releases
helm list -n homelab

# Get Helm release status
helm status <release-name> -n homelab

# View Helm values
helm get values <release-name> -n homelab
```

### Network Issues

```bash
# Test service connectivity
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- /bin/bash

# Inside the debug pod
nslookup prometheus-grafana.homelab.svc.cluster.local
curl http://prometheus-grafana.homelab.svc.cluster.local
```

### Vault Issues

```bash
# Check Vault status
kubectl exec -n homelab vault-0 -- vault status

# Unseal Vault
kubectl exec -n homelab vault-0 -- vault operator unseal

# Check Vault logs
kubectl logs -n homelab vault-0
```

## Maintenance

### Backup

Backup important data:

```bash
# Backup Grafana dashboards
kubectl exec -n homelab <grafana-pod> -- tar czf - /var/lib/grafana > grafana-backup.tar.gz

# Backup Vault data (ensure Vault is sealed first)
kubectl exec -n homelab vault-0 -- tar czf - /vault/data > vault-backup.tar.gz

# Backup Authentik data
kubectl exec -n homelab <authentik-postgres-pod> -- pg_dump -U authentik authentik > authentik-backup.sql
```

### Updates

Update services:

```bash
# Update Helm repositories
helm repo update

# Upgrade release
helm upgrade prometheus prometheus-community/kube-prometheus-stack -n homelab

# Or re-run playbook
ansible-playbook -i inventories/production.yaml playbooks/deploy-prometheus-grafana.yaml
```

### Scaling

Scale services:

```bash
# Scale Prometheus replicas
kubectl scale statefulset prometheus-prometheus-kube-prometheus-prometheus --replicas=2 -n homelab

# Or update in defaults/main.yaml and re-run playbook
```

## Uninstallation

### Uninstall All Services

```bash
ansible-playbook -i inventories/production.yaml playbooks/uninstall-services.yaml
```

### Uninstall Specific Service

```bash
ansible-playbook -i inventories/production.yaml playbooks/uninstall-services.yaml --tags prometheus
```

### Manual Cleanup

If automated uninstall fails:

```bash
# Delete namespace (will delete all resources)
kubectl delete namespace homelab

# Manually delete PVs if needed
kubectl delete pv <pv-name>
```

## Support and Contributions

For issues, questions, or contributions, please refer to the main repository documentation.
