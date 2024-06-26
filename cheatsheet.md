
## Basic Commands
- kubectl cluster-info
- kubectl get all
- kubectl get pods
- kubectl get services
- kubectl get deployments
- kubectl get nodes
- kubectl describe pod <pod-name>
- kubectl describe service <service-name>
- kubectl describe deployment <deployment-name>

## Managing Pods
- kubectl run <pod-name> --image=<image-name>
- kubectl logs <pod-name>
- kubectl logs -f <pod-name>  # Follow logs
- kubectl exec <pod-name> -- <command>
- kubectl exec -it <pod-name> -- /bin/sh  # Interactive shell
- kubectl delete pod <pod-name>

## Managing Deployments
- kubectl create deployment <deployment-name> --image=<image-name>
- kubectl scale deployment <deployment-name> --replicas=<number>
- kubectl set image deployment/<deployment-name> <container-name>=<new-image>
- kubectl rollout undo deployment/<deployment-name>
- kubectl rollout status deployment/<deployment-name>

## Services and Networking
- kubectl expose pod <pod-name> --port=<port> --target-port=<target-port> --name=<service-name>
- kubectl expose deployment <deployment-name> --type=<service-type> --port=<port> --target-port=<target-port>
- kubectl get services

## ConfigMaps and Secrets
- kubectl create configmap <configmap-name> --from-literal=<key>=<value>
- kubectl create configmap <configmap-name> --from-file=<file-path>
- kubectl create secret generic <secret-name> --from-literal=<key>=<value>
- kubectl create secret generic <secret-name> --from-file=<file-path>
- kubectl get configmaps
- kubectl get secrets

## Persistent Volumes and Claims
- kubectl get pv
- kubectl get pvc
- kubectl describe pv <pv-name>
- kubectl describe pvc <pvc-name>

## Namespaces
- kubectl get namespaces
- kubectl create namespace <namespace-name>
- kubectl config set-context --current --namespace=<namespace-name>

## Other Useful Commands
- kubectl apply -f <file.yaml>
- kubectl delete -f <file.yaml>
- kubectl run <pod-name> --image=<image-name> --dry-run=client -o yaml
- kubectl edit <resource>/<resource-name>
- kubectl port-forward <pod-name> <local-port>:<pod-port>

## Resource Management
```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

## Troubleshooting
- kubectl get events
- kubectl top pod
- kubectl top node

## Advanced Topics
- kubectl patch <resource-type> <resource-name> --type=strategic --patch '<patch>'
- kubectl patch <resource-type> <resource-name> --type=json --patch '<patch>'

## Network Policies Cheat Sheet

### Basic Network Policy Commands
- kubectl get networkpolicies
- kubectl get networkpolicy <policy-name>
- kubectl describe networkpolicy <policy-name>
- kubectl delete networkpolicy <policy-name>

### Creating a Network Policy
- kubectl apply -f <file.yaml>

### Example Network Policy YAML

#### Allow traffic from specific namespace
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-namespace
  namespace: <namespace>
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: <source-namespace>
```

#### Allow traffic from specific pods
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-pods
  namespace: <namespace>
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

#### Allow traffic on specific ports
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific-port
  namespace: <namespace>
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 3306
```

## Helm Cheat Sheet

### Helm Basic Commands
- helm version
- helm repo add <repo-name> <repo-url>
- helm repo update
- helm search repo <chart-name>
- helm show chart <repo-name>/<chart-name>

### Installing and Upgrading Releases
- helm install <release-name> <repo-name>/<chart-name>
- helm upgrade <release-name> <repo-name>/<chart-name>

### Uninstalling Releases
- helm uninstall <release-name>

### Listing Releases
- helm list

### Viewing Release Status
- helm status <release-name>

### Rolling Back Releases
- helm rollback <release-name> <revision-number>

### Helm Charts
#### Create a new Helm chart
- helm create <chart-name>

#### Package a Helm chart
- helm package <chart-directory>

#### Lint a Helm chart
- helm lint <chart-directory>

### Example Helm install command with values file
- helm install <release-name> <repo-name>/<chart-name> -f <values-file.yaml>

### Example values.yaml file for Helm
```yaml
replicaCount: 2

image:
  repository: nginx
  tag: stable
  pullPolicy: IfNotPresent

service:
  name: nginx
  type: NodePort
  port: 80

ingress:
  enabled: true
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  hosts:
    - host: chart-example.local
      paths:
        - path: /
  tls: []

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

nodeSelector: {}

tolerations: []

affinity: {}
```
