# LibreDesk Helm Chart

Helm chart for a modern, open source, self-hosted customer support desk.

## Overview

This Helm chart deploys [libredesk](https://github.com/abhinavxd/libredesk) on a Kubernetes cluster

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- PV provisioner support in the underlying infrastructure (for persistence)

## Installation

### Install from OCI Registry
Chart is available on MeProject OCI registry:

```bash
helm pull oci://registry.mprjct.ru/libredesk/libredesk-helm --version 1.0.2
```

To install the chart directly:

```bash
helm install libredesk oci://registry.mprjct.ru/libredesk/libredesk-helm --version 1.0.2
```

### Install with custom values

```bash
helm install libredesk oci://registry.mprjct.ru/libredesk/libredesk-helm --version 1.0.2 \
  --set systemUserPassword=your-secure-password \
  --set config.app.encryptionKey=$(openssl rand -hex 16)
```

### Key Configuration Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of LibreDesk replicas | `1` |
| `image.repository` | LibreDesk image repository | `libredesk/libredesk` |
| `image.tag` | LibreDesk image tag | Chart appVersion |
| `systemUserPassword` | Initial system user password | `changeme123` |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `9000` |
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `persistence.size` | Persistent volume size | `10Gi` |
| `postgresql.enabled` | Deploy PostgreSQL | `true` |
| `postgresql.auth.password` | PostgreSQL password | `libredesk` |
| `redis.enabled` | Deploy Redis | `true` |
| `config.app.encryptionKey` | Encryption key (32 chars) | `CHANGE-ME-32-CHAR-RANDOM-STRING!` |

For a complete list of configuration options, see the `values.yaml` file.

## Upgrading

### Upgrade to a new version

```bash
helm upgrade libredesk oci://registry.mprjct.ru/libredesk/libredesk-helm --version <new-version>
```

### Upgrade with new values

```bash
helm upgrade libredesk oci://registry.mprjct.ru/libredesk/libredesk-helm --version <new-version> -f values.yaml
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

## Components

This chart optionaly allows to deploy components: 

- **libredesk**: The main application (configurable replicas)
- **PostgreSQL**: Database backend (optional, can use external DB)
- **Redis**: Cache and queue management (optional, can use external Redis)

## Persistence

The chart creates PersistentVolumeClaims for:

- LibreDesk uploads and data
- PostgreSQL database (if enabled)

Specify `storageClass` to use a specific storage class, or leave empty to use the cluster default.

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

## Using External Databases

To use an external PostgreSQL database:

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

To use an external Redis:

```yaml
redis:
  enabled: false

config:
  redis:
    address: "external-redis.example.com:6379"
    password: "your-redis-password"
```

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

- **Homepage**: https://libredesk.io
- **libredesk sources**: https://github.com/abhinavxd/libredesk
- **Helm Chart Repository**: https://github.com/MeProjectStudio/libredesk-helm

## Support

For issues related to:
- LibreDesk application: https://github.com/abhinavxd/libredesk/issues
- Helm chart: https://github.com/MeProjectStudio/libredesk-helm/issues

## License

This Helm chart is provided as-is. Please refer to the LibreDesk project for application licensing information.
