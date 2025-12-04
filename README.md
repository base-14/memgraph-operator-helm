# Memgraph Operator Helm Chart

A Helm chart for deploying the Memgraph Kubernetes Operator, which manages MemgraphCluster custom resources for running highly available Memgraph graph database clusters.

## Prerequisites

- Kubernetes 1.26+
- Helm 3.x

## Installation

### Add the Helm repository

```bash
helm repo add memgraph-operator https://memgraph-operator.base14.io
helm repo update
```

### Install the chart

```bash
helm install memgraph-operator memgraph-operator/memgraph-operator -n memgraph-system --create-namespace
```

### Install from source

```bash
git clone https://github.com/base-14/memgraph-operator-helm
cd memgraph-operator-helm
helm install memgraph-operator . -n memgraph-system --create-namespace
```

## Usage

Once the operator is installed, you can create MemgraphCluster resources:

```yaml
apiVersion: memgraph.base14.io/v1alpha1
kind: MemgraphCluster
metadata:
  name: my-cluster
  namespace: default
spec:
  replicas: 3
  image: memgraph/memgraph:2.21.0
  storage:
    size: 10Gi
  replication:
    mode: ASYNC
```

### Connecting to Memgraph

The operator creates two services for each cluster:

- **Write service** (`<cluster-name>-write`): Routes to the MAIN instance for write operations
- **Read service** (`<cluster-name>-read`): Load-balanced across replicas for read operations

```bash
# Write queries (connects to MAIN instance)
kubectl port-forward svc/my-cluster-write 7687:7687

# Read queries (load-balanced across replicas)
kubectl port-forward svc/my-cluster-read 7688:7687
```

## Configuration

The following table lists the configurable parameters and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of operator replicas | `1` |
| `image.repository` | Operator image repository | `base14/memgraph-operator` |
| `image.tag` | Operator image tag | `0.1.1` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `imagePullSecrets` | Image pull secrets | `[]` |
| `nameOverride` | Override chart name | `""` |
| `fullnameOverride` | Override full name | `""` |
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.automount` | Automount service account token | `true` |
| `serviceAccount.annotations` | Service account annotations | `{}` |
| `serviceAccount.name` | Service account name | `""` |
| `podAnnotations` | Pod annotations | `{}` |
| `podLabels` | Pod labels | `{}` |
| `podSecurityContext.runAsNonRoot` | Run as non-root user | `true` |
| `podSecurityContext.seccompProfile.type` | Seccomp profile type | `RuntimeDefault` |
| `securityContext.readOnlyRootFilesystem` | Read-only root filesystem | `true` |
| `securityContext.allowPrivilegeEscalation` | Allow privilege escalation | `false` |
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `256Mi` |
| `resources.requests.cpu` | CPU request | `10m` |
| `resources.requests.memory` | Memory request | `64Mi` |
| `leaderElection.enabled` | Enable leader election | `true` |
| `healthProbes.port` | Health probe port | `8081` |
| `healthProbes.livenessProbe.initialDelaySeconds` | Liveness probe initial delay | `15` |
| `healthProbes.livenessProbe.periodSeconds` | Liveness probe period | `20` |
| `healthProbes.readinessProbe.initialDelaySeconds` | Readiness probe initial delay | `5` |
| `healthProbes.readinessProbe.periodSeconds` | Readiness probe period | `10` |
| `metrics.enabled` | Enable metrics endpoint | `true` |
| `metrics.port` | Metrics port | `8080` |
| `metrics.service.type` | Metrics service type | `ClusterIP` |
| `metrics.service.annotations` | Metrics service annotations | `{}` |
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Tolerations | `[]` |
| `affinity` | Affinity rules | `{}` |
| `rbac.create` | Create RBAC resources | `true` |

### Example: Custom values

```yaml
# custom-values.yaml
replicaCount: 2

resources:
  limits:
    cpu: 1000m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi

metrics:
  enabled: true
  service:
    annotations:
      prometheus.io/scrape: "true"
      prometheus.io/port: "8080"
```

Install with custom values:

```bash
helm install memgraph-operator . -f custom-values.yaml -n memgraph-system --create-namespace
```

## Uninstallation

```bash
helm uninstall memgraph-operator -n memgraph-system
```

**Note:** This will not remove MemgraphCluster CRDs or existing cluster resources. To fully clean up:

```bash
kubectl delete memgraphclusters --all -A
kubectl delete crd memgraphclusters.memgraph.base14.io
```

## Links

- [Memgraph Operator Source](https://github.com/base-14/memgraph-operator)
- [Memgraph Documentation](https://memgraph.com/docs)

## License

See [LICENSE](LICENSE) for details.
