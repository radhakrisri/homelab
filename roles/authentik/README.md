# Authentik Role

This role deploys Authentik, a Single Sign-On (SSO) and identity provider for homelab services.

## Features

- SAML 2.0, OAuth2, and OpenID Connect support
- LDAP provider
- User and group management
- Multi-factor authentication (MFA)
- Flow-based authentication
- Web UI for administration

## Requirements

- Kubernetes cluster with kubectl access
- Helm 3+
- kubernetes.core Ansible collection

## Role Variables

See `defaults/main.yaml` for all available variables.

Key variables:
- `authentik_namespace`: Namespace for deployment (default: homelab)
- `authentik_secret_key`: Secret key for Authentik - **MUST be changed!**
- `authentik_postgresql_password`: PostgreSQL password - **MUST be changed!**
- `authentik_storage_size`: Storage size for media files (default: 5Gi)

## Example Playbook

```yaml
- hosts: k3s_servers[0]
  vars:
    kubeconfig_path: /path/to/kubeconfig
    authentik_secret_key: "{{ vault_authentik_secret_key }}"
    authentik_postgresql_password: "{{ vault_authentik_postgresql_password }}"
  roles:
    - authentik
```

## Tags

- authentik
- sso

## Post-Deployment Steps

1. Access Authentik:
```bash
kubectl port-forward -n homelab svc/authentik-server 80:80
# Open http://localhost
```

2. Follow the setup wizard to create your first admin user

3. Configure authentication flows

4. Create applications and providers for your services

## Security Warning

⚠️ **MUST change the default secret key before deployment!**

Generate a secure secret key:
```bash
openssl rand -hex 32
```

Store in Ansible Vault:
```bash
ansible-vault encrypt_string '<generated-key>' --name 'vault_authentik_secret_key'
ansible-vault encrypt_string '<db-password>' --name 'vault_authentik_postgresql_password'
```

## Integration

Authentik can be integrated with many services:
- Grafana (OAuth2)
- Vault (OIDC)
- Traefik (Forward Auth)
- Any SAML/OAuth2/OIDC compatible application

See the Authentik documentation for integration guides.
