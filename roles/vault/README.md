# Vault Role

This role deploys HashiCorp Vault for centralized secret management.

## Features

- Secure secret storage
- Dynamic secrets generation
- Encryption as a service
- Kubernetes secrets injection
- Web UI for management
- Audit logging

## Requirements

- Kubernetes cluster with kubectl access
- Helm 3+
- kubernetes.core Ansible collection

## Role Variables

See `defaults/main.yaml` for all available variables.

Key variables:
- `vault_namespace`: Namespace for deployment (default: homelab)
- `vault_storage_size`: Storage size for Vault data (default: 10Gi)
- `vault_ui_enabled`: Enable web UI (default: true)
- `vault_dev_mode`: Enable dev mode - **NEVER use in production!** (default: false)
- `vault_ha_enabled`: Enable HA mode with Raft storage (default: false)

## Example Playbook

```yaml
- hosts: k3s_servers[0]
  vars:
    kubeconfig_path: /path/to/kubeconfig
  roles:
    - vault
```

## Tags

- vault
- secrets

## Post-Deployment Steps

### Production Mode (vault_dev_mode: false)

1. Initialize Vault (only once):
```bash
kubectl exec -n homelab vault-0 -- vault operator init
```

2. Save the unseal keys and root token securely!

3. Unseal Vault (required after each restart):
```bash
kubectl exec -n homelab vault-0 -- vault operator unseal <key-1>
kubectl exec -n homelab vault-0 -- vault operator unseal <key-2>
kubectl exec -n homelab vault-0 -- vault operator unseal <key-3>
```

### Development Mode (vault_dev_mode: true)

Vault is automatically initialized and unsealed.
- Root token: `root` (or value of `vault_dev_root_token`)

## Access

After deployment:
```bash
kubectl port-forward -n homelab svc/vault 8200:8200
# Open http://localhost:8200
```

## Security Warning

⚠️ **NEVER use dev mode in production!**
⚠️ **Always secure your unseal keys and root token!**
⚠️ **Consider implementing auto-unseal in production!**
