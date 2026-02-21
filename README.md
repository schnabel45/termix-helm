# Termix Helm Chart

***THIS HELM CHART WAS WRITTEN BY AI***

A Helm chart for deploying the Termix terminal sharing application on Kubernetes.

THIS WAS WRITTEN BY AI, DO NOT USE THIS IF YOU CRINGE AT THAT FACT

## Features

- **Persistence**: Configure PersistentVolumeClaim for data storage
- **Ingress**: Support for Kubernetes Ingress for external access
- **Scalability**: Deploy multiple instances with replica configuration
- **Auto-scaling**: Optional Horizontal Pod Autoscaler (HPA)
- **Flexible Configuration**: Comprehensive values for customization

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+

## Installation

### Helm Repo
```bash
helm repo add termix https://schnabel45.github.io/termix-helm/
helm repo update
```

### Basic Installation

```bash
helm install termix termix/termix
```

### With Custom Values

```bash
helm install termix ./termix-helm -f custom-values.yaml
```

### Multiple Instances

Deploy multiple instances with different configurations:

```bash
helm install termix-prod ./termix-helm -f prod-values.yaml
helm install termix-test ./termix-helm -f test-values.yaml
```

## Configuration

### Key Configuration Options

#### Persistence

Enable data persistence with PVC:

```yaml
persistence:
  enabled: true
  storageClassName: "default"
  size: 10Gi
  mountPath: /app/data
```

#### Ingress

Enable external access via Ingress:

```yaml
ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: termix.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: termix-tls
      hosts:
        - termix.example.com
```

#### Replicas

Configure number of replicas:

```yaml
replicaCount: 2
```

Or use auto-scaling:

```yaml
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 3
  targetCPUUtilizationPercentage: 80
```

#### Resources

Configure resource limits:

```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

### All Available Values

See `values.yaml` for the complete list of configurable options.

## Usage

### Check Deployment Status

```bash
kubectl get deployment -l app.kubernetes.io/instance=termix
kubectl logs -l app.kubernetes.io/name=termix
```

### Access the Application

#### Via Port-Forward

```bash
kubectl port-forward svc/termix 8080:80
# Visit http://localhost:8080
```

#### Via Ingress (if enabled)

Visit the configured ingress hostname (e.g., `termix.example.com`)

### Update Values

```bash
helm upgrade termix ./termix-helm -f custom-values.yaml
```

### Uninstall

```bash
helm uninstall termix
```

## Examples

### Example 1: Production Deployment

Create `prod-values.yaml`:

```yaml
replicaCount: 3
image:
  tag: "v1.0.0"
persistence:
  enabled: true
  size: 50Gi
  storageClassName: "fast-ssd"
ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: termix.prod.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: termix-prod-tls
      hosts:
        - termix.prod.com
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 75
resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi
```

Deploy:

```bash
helm install termix-prod ./termix-helm -f prod-values.yaml
```

### Example 2: Development Deployment

Create `dev-values.yaml`:

```yaml
replicaCount: 1
persistence:
  enabled: true
  size: 5Gi
  storageClassName: "default"
ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: termix.dev.local
      paths:
        - path: /
          pathType: Prefix
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 50m
    memory: 64Mi
```

Deploy:

```bash
helm install termix-dev ./termix-helm -f dev-values.yaml
```

## Support

For issues and questions, please refer to the Termix project repository.

### THIS HELM CHART WAS WRITTEN BY AI
THIS HELM CHART WAS WRITTEN BY AI, DO NOT USE THIS IF YOU CRINGE AT THAT FACT