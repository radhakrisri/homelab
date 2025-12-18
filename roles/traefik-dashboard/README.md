# Traefik Dashboard Role

This role enables and exposes the Traefik dashboard for monitoring and managing the ingress controller.

## Features

- Real-time traffic monitoring
- Router and service configuration view
- Middleware inspection
- Certificate management
- Health checks

## Requirements

- K3s cluster (comes with Traefik pre-installed)
- kubernetes.core Ansible collection

## Role Variables

See `defaults/main.yaml` for all available variables.

Key variables:
- `traefik_namespace`: Namespace where Traefik is deployed (default: kube-system)
- `traefik_dashboard_port`: Dashboard port (default: 9000)
- `traefik_dashboard_enabled`: Enable dashboard (default: true)

## Example Playbook

```yaml
- hosts: k3s_servers[0]
  vars:
    kubeconfig_path: /path/to/kubeconfig
  roles:
    - traefik-dashboard
```

## Tags

- traefik
- traefik-dashboard

## Access

After deployment:
```bash
kubectl port-forward -n kube-system svc/traefik-dashboard 9000:9000
# Open http://localhost:9000/dashboard/
# Note: The trailing slash is important!
```

## Note

K3s comes with Traefik pre-installed. This role only enables and exposes the dashboard.
If Traefik is not installed, the role will fail with an error message.
