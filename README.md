# nginx-k8s-helm

A Helm chart for deploying an nginx web application on Kubernetes. Runs nginx as a non-root user with a configurable nginx config, health checks, autoscaling, and ingress support.

## Requirements

- Kubernetes 1.23+
- Helm 3.x
- kubectl configured against your cluster

## Install

```bash
# default install
helm install webapp .

# into a specific namespace
helm install webapp . -n production --create-namespace

# with custom values
helm install webapp . --set replicaCount=3
```

## Upgrade

```bash
helm upgrade webapp .

# override values inline
helm upgrade webapp . --set replicaCount=3

# use a values file
helm upgrade webapp . -f values.prod.yaml
```

## Uninstall

```bash
helm uninstall webapp
```

## Access

```bash
kubectl port-forward deployment/webapp 8080:8080
# open http://127.0.0.1:8080
```

## Configuration

All values can be overridden via `--set` or a custom values file.

| Parameter | Description | Default |
|---|---|---|
| `replicaCount` | Number of pod replicas | `2` |
| `image.repository` | Container image repository | `nginxinc/nginx-unprivileged` |
| `image.tag` | Container image tag | `1.27.0` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `service.targetPort` | Container port | `8080` |
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `nginx` |
| `ingress.hosts` | Ingress host rules | `webapp.local` |
| `autoscaling.enabled` | Enable HorizontalPodAutoscaler | `false` |
| `autoscaling.minReplicas` | HPA minimum replicas | `2` |
| `autoscaling.maxReplicas` | HPA maximum replicas | `10` |
| `autoscaling.targetCPUUtilizationPercentage` | HPA CPU target | `80` |
| `autoscaling.targetMemoryUtilizationPercentage` | HPA memory target | `80` |
| `resources.requests.cpu` | CPU request | `100m` |
| `resources.requests.memory` | Memory request | `128Mi` |
| `resources.limits.cpu` | CPU limit | `200m` |
| `resources.limits.memory` | Memory limit | `256Mi` |
| `configmap.enabled` | Mount custom nginx config via ConfigMap | `true` |
| `configmap.nginxConfig` | nginx server block config | see `values.yaml` |
| `serviceAccount.create` | Create a ServiceAccount | `true` |
| `nodeSelector` | Node selector rules | `{}` |
| `tolerations` | Pod tolerations | `[]` |
| `affinity` | Pod affinity rules | `{}` |

## Enable Ingress

```yaml
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: webapp.local
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: webapp-tls
      hosts:
        - webapp.local
```

## Enable Autoscaling

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80
```

## Custom nginx Config

Override the nginx server block via `values.yaml`:

```yaml
configmap:
  enabled: true
  nginxConfig: |
    server {
        listen 8080;
        server_name _;

        location /healthz {
            access_log off;
            return 200 "ok\n";
            add_header Content-Type text/plain;
        }

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
    }
```

## Security

- Runs as non-root user (uid `101`)
- `allowPrivilegeEscalation: false`
- All Linux capabilities dropped
- Uses `nginxinc/nginx-unprivileged` image

## Health Checks

| Endpoint | Type | Response |
|---|---|---|
| `GET /healthz` | liveness + readiness | `200 ok` |

## Deployed Resources

| Kind | Description |
|---|---|
| `Deployment` | Manages nginx pods |
| `Service` | ClusterIP on port 80 |
| `ConfigMap` | nginx server config |
| `ServiceAccount` | Dedicated SA per release |
| `Ingress` | Optional — disabled by default |
| `HorizontalPodAutoscaler` | Optional — disabled by default |
| `PodDisruptionBudget` | Created when `replicaCount > 1` |

## Release History

```bash
helm history webapp
```

## Rollback

```bash
helm rollback webapp 1    # roll back to revision 1
```

## Package

```bash
helm package .            # produces webapp-0.1.0.tgz
```
