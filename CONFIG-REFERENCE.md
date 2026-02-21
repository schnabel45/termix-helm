# Termix Helm Chart - Configuration Reference

Complete reference for all configurable values in the Termix Helm chart.

## Image Configuration

```yaml
image:
  repository: ghcr.io/lukegus/termix    # Docker image repository
  pullPolicy: IfNotPresent              # Image pull policy (IfNotPresent, Always, Never)
  tag: "latest"                         # Image tag override
```

## Replica & Scaling

```yaml
replicaCount: 1                         # Number of replicas (ignored if autoscaling is enabled)

autoscaling:
  enabled: false                        # Enable Horizontal Pod Autoscaler
  minReplicas: 1                        # Minimum replicas
  maxReplicas: 3                        # Maximum replicas
  targetCPUUtilizationPercentage: 80    # Target CPU usage
  targetMemoryUtilizationPercentage: 80 # Target memory usage
```

## Service Configuration

```yaml
service:
  type: ClusterIP                       # Service type (ClusterIP, NodePort, LoadBalancer)
  port: 80                              # Service port
  targetPort: 8080                      # Container port
  annotations: {}                       # Service annotations
```

**Service Types:**
- `ClusterIP` - Internal cluster access only (default)
- `NodePort` - Access via node IP and port
- `LoadBalancer` - External load balancer

## Ingress Configuration

```yaml
ingress:
  enabled: false                        # Enable Ingress
  className: "nginx"                    # Ingress class (nginx, traefik, etc.)
  annotations: {}                       # Additional annotations
    # cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: termix.example.com          # Hostname
      paths:
        - path: /                       # URL path
          pathType: Prefix              # Path type (Prefix, Exact, ImplementationSpecific)
  tls: []                               # TLS configuration
    # - secretName: termix-tls
    #   hosts:
    #     - termix.example.com
```

**Common Annotations:**
```yaml
# Let's Encrypt certificate via cert-manager
cert-manager.io/cluster-issuer: letsencrypt-prod

# NGINX rate limiting
nginx.ingress.kubernetes.io/limit-rps: "100"

# NGINX authentication
nginx.ingress.kubernetes.io/auth-type: basic
nginx.ingress.kubernetes.io/auth-secret: basic-auth

# CORS
nginx.ingress.kubernetes.io/enable-cors: "true"
nginx.ingress.kubernetes.io/cors-allow-origin: "*"
```

## Persistence (PVC)

```yaml
persistence:
  enabled: true                         # Enable PersistentVolumeClaim
  storageClassName: "default"           # Storage class name
  accessMode: ReadWriteOnce             # Access mode
    # ReadWriteOnce (RWO) - Single node read/write
    # ReadOnlyMany (ROX) - Multiple nodes read-only
    # ReadWriteMany (RWX) - Multiple nodes read/write
  size: 10Gi                            # PVC size
  mountPath: /app/data                  # Container mount path
  annotations: {}                       # PVC annotations
```

**Check available storage classes:**
```bash
kubectl get storageclass
```

## Resource Limits & Requests

```yaml
resources:
  limits:                               # Maximum resources allowed
    cpu: 500m                           # CPU limit
    memory: 512Mi                       # Memory limit
  requests:                             # Minimum resources guaranteed
    cpu: 100m                           # CPU request
    memory: 128Mi                       # Memory request
```

**CPU & Memory Units:**
- CPU: `100m` (millicore), `1` (1000m), `0.5` (500m)
- Memory: `128Mi` (MiB), `1Gi` (GiB), `512M` (MB)

## Security & Access Control

```yaml
serviceAccount:
  create: true                          # Create service account
  name: ""                              # Service account name (auto-generated if empty)
  annotations: {}                       # Service account annotations

podSecurityContext: {}                  # Pod-level security context
  # fsGroup: 2000
  # runAsUser: 1000
  # runAsNonRoot: true

securityContext: {}                     # Container-level security context
  # capabilities:
  #   drop:
  #     - ALL
  # readOnlyRootFilesystem: true
```

## Pod Configuration

```yaml
podAnnotations: {}                      # Pod annotations

imagePullSecrets: []                    # Private registry credentials
  # - name: regcred

nameOverride: ""                        # Override chart name
fullnameOverride: ""                    # Override full release name

nodeSelector: {}                        # Node selector
  # disktype: ssd
  # node-role: app

tolerations: []                         # Tolerations for tainted nodes
  # - key: "key1"
  #   operator: "Equal"
  #   value: "value1"
  #   effect: "NoSchedule"

affinity: {}                            # Pod affinity rules
  # podAntiAffinity:
  #   preferredDuringSchedulingIgnoredDuringExecution:
  #   - weight: 100
  #     podAffinityTerm:
  #       labelSelector:
  #         matchExpressions:
  #         - key: app.kubernetes.io/name
  #           operator: In
  #           values:
  #           - termix
```

## Environment Variables

```yaml
env:                                    # Environment variables
  PORT: "8080"                          # Application port
  # Add custom variables as needed
  # LOG_LEVEL: "INFO"
  # CUSTOM_VAR: "value"
```

## Container Configuration

```yaml
containerPort: 8080                     # Container port number
```

## Advanced Configuration Examples

### Production Setup

```yaml
replicaCount: 3
image:
  tag: "v1.0.0"
  pullPolicy: Always
persistence:
  enabled: true
  size: 50Gi
  storageClassName: "fast-ssd"
ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
  hosts:
    - host: termix.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: termix-tls
      hosts:
        - termix.example.com
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi
nodeSelector:
  node-role: app
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app.kubernetes.io/name
              operator: In
              values:
                - termix
        topologyKey: kubernetes.io/hostname
```

### Development Setup

```yaml
replicaCount: 1
image:
  tag: "latest"
  pullPolicy: Always
persistence:
  enabled: false                        # Disable for ephemeral dev
ingress:
  enabled: false                        # Use port-forward instead
service:
  type: NodePort                        # Access via node
autoscaling:
  enabled: false
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 50m
    memory: 64Mi
```

### High-Availability Setup

```yaml
replicaCount: 3
persistence:
  enabled: true
  accessMode: ReadWriteMany             # Multi-node access
  storageClassName: "nfs"               # NFS for shared storage
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app.kubernetes.io/name
              operator: In
              values:
                - termix
        topologyKey: kubernetes.io/hostname
```

## Deployment Commands

### Validate Configuration
```bash
# Validate values
helm lint ./termix-helm
helm template termix ./termix-helm --values values.yaml

# Check if manifests are valid Kubernetes
helm template termix ./termix-helm | kubectl apply --dry-run=client -f -
```

### Install with Custom Config
```bash
# Using values file
helm install termix ./termix-helm -f values-prod.yaml

# Override specific values
helm install termix ./termix-helm \
  --set replicaCount=2 \
  --set persistence.size=20Gi \
  --set ingress.enabled=true
```

### Update Configuration
```bash
# Get current values
helm get values termix

# Upgrade with new values
helm upgrade termix ./termix-helm -f new-values.yaml
```

## Troubleshooting Configuration

### Check Applied Configuration
```bash
# Get rendered manifests
helm get manifest termix

# Validate pod is using correct config
kubectl get pod -l app.kubernetes.io/name=termix -o yaml

# Check environment variables
kubectl exec -it <pod-name> -- env | grep PORT
```

### Common Issues

**Pod not starting:**
```bash
# Check resource availability
kubectl describe node
kubectl top node

# Check resource requests
kubectl get pod <pod-name> -o yaml | grep -A 5 resources:
```

**Ingress not working:**
```bash
# Verify ingress controller is running
kubectl get deployment -n ingress-nginx

# Check ingress status
kubectl describe ingress termix
```

**Storage issues:**
```bash
# Check storage class
kubectl get storageclass

# Check PVC binding
kubectl get pvc
kubectl describe pvc termix-data
```

## Related Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/)
- [YAML Syntax](https://yaml.org/)
