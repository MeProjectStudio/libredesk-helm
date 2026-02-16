# [libredesk](https://github.com/abhinavxd/libredesk) Helm Chart

Helm chart for a modern, open source, self-hosted customer support desk.

## Overview

This Helm chart deploys [libredesk](https://github.com/abhinavxd/libredesk) on a Kubernetes cluster.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- PV provisioner support in the underlying infrastructure (for persistence)

## Installation

### Install from OCI Registry

Chart is available on GitHub Container Registry:

```bash
helm pull oci://ghcr.io/meprojectstudio/libredesk-helm --version 1.0.2
```

To install the chart directly:

```bash
helm install libredesk oci://ghcr.io/meprojectstudio/libredesk-helm --version 1.0.2
```

Install with custom values, for example with custom libredesk image tag:

```bash
helm install libredesk oci://ghcr.io/meprojectstudio/libredesk-helm --version 1.0.2 \
  --set image.tag=1.0.1 \
```

## Configuration Overview

LibreDesk configuration is rendered into a `ConfigMap` (`config.toml`) and mounted into the pod. On startup, the container copies the template into an `emptyDir` and optionally replaces placeholders with values pulled from existing Kubernetes Secrets:

- `__ENCRYPTION_KEY__` (from `config.app.existingSecret`)
- `__DB_PASSWORD__` (from `config.db.existingSecret`)
- `__REDIS_PASSWORD__` (from `config.redis.existingSecret`)

The chart also creates a `Secret` for the initial system user password. If a secret already exists, its password is reused.

## Key Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of LibreDesk replicas | `1` |
| `image.repository` | LibreDesk image repository | `libredesk/libredesk` |
| `image.tag` | LibreDesk image tag | Chart appVersion |
| `systemUserPassword.existingSecret.enabled` | Use an existing secret for system user password | `false` |
| `systemUserPassword.existingSecret.name` | Existing secret name for system user password | `""` |
| `systemUserPassword.existingSecret.key` | Existing secret key for system user password | `systemUserPassword` |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `9000` |
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `persistence.size` | Persistent volume size for uploads | `10Gi` |
| `postgresql.enabled` | Deploy PostgreSQL | `true` |
| `postgresql.auth.password` | PostgreSQL password | `libredesk` |
| `redis.enabled` | Deploy Redis | `true` |
| `config.app.encryptionKey` | Encryption key (32 chars) | `CHANGE-ME-32-CHAR-RANDOM-STRING!` |
| `config.app.existingSecret.enabled` | Use an existing secret for encryption key | `false` |
| `config.db.existingSecret.enabled` | Use an existing secret for DB password | `false` |
| `config.redis.existingSecret.enabled` | Use an existing secret for Redis password | `false` |

For a complete list of configuration options, see `chart/values.yaml`.

## Secrets and Passwords

### System user password

By default, the chart creates a `Secret` named `<release>-sensitive` and auto-generates a password if one doesn’t already exist.  
To supply your own secret:

```yaml
systemUserPassword:
  existingSecret:
    enabled: true
    name: "libredesk-sensitive"
    key: "systemUserPassword"
```

### Encryption key, DB password, Redis password

You can store these values in your own Secrets and reference them in `values.yaml`:

```yaml
config:
  app:
    existingSecret:
      enabled: true
      name: "libredesk-app-secret"
      key: "encryptionKey"
  db:
    existingSecret:
      enabled: true
      name: "libredesk-db-secret"
      key: "password"
  redis:
    existingSecret:
      enabled: true
      name: "libredesk-redis-secret"
      key: "password"
```

## Using External Databases

### PostgreSQL

```yaml
postgresql:
  enabled: false

config:
  db:
    host: "external-postgres.example.com"
    port: 5432
    user: "libredesk"
    password: "your-password"
    database: "libredesk"
    sslMode: "require"
```

### Redis

```yaml
redis:
  enabled: false

config:
  redis:
    address: "external-redis.example.com:6379"
    password: "your-redis-password"
```

## Ingress

To expose LibreDesk via Ingress:

```yaml
ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/proxy-body-size: "100m"
  hosts:
    - host: libredesk.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: libredesk-tls
      hosts:
        - libredesk.example.com
```

## Persistence

The chart creates PersistentVolumeClaims for:

- LibreDesk uploads and data
- PostgreSQL database (if enabled)

Specify `persistence.storageClass` to use a specific storage class, or leave empty to use the cluster default.

## Resource Limits

Configure resource requests and limits:

```yaml
resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

postgresql:
  resources:
    limits:
      cpu: 500m
      memory: 512Mi
    requests:
      cpu: 250m
      memory: 256Mi
```

## Upgrading

### Upgrade to a new version

```bash
helm upgrade libredesk oci://ghcr.io/meprojectsudio/libredesk-helm --version <new-version>
```

### Upgrade with new values

```bash
helm upgrade libredesk oci://ghcr.io/meprojectsudio/libredesk-helm --version <new-version> -f values.yaml
```

## Uninstalling

To uninstall/delete the `libredesk` deployment:

```bash
helm uninstall libredesk
```

This command removes all the Kubernetes components associated with the chart and deletes the release.

⚠️ **Note**: PersistentVolumeClaims are not automatically deleted. To delete them:

```bash
kubectl delete pvc -l app.kubernetes.io/name=libredesk
```

## Troubleshooting

### Check pod status

```bash
kubectl get pods -l app.kubernetes.io/name=libredesk
```

### View logs

```bash
kubectl logs -l app.kubernetes.io/name=libredesk
```

### Check configuration

```bash
kubectl get configmap libredesk -o yaml
```

## Links

- **Libredesk homepage**: https://libredesk.io
- **Libredesk sources**: https://github.com/abhinavxd/libredesk
- **Helm Chart Repository**: https://github.com/MeProjectStudio/libredesk-helm

## Support

For issues related to:
- LibreDesk application: https://github.com/abhinavxd/libredesk/issues
- Helm chart: https://github.com/MeProjectStudio/libredesk-helm/issues

## License

This Helm chart is provided as-is under MIT License. Please refer to the Libredesk project for it's own licensing information.
