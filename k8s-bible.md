# Comprehensive Kubernetes Resources and Advanced Scheduling Cheat Sheet

A single-stop reference guide (a "Kubernetes bible") for the key objects, structures, and best practices in Kubernetes. This includes workload resources, networking, storage, security, advanced scheduling (node affinity, taints, etc.), and more.  

---

## Table of Contents

1. [Kubernetes Object Structure](#1-kubernetes-object-structure)  
2. [Workload Resources](#2-workload-resources)  
   - [2.1 Pods](#21-pods)  
   - [2.2 ReplicaSet](#22-replicaset)  
   - [2.3 Deployment](#23-deployment)  
   - [2.4 StatefulSet](#24-statefulset)  
   - [2.5 DaemonSet](#25-daemonset)  
   - [2.6 Job & CronJob](#26-job--cronjob)  
3. [Service & Networking](#3-service--networking)  
   - [3.1 Service](#31-service)  
   - [3.2 Ingress & IngressClass](#32-ingress--ingressclass)  
   - [3.3 NetworkPolicy](#33-networkpolicy)  
   - [3.4 DNS & Endpoints](#34-dns--endpoints)  
4. [Storage](#4-storage)  
   - [4.1 Volumes & Volume Types](#41-volumes--volume-types)  
   - [4.2 PersistentVolume (PV)](#42-persistentvolume-pv)  
   - [4.3 PersistentVolumeClaim (PVC)](#43-persistentvolumeclaim-pvc)  
   - [4.4 StorageClass](#44-storageclass)  
5. [Configuration & Secrets](#5-configuration--secrets)  
   - [5.1 ConfigMap](#51-configmap)  
   - [5.2 Secret](#52-secret)  
6. [Security & Policies](#6-security--policies)  
   - [6.1 ServiceAccount](#61-serviceaccount)  
   - [6.2 RBAC (Roles, ClusterRoles, RoleBindings, ClusterRoleBindings)](#62-rbac-roles-clusterroles-rolebindings-clusterrolebindings)  
   - [6.3 PodSecurity Standards](#63-podsecurity-standards)  
7. [Resource Quotas & Limits](#7-resource-quotas--limits)  
   - [7.1 ResourceQuota](#71-resourcequota)  
   - [7.2 LimitRange](#72-limitrange)  
8. [Autoscaling & Scheduling](#8-autoscaling--scheduling)  
   - [8.1 HorizontalPodAutoscaler (HPA)](#81-horizontalpodautoscaler-hpa)  
   - [8.2 PodDisruptionBudget (PDB)](#82-poddisruptionbudget-pdb)  
   - [8.3 PriorityClass](#83-priorityclass)  
   - [8.4 Node Selection (nodeSelector, nodeName)](#84-node-selection-nodeselector-nodename)  
   - [8.5 Node Affinity & Anti-Affinity](#85-node-affinity--anti-affinity)  
   - [8.6 Pod Affinity & Pod Anti-Affinity](#86-pod-affinity--pod-anti-affinity)  
   - [8.7 Taints & Tolerations](#87-taints--tolerations)  
9. [Custom Resources & Operators](#9-custom-resources--operators)  
   - [9.1 CustomResourceDefinition (CRD)](#91-customresourcedefinition-crd)  
   - [9.2 Operators](#92-operators)  
10. [Common kubectl Commands](#10-common-kubectl-commands)  
11. [Additional Advanced Topics](#11-additional-advanced-topics)  
   - [11.1 Ephemeral Containers](#111-ephemeral-containers)  
   - [11.2 Sidecar Containers](#112-sidecar-containers)  
   - [11.3 Extended Resources (GPUs, etc.)](#113-extended-resources-gpus-etc)  
12. [General Best Practices & Tips](#12-general-best-practices--tips)  
13. [References & Further Reading](#13-references--further-reading)  

---

## 1. Kubernetes Object Structure

Every Kubernetes resource has a standard structure in YAML (or JSON):

```yaml
apiVersion: <group/version>    # e.g., apps/v1, v1, batch/v1, etc.
kind: <ResourceKind>           # e.g., Pod, Deployment, Service
metadata:
  name: <Name>
  namespace: <Namespace>       # Defaults to 'default' if not specified
  labels:
    key: value
  annotations:
    key: value
spec:                          # Desired state
  ...
status:                        # Current state (read-only, managed by K8s)
  ...
```

- **apiVersion**: API group + version, for example `apps/v1`, `batch/v1`, `networking.k8s.io/v1`.
- **kind**: Resource type (e.g., `Pod`, `Deployment`, `Service`).
- **metadata**: Identifiers, labels, annotations.
- **spec**: Describes the desired configuration.
- **status**: Describes the current state (Kubernetes populates this automatically).

---

## 2. Workload Resources

### 2.1 Pods

- **Definition**: The smallest deployable unit in Kubernetes. A Pod can run one or more containers that share networking and storage.
- **Common Fields**:
  - `spec.containers[]`: Defines each container’s image, env variables, ports, etc.
  - `spec.initContainers[]`: Containers that run first for setup tasks.
  - `spec.volumes[]`: Volumes mounted into containers.
  - `spec.nodeSelector`, `spec.affinity`: Scheduling constraints.
  - `spec.tolerations`: Tolerations for taints on nodes.
- **Usage**: Often managed by higher-level controllers like Deployments.

**Example Pod Manifest**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: app-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

---

### 2.2 ReplicaSet

- **Definition**: Ensures a specified number of Pod replicas are running at all times.
- **Key Fields**:
  - `spec.replicas`: Desired number of Pods.
  - `spec.selector`: Label selector for the Pods.
  - `spec.template`: Pod template to create new replicas.
- **Usage**: Typically used indirectly via Deployments. Rarely used alone.

**Example ReplicaSet**:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: example-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
        - name: example-container
          image: nginx:latest
```

---

### 2.3 Deployment

- **Definition**: Higher-level controller managing ReplicaSets and Pods. Provides rolling updates and rollbacks.
- **Key Fields**:
  - `spec.replicas`
  - `spec.selector`
  - `spec.template`
  - `spec.strategy.type`: RollingUpdate or Recreate.
  - `spec.strategy.rollingUpdate`: Fine-tune `maxSurge`, `maxUnavailable`.
- **Usage**: Most common controller for stateless applications.

**Example Deployment**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
        - name: example-container
          image: nginx:latest
```

---

### 2.4 StatefulSet

- **Definition**: Manages stateful applications. Pods have stable identities, ordered deployments, and persistent storage.
- **Key Fields**:
  - `spec.serviceName`: Must reference a headless service.
  - `spec.replicas`
  - `spec.selector`
  - `spec.template`
  - `spec.volumeClaimTemplates[]`
- **Usage**: For databases or applications needing stable network IDs or storage.

**Example StatefulSet**:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: example-statefulset
spec:
  serviceName: "example-headless"
  replicas: 3
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
        - name: example-container
          image: mysql:5.7
          volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "standard"
        resources:
          requests:
            storage: 1Gi
```

---

### 2.5 DaemonSet

- **Definition**: Ensures that all (or some) Nodes run a copy of a specific Pod.
- **Key Fields**:
  - `spec.selector`
  - `spec.template`
- **Usage**: Cluster-wide agents such as logging, monitoring, or networking daemons.

**Example DaemonSet**:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: example-daemonset
spec:
  selector:
    matchLabels:
      app: daemon
  template:
    metadata:
      labels:
        app: daemon
    spec:
      containers:
        - name: example-sidecar
          image: fluentd:latest
```

---

### 2.6 Job & CronJob

#### Job

- **Definition**: Runs Pods to completion (for batch processing).
- **Key Fields**:
  - `spec.template`: Pod template
  - `spec.completions`: Number of Pods that need to finish.
  - `spec.parallelism`: Number of Pods that run in parallel.
- **Usage**: For one-time tasks (scripts, data processing).

#### CronJob

- **Definition**: Schedules Jobs based on time (cron syntax).
- **Key Fields**:
  - `spec.schedule`: Cron schedule.
  - `spec.jobTemplate`: Defines the Job to run.
  - `spec.concurrencyPolicy`: `Allow`, `Forbid`, or `Replace`.
  - `spec.successfulJobsHistoryLimit` / `spec.failedJobsHistoryLimit`.
- **Usage**: Recurring tasks, backups, etc.

**Example CronJob**:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: example-cron
spec:
  schedule: "0 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: example-cron-container
              image: bash:latest
              command: ["/bin/bash"]
              args: ["-c", "echo Hello from CronJob!"]
          restartPolicy: OnFailure
```

---

## 3. Service & Networking

### 3.1 Service

- **Definition**: Provides a stable IP and DNS name to a set of Pods.
- **Types**:
  - `ClusterIP` (default): Internal only.
  - `NodePort`: Exposed on a port on each Node.
  - `LoadBalancer`: External LB through a cloud provider.
  - `ExternalName`: Maps a Service to an external DNS name.
- **Key Fields**:
  - `spec.selector`
  - `spec.ports[]`
- **Usage**: Stable endpoint for a group of Pods.

**Example Service**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-service
spec:
  type: ClusterIP
  selector:
    app: example
  ports:
    - name: http
      port: 80
      targetPort: 80
```

---

### 3.2 Ingress & IngressClass

#### Ingress
- **Definition**: Manages external HTTP/HTTPS access to Services.
- **Key Fields**:
  - `spec.rules[]`: Host/path routing.
  - `spec.tls[]`: TLS configuration.
- **Usage**: Terminates or routes external traffic into the cluster.

#### IngressClass
- **Definition**: Defines which controller (e.g., nginx, haproxy) implements the Ingress.
- **Usage**: Ties an Ingress resource to a specific Ingress controller in the cluster.

**Example Ingress**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: example-service
                port:
                  number: 80
  tls:
    - hosts:
        - example.com
      secretName: example-tls-secret
```

---

### 3.3 NetworkPolicy

- **Definition**: Controls traffic (ingress/egress) at the IP/port level.
- **Key Fields**:
  - `spec.podSelector`
  - `spec.policyTypes`: `Ingress`, `Egress`, or both.
  - `spec.ingress[]` and/or `spec.egress[]`: Allowed traffic rules.
- **Usage**: Secure Pod communication by restricting or allowing specific traffic patterns.

**Example NetworkPolicy**:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example-network-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              environment: production
```

---

### 3.4 DNS & Endpoints

- **DNS**: Kubernetes automatically assigns DNS names for Services and Pods. Services are typically accessible via `<service-name>.<namespace>.svc.cluster.local`.
- **Endpoints**: Tracks IP addresses and ports of the Pods in a Service.
- **EndpointSlice**: A scalable approach to storing endpoints for larger Services.

---

## 4. Storage

### 4.1 Volumes & Volume Types

- **emptyDir**: Ephemeral storage for a Pod’s lifetime.  
- **hostPath**: Direct path on the host node.  
- **configMap / secret**: Mount ConfigMap/Secret data as files.  
- **PersistentVolumeClaim**: Abstracted persistent storage.  
- **NFS, gcePersistentDisk, awsElasticBlockStore**, etc.: Specialized external storage options.

---

### 4.2 PersistentVolume (PV)

- **Definition**: A piece of storage with certain capacity and access modes.
- **Key Fields**:
  - `spec.capacity.storage`
  - `spec.accessModes` (e.g., `ReadWriteOnce`, `ReadWriteMany`)
  - `spec.persistentVolumeReclaimPolicy`: `Recycle`, `Delete`, `Retain`
  - `spec.storageClassName`
- **Usage**: Provides a cluster-wide pool of storage that can be claimed.

**Example PV**:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/data"
```

---

### 4.3 PersistentVolumeClaim (PVC)

- **Definition**: A request for storage by a user. Binds to a matching PV.
- **Key Fields**:
  - `spec.accessModes`
  - `spec.resources.requests.storage`
  - `spec.storageClassName`
- **Usage**: Pods use PVCs to reference the bound storage volume.

**Example PVC**:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
```

---

### 4.4 StorageClass

- **Definition**: Defines how to dynamically provision PVs.
- **Key Fields**:
  - `provisioner`: e.g., `kubernetes.io/aws-ebs`, `kubernetes.io/gce-pd`.
  - `parameters`: Backend-specific options.
  - `reclaimPolicy`: Behavior after PVC is released.
  - `volumeBindingMode`: e.g., `Immediate`, `WaitForFirstConsumer`.
- **Usage**: Allows on-demand creation of PersistentVolumes.

**Example StorageClass**:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-storageclass
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

---

## 5. Configuration & Secrets

### 5.1 ConfigMap

- **Definition**: Stores non-confidential configuration data as key-value pairs.
- **Key Fields**:
  - `data`: Key-value pairs.
- **Usage**: Decouple configuration from container images. Mount as volumes or use as environment variables.

**Example ConfigMap**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  config.yml: |
    someSetting: "value"
    anotherSetting: "anotherValue"
```

---

### 5.2 Secret

- **Definition**: Stores sensitive data (passwords, tokens, keys) in base64-encoded form.
- **Types**: `Opaque`, `kubernetes.io/tls`, `kubernetes.io/dockerconfigjson`, etc.
- **Key Fields**:
  - `data`: Base64-encoded key-value pairs.
  - `stringData`: Unencoded strings (Kubernetes will encode them automatically).
- **Usage**: Pass sensitive info to Pods as env variables or mounted volumes. 

**Example Secret**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-secret
type: Opaque
data:
  username: c3lzdGVtYWRtaW4=
  password: cGFzc3dvcmQ=
```

---

## 6. Security & Policies

### 6.1 ServiceAccount

- **Definition**: Provides an identity for processes that run in a Pod.
- **Usage**: Needed when a Pod needs to interact with the Kubernetes API or other services with specific RBAC permissions.

**Example ServiceAccount**:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: example-serviceaccount
```

---

### 6.2 RBAC (Roles, ClusterRoles, RoleBindings, ClusterRoleBindings)

- **Definition**: Role-Based Access Control for Kubernetes API actions.
- **Role**: Permissions within a specific namespace.
- **ClusterRole**: Permissions cluster-wide.
- **RoleBinding**: Grants a Role to subjects (users, groups, ServiceAccounts) in a namespace.
- **ClusterRoleBinding**: Grants a ClusterRole to subjects cluster-wide.

**Example Role & RoleBinding**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: example-role
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: example-rolebinding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: example-serviceaccount
roleRef:
  kind: Role
  name: example-role
  apiGroup: rbac.authorization.k8s.io
```

---

### 6.3 PodSecurity Standards

- **Definition**: Labels on a namespace enforce pod security profiles (`privileged`, `baseline`, `restricted`).
- **Usage**: Restrict capabilities, volume types, host networking, etc. by namespace.

**Example Label for PodSecurity**:
```bash
kubectl label namespace my-namespace \
  pod-security.kubernetes.io/enforce=restricted
```

> PodSecurityPolicy is deprecated and removed in Kubernetes 1.25+; use PodSecurity admission instead.

---

## 7. Resource Quotas & Limits

### 7.1 ResourceQuota

- **Definition**: Limits overall resource usage (CPU, memory, storage, object counts) in a namespace.
- **Key Fields**:
  - `spec.hard`
- **Usage**: Prevents a namespace from consuming more resources than allocated.

**Example ResourceQuota**:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: example-quota
  namespace: development
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: "2Gi"
    limits.cpu: "10"
    limits.memory: "4Gi"
```

---

### 7.2 LimitRange

- **Definition**: Sets min/max constraints and default requests/limits for Pods/Containers in a namespace.
- **Key Fields**:
  - `spec.limits[]`
- **Usage**: Ensures containers have proper default resources and cannot exceed certain limits.

**Example LimitRange**:
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: example-limitrange
  namespace: development
spec:
  limits:
    - type: Container
      defaultRequest:
        cpu: 100m
        memory: 128Mi
      default:
        cpu: 500m
        memory: 512Mi
      max:
        cpu: "1"
        memory: 1Gi
      min:
        cpu: 50m
        memory: 64Mi
```

---

## 8. Autoscaling & Scheduling

### 8.1 HorizontalPodAutoscaler (HPA)

- **Definition**: Scales the number of Pods in a Deployment, ReplicaSet, or StatefulSet based on metrics (CPU, memory, custom).
- **Key Fields**:
  - `spec.scaleTargetRef`: The target resource to scale.
  - `spec.minReplicas`, `spec.maxReplicas`
  - `spec.metrics[]`
- **Usage**: Automatic scaling up/down based on workload.

**Example HPA**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: example-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: example-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

### 8.2 PodDisruptionBudget (PDB)

- **Definition**: Limits how many Pods of a replicated application can be down voluntarily at once.
- **Key Fields**:
  - `spec.selector`
  - `spec.minAvailable` or `spec.maxUnavailable`
- **Usage**: Ensures high availability during Node drains or cluster operations.

**Example PDB**:
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: example-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: example
```

---

### 8.3 PriorityClass

- **Definition**: Assigns a priority to Pods, which affects scheduling and eviction order.
- **Key Fields**:
  - `value`: Integer priority value.
  - `globalDefault`: If true, this priority class is used for Pods without a specified priority.
- **Usage**: Protect critical workloads from preemption or eviction.

**Example PriorityClass**:
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
globalDefault: false
description: "High priority workloads."
```

---

### 8.4 Node Selection (nodeSelector, nodeName)

- **nodeSelector**: A simple key-value match to schedule Pods on nodes that have certain labels.
  ```yaml
  spec:
    nodeSelector:
      disktype: ssd
  ```
- **nodeName**: Directly specify the node’s name for a Pod, bypassing the scheduler (use with caution).
  ```yaml
  spec:
    nodeName: mynode
  ```

**Example Pod with nodeSelector**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
    - name: nginx
      image: nginx:latest
```

---

### 8.5 Node Affinity & Anti-Affinity

- **Node Affinity** (more flexible than nodeSelector):
  - `requiredDuringSchedulingIgnoredDuringExecution`: Hard constraints.
  - `preferredDuringSchedulingIgnoredDuringExecution`: Soft constraints.

**Example Deployment with NodeAffinity**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: node-affinity-example
  template:
    metadata:
      labels:
        app: node-affinity-example
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: disktype
                    operator: In
                    values:
                      - ssd
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 1
              preference:
                matchExpressions:
                  - key: region
                    operator: In
                    values:
                      - us-east
      containers:
        - name: example-container
          image: nginx:latest
```

---

### 8.6 Pod Affinity & Pod Anti-Affinity

- **Pod Affinity**: Co-locate Pods that match certain labels.
- **Pod Anti-Affinity**: Spread out Pods that match certain labels.

**Example Pod Anti-Affinity**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-anti-affinity
spec:
  replicas: 3
  selector:
    matchLabels:
      app: anti-affinity-example
  template:
    metadata:
      labels:
        app: anti-affinity-example
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  app: anti-affinity-example
              topologyKey: "kubernetes.io/hostname"
      containers:
        - name: example-container
          image: nginx:latest
```

---

### 8.7 Taints & Tolerations

- **Taints**: Applied on Nodes to repel Pods that do not tolerate them.
  ```bash
  kubectl taint nodes <node-name> key=value:NoSchedule
  ```
- **Tolerations**: Applied on Pods to allow scheduling on nodes with matching taints.

**Example Pod with Toleration**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-example
spec:
  tolerations:
    - key: "example-key"
      operator: "Equal"
      value: "example-value"
      effect: "NoSchedule"
  containers:
    - name: nginx
      image: nginx:latest
```

---

## 9. Custom Resources & Operators

### 9.1 CustomResourceDefinition (CRD)

- **Definition**: Extend Kubernetes API with custom resource types.
- **Key Fields**:
  - `spec.group`
  - `spec.versions[]`
  - `spec.names`
  - `spec.scope`
- **Usage**: Build domain-specific automation or operators.

**Example CRD**:
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: widgets.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                size:
                  type: integer
  scope: Namespaced
  names:
    plural: widgets
    singular: widget
    kind: Widget
    shortNames:
      - w
```

---

### 9.2 Operators

- **Definition**: A design pattern that uses CRDs + Controllers to automate application lifecycle management (install, upgrade, scale, backup, etc.).
- **Usage**: Complex stateful apps like databases or streaming platforms (Kafka, etc.) often use an Operator to manage them inside Kubernetes.

---

## 10. Common kubectl Commands

1. **Get and Describe**:
   ```bash
   kubectl get pods
   kubectl get deployments
   kubectl describe pod <pod-name>
   ```
2. **Apply and Create**:
   ```bash
   kubectl apply -f <file|dir>
   kubectl create -f <file|dir>
   ```
3. **Edit and Delete**:
   ```bash
   kubectl edit <resource> <name>
   kubectl delete <resource> <name>
   ```
4. **Logs and Exec**:
   ```bash
   kubectl logs <pod-name> [-c <container-name>]
   kubectl exec -it <pod-name> -- /bin/sh
   ```
5. **Scaling and Rollout**:
   ```bash
   kubectl scale deployment <name> --replicas=5
   kubectl rollout status deployment <name>
   kubectl rollout history deployment <name>
   kubectl rollout undo deployment <name> --to-revision=<rev>
   ```
6. **Context and Namespace**:
   ```bash
   kubectl config get-contexts
   kubectl config use-context <context-name>
   kubectl config set-context --current --namespace=<namespace>
   ```
7. **Port-Forward**:
   ```bash
   kubectl port-forward <pod-name> <local-port>:<pod-port>
   ```
8. **API Resources and Versions**:
   ```bash
   kubectl api-resources
   kubectl api-versions
   ```

---

## 11. Additional Advanced Topics

### 11.1 Ephemeral Containers

- **Definition**: Special debugging containers that can be temporarily added to a running Pod (without restarting it).
- **Usage**: Troubleshoot a running container without modifying the Pod spec.

```bash
kubectl debug <pod-name> -it --image=busybox --target=<container-name>
```

### 11.2 Sidecar Containers

- **Definition**: Additional containers running alongside main containers in the same Pod. Often for logs, proxies, or monitoring.
- **Usage**: Include them in `spec.containers[]` or `spec.initContainers[]` for specialized tasks.

### 11.3 Extended Resources (GPUs, etc.)

- **Definition**: Custom hardware resources on nodes (e.g., `nvidia.com/gpu`).
- **Usage**: Label nodes and request them in Pod specs:
  ```yaml
  resources:
    limits:
      nvidia.com/gpu: 1
  ```

---

## 12. General Best Practices & Tips

1. **Deployments Over ReplicaSets** for stateless apps.  
2. **Separate Config and Secrets** using ConfigMaps and Secrets.  
3. **Use Probes** (`livenessProbe`, `readinessProbe`) for robust health checks.  
4. **Consistent Labeling** to aid in grouping and querying.  
5. **Namespace Organization** for environment/teams (dev, staging, prod).  
6. **Secure by Default**: 
   - Enforce Pod Security standards (e.g., restricted).  
   - Restrict Pod communication with NetworkPolicies.  
   - Use RBAC for fine-grained permissions.  
7. **Set Resource Requests/Limits** so the scheduler can make informed decisions and to avoid resource contention.  
8. **Autoscale** (HPA) if possible, to handle variable workloads.  
9. **Monitor & Observe** with metrics, logs, and tracing (Prometheus, Grafana, Fluentd, Jaeger).  
10. **Rolling Updates & Rollbacks**: Keep track of application versions for quick rollback.  
11. **Use Node Affinity Over nodeSelector** for flexible scheduling.  
12. **Combine Taints/Tolerations With Affinity**: Taints repel unwanted Pods; affinity can attract Pods to certain nodes.  
13. **Test Scheduling Rules** in non-production namespaces before rolling out cluster-wide changes.

---

## 13. References & Further Reading

- [Official Kubernetes Documentation](https://kubernetes.io/docs)  
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api)  
- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)  
- [CNCF & Kubernetes Best Practices](https://www.cncf.io/)  
- Cloud Provider Guides (e.g., [GKE](https://cloud.google.com/kubernetes-engine), [EKS](https://aws.amazon.com/eks), [AKS](https://azure.microsoft.com/en-us/products/kubernetes-service/))  

---

**End of Cheat Sheet**  
```
