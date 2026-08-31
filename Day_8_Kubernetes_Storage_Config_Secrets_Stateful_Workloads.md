# Day 8 — Kubernetes Storage, Config, Secrets & Stateful Workloads

> **30-Day DevOps → MLOps Study Plan**  
> **Focus:** Kubernetes storage, Volumes, `emptyDir`, PersistentVolumes, PersistentVolumeClaims, StorageClasses, dynamic provisioning, CSI, StatefulSets, Headless Services, ConfigMaps, Secrets, external secret management, Azure/AWS storage concepts, backup/recovery, troubleshooting, and senior-level interview preparation  
> **Recommended time:** 4–4.5 hours  
> **Target:** ~6 years DevOps experience

---

# 1. Day 8 Objectives

By the end of Day 8, you should be able to:

- Explain why container-local storage is ephemeral.
- Understand Kubernetes Volumes.
- Understand `emptyDir`.
- Understand PersistentVolumes (PV).
- Understand PersistentVolumeClaims (PVC).
- Understand StorageClasses.
- Explain dynamic provisioning.
- Understand CSI drivers.
- Explain StatefulSets.
- Understand stable Pod identity.
- Understand Headless Services.
- Understand StatefulSet storage using `volumeClaimTemplates`.
- Understand ConfigMaps and Secrets in production.
- Understand Azure Disk / Azure Files concepts.
- Understand AWS EBS / EFS concepts.
- Explain storage backup and recovery.
- Troubleshoot Pending PVCs and mount failures.
- Troubleshoot StatefulSets.
- Complete a Kubernetes storage lab.
- Answer senior-level storage/stateful-workload interview questions.

---

# 2. Recommended Schedule

| Time | Topic |
|---|---|
| 00:00–00:35 | Container storage and Kubernetes Volumes |
| 00:35–01:20 | PV, PVC and StorageClass |
| 01:20–02:00 | Dynamic provisioning and CSI |
| 02:00–02:45 | StatefulSets and Headless Services |
| 02:45–03:15 | ConfigMaps and Secrets |
| 03:15–03:50 | Azure/AWS storage concepts |
| 03:50–04:30 | Hands-on lab + troubleshooting |
| 04:30–05:00 | Interview Q&A |

---

# 3. Why Storage Is Different in Kubernetes

Containers are designed to be replaceable.

Example:

```text
Pod
 ↓
Container
 ↓
Application writes file
 ↓
Container deleted
 ↓
Container filesystem disappears
```

Therefore:

> Important application data should not normally depend only on the container's writable filesystem.

For persistent data:

```text
Application
    ↓
Pod
    ↓
PVC
    ↓
PV
    ↓
Storage
```

---

# 4. Kubernetes Storage Architecture

High-level:

```text
Application
     ↓
Pod
     ↓
Volume Mount
     ↓
PVC
     ↓
PV
     ↓
StorageClass / CSI
     ↓
Cloud / Storage System
```

For dynamically provisioned storage:

```text
PVC
 ↓
StorageClass
 ↓
CSI Provisioner
 ↓
Storage Backend
 ↓
PV
```

---

# 5. Kubernetes Volumes

A Volume provides storage that can be mounted into a Pod.

Example:

```yaml
volumes:

- name: app-data
  emptyDir: {}
```

Mount:

```yaml
volumeMounts:

- name: app-data
  mountPath: /data
```

Concept:

```text
Pod
 ├── Container
 │     ↓
 │   /data
 │
 └── Volume
```

---

# 6. `emptyDir`

`emptyDir` is created when a Pod is assigned to a node.

Example:

```yaml
volumes:

- name: cache
  emptyDir: {}
```

Mount:

```yaml
volumeMounts:

- name: cache
  mountPath: /cache
```

Lifecycle:

```text
Pod starts
   ↓
emptyDir created
   ↓
Containers use it
   ↓
Pod removed
   ↓
emptyDir removed
```

Use cases:

- Temporary files
- Scratch space
- Shared temporary data between containers
- Build/cache operations

Do not use `emptyDir` for data that must survive Pod replacement.

---

# 7. `emptyDir` with Memory

You can use memory-backed storage:

```yaml
volumes:

- name: cache
  emptyDir:
    medium: Memory
```

This stores data in memory.

Important:

- Consumes node memory.
- Data is still tied to Pod lifetime.
- Should be sized carefully.

---

# 8. HostPath

Example:

```yaml
volumes:

- name: host-data
  hostPath:
    path: /data
```

This maps a node filesystem path into a Pod.

Problems:

- Ties workload to node filesystem.
- Can create security risks.
- Makes scheduling and portability harder.
- Data may not exist on another node.

Avoid `hostPath` for normal application persistence unless the workload genuinely requires host-level access.

---

# 9. PersistentVolume

A PersistentVolume (PV) represents storage available to the cluster.

Concept:

```text
PersistentVolume
       ↓
Actual storage resource
```

A PV can be:

- Dynamically provisioned
- Statically provisioned

---

# 10. PersistentVolumeClaim

A PVC is a request for storage.

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: app-data

spec:

  accessModes:
  - ReadWriteOnce

  resources:

    requests:
      storage: 5Gi
```

Concept:

```text
Application
    ↓
PVC
    ↓
PV
    ↓
Storage
```

The application normally references the PVC rather than directly managing the underlying storage resource.

---

# 11. PV vs PVC

| PV | PVC |
|---|---|
| Storage resource | Storage request |
| Cluster-side resource | User/workload request |
| Represents provisioned storage | Requests capacity/access |
| Bound to a claim | Binds to a suitable PV |

Easy memory:

```text
PV  = Storage
PVC = Request for storage
```

---

# 12. StorageClass

StorageClass defines a class/type of storage and how it can be dynamically provisioned.

Example concept:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: fast-storage

provisioner: <csi-driver>
```

A PVC can request it:

```yaml
spec:

  storageClassName: fast-storage
```

Flow:

```text
PVC
 ↓
StorageClass
 ↓
CSI Driver
 ↓
Storage Backend
 ↓
PV
```

---

# 13. Dynamic Provisioning

Without dynamic provisioning:

```text
Admin
 ↓
Create PV
 ↓
User creates PVC
 ↓
PVC binds to PV
```

With dynamic provisioning:

```text
User creates PVC
       ↓
StorageClass
       ↓
CSI Provisioner
       ↓
Creates storage
       ↓
PV
       ↓
PVC Bound
```

Dynamic provisioning is the normal approach in many cloud Kubernetes environments.

---

# 14. Check StorageClasses

```bash
kubectl get storageclass
```

Detailed:

```bash
kubectl describe storageclass <name>
```

Check the default:

```bash
kubectl get storageclass
```

Look for the default StorageClass marker.

Do not assume the StorageClass name is the same across Minikube, AKS, EKS, or different cluster versions.

---

# 15. Access Modes

Common access modes:

```text
ReadWriteOnce      RWO
ReadOnlyMany       ROX
ReadWriteMany      RWX
ReadWriteOncePod   RWOP
```

### RWO

A volume can be mounted read-write by a single node at a time, depending on the storage implementation.

### ROX

Multiple nodes can mount it read-only, where supported.

### RWX

Multiple nodes can mount it read-write, where supported.

### RWOP

The volume can be mounted read-write by only one Pod at a time, where supported.

Important:

> Access mode support depends on the storage backend/CSI driver.

---

# 16. Access Mode Interview Trap

Do not say:

> RWO means exactly one Pod.

More accurate:

> RWO means the volume can be mounted read-write by a single node. Whether multiple Pods on that node can use it depends on the workload and storage behavior.

For strict single-Pod read-write semantics, RWOP may be appropriate when supported.

---

# 17. Reclaim Policy

PV reclaim policies commonly include:

```text
Retain
Delete
```

### Delete

When the claim is deleted, the dynamically provisioned storage may also be deleted, depending on the provisioner and configuration.

### Retain

The underlying storage is retained for manual recovery/reuse.

For important production data, understand the reclaim policy before deleting PVCs.

---

# 18. Storage Lifecycle

Typical:

```text
PVC Created
    ↓
Provisioning
    ↓
PV Created
    ↓
PVC Bound
    ↓
Pod Mounts Volume
    ↓
Application Uses Data
```

Potential failure:

```text
PVC
 ↓
Pending
 ↓
No suitable StorageClass/Provisioner/Capacity
```

---

# 19. CSI

CSI means:

> Container Storage Interface

CSI provides a standard interface for storage drivers.

Concept:

```text
Kubernetes
    ↓
CSI Driver
    ↓
Storage Platform
```

Examples:

```text
Azure Disk CSI
Azure File CSI
Amazon EBS CSI
Amazon EFS CSI
```

CSI enables Kubernetes to work with different storage systems through drivers.

---

# 20. CSI Architecture

Simplified:

```text
Kubernetes
     ↓
CSI Controller
     ↓
Storage API
     ↓
Cloud Storage

Node
 ↓
CSI Node Plugin
 ↓
Mount Volume
 ↓
Pod
```

There are control-plane and node-side responsibilities in CSI-based storage systems.

---

# 21. PersistentVolumeClaim Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: app-data

spec:

  accessModes:
  - ReadWriteOnce

  resources:

    requests:
      storage: 5Gi
```

Check:

```bash
kubectl get pvc
kubectl describe pvc app-data
```

---

# 22. Pod Using PVC

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: storage-test

spec:

  containers:

  - name: app
    image: nginx:1.27

    volumeMounts:

    - name: data
      mountPath: /data

  volumes:

  - name: data

    persistentVolumeClaim:
      claimName: app-data
```

---

# 23. Verify Persistence

Write data:

```bash
kubectl exec -it storage-test --   sh -c 'echo "hello storage" > /data/test.txt'
```

Delete Pod:

```bash
kubectl delete pod storage-test
```

Create another Pod using the same PVC.

Then:

```bash
kubectl exec -it <new-pod> --   cat /data/test.txt
```

Expected:

```text
hello storage
```

The Pod changed, but the persistent storage remained.

---

# 24. StatefulSet

StatefulSet manages stateful applications.

Examples:

- Databases
- Message brokers
- Distributed systems
- Systems requiring stable identity

Concept:

```text
StatefulSet
    ↓
Pod-0
Pod-1
Pod-2
```

Unlike typical Deployment Pods, StatefulSet Pods have stable ordinal identities.

---

# 25. StatefulSet Pod Names

For:

```yaml
replicas: 3
```

Pods typically become:

```text
database-0
database-1
database-2
```

If:

```text
database-1
```

is deleted, Kubernetes can recreate:

```text
database-1
```

The identity is stable.

---

# 26. StatefulSet vs Deployment

| Deployment | StatefulSet |
|---|---|
| Stateless workloads | Stateful workloads |
| Pods are interchangeable | Pods have stable identity |
| Usually shared application pattern | Often individual persistent storage |
| Random-looking generated names | Stable ordinal names |
| Common web/API workloads | Databases/distributed systems |

Important:

> StatefulSet does not magically make an application distributed or highly available. The application itself must support the required state-management and replication behavior.

---

# 27. StatefulSet Storage

StatefulSets commonly use:

```yaml
volumeClaimTemplates
```

Example:

```yaml
volumeClaimTemplates:

- metadata:
    name: data

  spec:

    accessModes:
    - ReadWriteOnce

    resources:

      requests:
        storage: 10Gi
```

Concept:

```text
StatefulSet
     ↓
Pod-0 → PVC-0 → Storage
Pod-1 → PVC-1 → Storage
Pod-2 → PVC-2 → Storage
```

Each Pod gets its own claim.

---

# 28. Headless Service

A Headless Service has:

```yaml
clusterIP: None
```

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: database

spec:

  clusterIP: None

  selector:
    app: database

  ports:

  - port: 5432
```

Unlike a normal Service, it does not provide a conventional virtual ClusterIP.

It enables DNS records for individual Pod endpoints.

---

# 29. StatefulSet + Headless Service

Typical architecture:

```text
Headless Service
       ↓
DNS
 ┌─────┼─────┐
 ↓     ↓     ↓
pod-0 pod-1 pod-2
 ↓     ↓     ↓
PVC   PVC   PVC
```

This is useful for stateful systems that need stable network identities.

---

# 30. StatefulSet Example

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: web

spec:

  serviceName: web

  replicas: 3

  selector:

    matchLabels:
      app: web

  template:

    metadata:
      labels:
        app: web

    spec:

      containers:

      - name: nginx
        image: nginx:1.27

        volumeMounts:

        - name: data
          mountPath: /data

  volumeClaimTemplates:

  - metadata:
      name: data

    spec:

      accessModes:
      - ReadWriteOnce

      resources:

        requests:
          storage: 1Gi
```

A matching Headless Service named `web` should normally be created separately.

---

# 31. StatefulSet Ordering

StatefulSets can provide ordered creation/deletion/update behavior depending on configuration.

Traditional behavior:

```text
web-0
  ↓
web-1
  ↓
web-2
```

This can be useful for applications where startup ordering matters.

However:

> Do not rely on ordering unless the application actually requires it. Design distributed applications to handle restarts and partial availability.

---

# 32. ConfigMaps

ConfigMaps hold non-sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  APP_ENV: production
  LOG_LEVEL: INFO
```

Consume:

```yaml
envFrom:

- configMapRef:
    name: app-config
```

Or mount as files:

```yaml
volumeMounts:

- name: config
  mountPath: /etc/app
```

---

# 33. Secrets

Secrets are intended for sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: app-secret

type: Opaque

stringData:
  DB_USER: appuser
  DB_PASSWORD: change-me
```

Use:

```yaml
envFrom:

- secretRef:
    name: app-secret
```

Important:

> Kubernetes Secrets are not automatically equivalent to an enterprise secret manager. Configure RBAC and encryption at rest appropriately, and consider external secret-management solutions.

---

# 34. Secrets Are Not Automatically Safe

Do not commit:

```yaml
DB_PASSWORD: real-production-password
```

to Git.

Also avoid:

```dockerfile
ENV DB_PASSWORD=...
```

or:

```bash
kubectl create secret ... --from-literal=password=...
```

in shell history where sensitive values could be exposed.

Use:

- Azure Key Vault
- AWS Secrets Manager
- External Secrets Operator
- Cloud-native workload identity
- Other approved secret-management systems

---

# 35. Secret Delivery Architecture

A production pattern:

```text
Application
    ↓
Kubernetes
    ↓
External Secret Integration
    ↓
Azure Key Vault / AWS Secrets Manager
```

Another approach:

```text
Application
    ↓
Workload Identity
    ↓
Secret Manager
```

This reduces the need to distribute long-lived credentials.

---

# 36. Azure Storage Concepts

In AKS, common storage services include:

```text
Azure Managed Disks
Azure Files
```

Simplified:

### Azure Disk

Usually block storage attached to a node/workload.

Good for workloads needing block storage.

### Azure Files

Managed file share.

Useful when shared file access is required.

The exact access modes and capabilities depend on the CSI driver and storage configuration.

---

# 37. AWS Storage Concepts

In EKS, common storage options include:

```text
Amazon EBS
Amazon EFS
```

### EBS

Block storage.

Common for:

- Databases
- Stateful applications
- Block-device workloads

### EFS

Managed shared file storage.

Useful when multiple workloads need shared file access.

Again, actual Kubernetes access modes depend on the CSI driver and configuration.

---

# 38. Azure vs AWS Storage

| Requirement | Azure | AWS |
|---|---|---|
| Block storage | Managed Disk | EBS |
| Shared file storage | Azure Files | EFS |
| Kubernetes integration | CSI | CSI |
| Dynamic provisioning | StorageClass + CSI | StorageClass + CSI |

Learn the concept rather than memorizing provider-specific YAML.

---

# 39. Backup Is Not the Same as Persistence

Persistence:

```text
Pod deleted
   ↓
Data remains
```

Backup:

```text
Storage damaged/deleted
   ↓
Recover data from backup
```

A persistent volume is not automatically a backup.

Production needs:

```text
Persistence
+
Backup
+
Restore Testing
```

---

# 40. Backup Strategy

Think about:

```text
RPO
RTO
Retention
Encryption
Cross-region recovery
Restore testing
Application consistency
```

### RPO

Recovery Point Objective:

> How much data loss is acceptable?

### RTO

Recovery Time Objective:

> How quickly must the service recover?

---

# 41. Stateful Application Backup

For databases, filesystem snapshots alone may not always guarantee application consistency.

Prefer database-aware backup where required:

```text
Database
   ↓
Consistent Backup
   ↓
Object Storage / Backup System
```

For example:

```text
PostgreSQL
 ↓
Logical/physical backup
 ↓
Object Storage
```

Use the application's supported backup mechanism.

---

# 42. Storage Troubleshooting

## PVC Pending

Check:

```bash
kubectl get pvc
kubectl describe pvc <pvc>
kubectl get storageclass
```

Look for:

- No StorageClass
- Incorrect StorageClass
- CSI provisioner unavailable
- Insufficient capacity
- Access mode mismatch
- Topology constraints

---

# 43. Pod Pending Because PVC Is Pending

Flow:

```text
Pod
 ↓
PVC
 ↓
PVC Pending
 ↓
Pod cannot mount storage
 ↓
Pod remains Pending
```

Start with:

```bash
kubectl describe pvc <pvc>
```

Do not only troubleshoot the Pod.

---

# 44. MountVolume Errors

Check:

```bash
kubectl describe pod <pod>
```

Look at Events.

Potential causes:

- CSI driver problem
- Volume attachment issue
- Wrong mount configuration
- Permission issue
- Node/storage connectivity
- Volume already attached elsewhere
- Storage backend failure

---

# 45. Access Mode Conflict

Example:

```text
PVC: RWO
Pod A → Node 1
Pod B → Node 2
```

The storage backend may not allow the same volume to be mounted read-write from both nodes.

If the application needs shared read/write access, verify whether an RWX-capable storage backend is appropriate.

---

# 46. StatefulSet Troubleshooting

Check:

```bash
kubectl get statefulset
kubectl get pods
kubectl get pvc
kubectl describe statefulset <name>
```

Investigate:

```text
Pod identity
 ↓
PVC
 ↓
StorageClass
 ↓
CSI
 ↓
Node
 ↓
Application
```

---

# 47. Common Storage Mistakes

### Mistake 1

Using `emptyDir` for database data.

### Mistake 2

Assuming a PVC is a backup.

### Mistake 3

Assuming RWO means exactly one Pod.

### Mistake 4

Deleting PVCs without understanding reclaim policy.

### Mistake 5

Hard-coding cloud-specific storage assumptions.

### Mistake 6

Running a stateful application without understanding its replication model.

### Mistake 7

Storing production secrets in Git.

---

# 48. Day 8 Hands-On Lab

## Objective

Build:

```text
StatefulSet
    ↓
Pods
    ↓
PVCs
    ↓
StorageClass
    ↓
Persistent Storage
```

Then practice:

- PVC creation
- Persistent data
- StatefulSet identity
- Headless Service
- Storage troubleshooting

---

# 49. Step 1 — Create Namespace

```bash
kubectl create namespace day8
```

---

# 50. Step 2 — Inspect Storage

```bash
kubectl get storageclass
```

Find the available/default StorageClass.

Do not assume its name.

---

# 51. Step 3 — Create PVC

`pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: app-data
  namespace: day8

spec:

  accessModes:
  - ReadWriteOnce

  resources:

    requests:
      storage: 1Gi
```

Apply:

```bash
kubectl apply -f pvc.yaml
```

Check:

```bash
kubectl get pvc -n day8
```

Expected after successful dynamic provisioning:

```text
STATUS: Bound
```

---

# 52. Step 4 — Create Storage Test Pod

`storage-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: storage-test
  namespace: day8

spec:

  containers:

  - name: app
    image: nginx:1.27

    volumeMounts:

    - name: data
      mountPath: /data

  volumes:

  - name: data

    persistentVolumeClaim:
      claimName: app-data
```

Apply:

```bash
kubectl apply -f storage-pod.yaml
```

Check:

```bash
kubectl get pod -n day8
```

---

# 53. Step 5 — Write Data

```bash
kubectl exec -it storage-test -n day8 --   sh -c 'echo "persistent-data" > /data/test.txt'
```

Read:

```bash
kubectl exec -it storage-test -n day8 --   cat /data/test.txt
```

Expected:

```text
persistent-data
```

---

# 54. Step 6 — Delete Pod

```bash
kubectl delete pod storage-test -n day8
```

Recreate:

```bash
kubectl apply -f storage-pod.yaml
```

Read again:

```bash
kubectl exec -it storage-test -n day8 --   cat /data/test.txt
```

Expected:

```text
persistent-data
```

This demonstrates persistence beyond Pod replacement.

---

# 55. Step 7 — Headless Service

`headless-service.yaml`:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: web
  namespace: day8

spec:

  clusterIP: None

  selector:
    app: web

  ports:

  - port: 80
```

Apply:

```bash
kubectl apply -f headless-service.yaml
```

---

# 56. Step 8 — StatefulSet

`statefulset.yaml`:

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: web
  namespace: day8

spec:

  serviceName: web

  replicas: 3

  selector:

    matchLabels:
      app: web

  template:

    metadata:
      labels:
        app: web

    spec:

      containers:

      - name: nginx
        image: nginx:1.27

        volumeMounts:

        - name: data
          mountPath: /data

  volumeClaimTemplates:

  - metadata:
      name: data

    spec:

      accessModes:
      - ReadWriteOnce

      resources:

        requests:
          storage: 1Gi
```

Apply:

```bash
kubectl apply -f statefulset.yaml
```

---

# 57. Step 9 — Observe StatefulSet

```bash
kubectl get statefulset -n day8
```

Pods:

```bash
kubectl get pods -n day8
```

You should see stable ordinal names such as:

```text
web-0
web-1
web-2
```

PVCs:

```bash
kubectl get pvc -n day8
```

You should see separate claims for the StatefulSet Pods.

---

# 58. Step 10 — Test Individual Identity

Inspect:

```bash
kubectl describe pod web-0 -n day8
kubectl describe pod web-1 -n day8
```

Notice:

```text
web-0 → its PVC
web-1 → its PVC
web-2 → its PVC
```

Concept:

```text
web-0 → data-web-0
web-1 → data-web-1
web-2 → data-web-2
```

Exact PVC naming is generated by Kubernetes from the claim template and StatefulSet identity.

---

# 59. Step 11 — Test Pod Replacement

Delete:

```bash
kubectl delete pod web-1 -n day8
```

Watch:

```bash
kubectl get pods -n day8 -w
```

Observe:

```text
web-1 deleted
     ↓
web-1 recreated
     ↓
Identity remains web-1
     ↓
Its persistent claim remains associated
```

---

# 60. Step 12 — Test DNS

Check:

```bash
kubectl get svc -n day8
```

Use a temporary DNS/debugging Pod if required.

Concept:

```text
web-0
web-1
web-2
   ↓
Headless Service
   ↓
DNS
```

This is important for distributed applications.

---

# 61. Step 13 — Inspect Storage Objects

```bash
kubectl get pv
kubectl get pvc -n day8
kubectl get storageclass
```

Detailed:

```bash
kubectl describe pvc <pvc> -n day8
kubectl describe pv <pv>
```

---

# 62. Step 14 — Intentionally Break a PVC

Create a PVC using a nonexistent StorageClass:

```yaml
storageClassName: does-not-exist
```

Apply:

```bash
kubectl apply -f broken-pvc.yaml
```

Check:

```bash
kubectl get pvc -n day8
kubectl describe pvc <pvc> -n day8
```

Find why it remains:

```text
Pending
```

Fix the StorageClass.

---

# 63. Step 15 — Troubleshoot StatefulSet

Check:

```bash
kubectl get statefulset -n day8
kubectl get pods -n day8
kubectl get pvc -n day8
kubectl describe pod <pod> -n day8
```

Follow:

```text
StatefulSet
 ↓
Pod
 ↓
PVC
 ↓
PV
 ↓
StorageClass
 ↓
CSI
 ↓
Storage backend
```

---

# 64. Storage Troubleshooting Flow

Use this order:

```text
1. kubectl get pvc
       ↓
2. kubectl describe pvc
       ↓
3. kubectl get pv
       ↓
4. kubectl get storageclass
       ↓
5. Check CSI driver
       ↓
6. kubectl describe pod
       ↓
7. Check Events
       ↓
8. Check node/storage backend
```

---

# 65. Day 8 Interview Q&A

## Q1. Why is container storage ephemeral?

> A container's writable filesystem belongs to that container instance. When the container/Pod is replaced, that local data should not be assumed to survive.

## Q2. What is a Kubernetes Volume?

> A Volume provides storage that can be mounted into one or more containers in a Pod according to the volume type.

## Q3. What is `emptyDir`?

> Temporary Pod-scoped storage that is created when the Pod starts on a node and removed when the Pod is removed.

## Q4. What is a PV?

> A PersistentVolume represents persistent storage available to the cluster.

## Q5. What is a PVC?

> A PersistentVolumeClaim is a request for storage with capacity and access-mode requirements.

## Q6. PV vs PVC?

> PV is the storage resource; PVC is the workload's request for that storage.

## Q7. What is a StorageClass?

> It defines a class of storage and typically identifies how storage should be dynamically provisioned.

## Q8. What is dynamic provisioning?

> Kubernetes creates storage/PVs automatically when a PVC requests storage through an appropriate StorageClass and provisioner.

## Q9. What is CSI?

> Container Storage Interface is a standard interface through which storage drivers integrate storage systems with Kubernetes.

## Q10. What is RWO?

> ReadWriteOnce generally allows a volume to be mounted read-write by a single node at a time, depending on the storage implementation.

## Q11. What is RWX?

> ReadWriteMany allows a volume to be mounted read-write by multiple nodes when supported by the storage backend.

## Q12. What is RWOP?

> ReadWriteOncePod restricts read-write mounting to a single Pod when supported by the storage implementation.

## Q13. Does RWO mean only one Pod?

> Not exactly. RWO is a node-level access mode. Multiple Pods on the same node may potentially use the volume depending on the workload and implementation.

## Q14. What is a StatefulSet?

> A controller for stateful workloads that provides stable Pod identity, predictable naming, and commonly persistent storage per replica.

## Q15. StatefulSet vs Deployment?

> Deployment is generally used for interchangeable stateless replicas; StatefulSet is designed for workloads requiring stable identity and/or persistent per-replica storage.

## Q16. What is a Headless Service?

> A Service with `clusterIP: None`, commonly used for direct DNS discovery of individual Pod endpoints.

## Q17. Why does StatefulSet use a Headless Service?

> It provides stable DNS identities for StatefulSet Pods, which is useful for distributed stateful applications.

## Q18. Does StatefulSet provide database replication?

> No. It provides orchestration primitives such as stable identity and storage. Database replication must be implemented by the database/application.

## Q19. What happens when a StatefulSet Pod is deleted?

> The controller recreates the Pod with the same ordinal identity and normally reconnects it to its associated persistent claim.

## Q20. Is a PVC a backup?

> No. A PVC provides persistent storage; backup is a separate data-protection mechanism.

## Q21. What causes a PVC to remain Pending?

> Possible causes include missing/incorrect StorageClass, unavailable provisioner, unsupported access mode, insufficient capacity, topology constraints, or storage backend problems.

## Q22. How do you troubleshoot PVC Pending?

> Start with `kubectl describe pvc`, inspect Events, StorageClass, PVs, CSI components, and the storage backend.

## Q23. What is a reclaim policy?

> It defines what happens to the PV/storage after its claim is released. Common policies include Retain and Delete.

## Q24. Why can deleting a PVC be dangerous?

> Depending on the reclaim policy and provisioner, deleting a PVC may lead to deletion of the associated persistent storage.

## Q25. Azure Disk vs Azure Files?

> Azure Disk is block storage commonly used for workloads needing block-device semantics; Azure Files is managed file storage useful for shared file access.

## Q26. EBS vs EFS?

> EBS is block storage; EFS is shared managed file storage.

## Q27. What is volume attachment?

> It is the process of making a persistent storage volume available to the appropriate node before it can be mounted into a Pod, depending on the storage architecture.

## Q28. What is a CSI driver?

> A plugin implementing the CSI interface so Kubernetes can provision, attach, mount, and manage storage from a specific storage platform.

## Q29. Why shouldn't databases simply use Deployments?

> A database may need stable identity, per-replica storage, ordered operations, and application-specific replication behavior, which StatefulSet primitives can support better.

## Q30. How would you design production storage?

> Choose storage based on workload requirements, access mode, performance, availability, backup/restore objectives, encryption, topology, failure domains, cost, and operational support.

---

# 66. Senior-Level Scenarios

## Scenario 1 — PVC Is Pending

Architecture:

```text
PVC
 ↓
StorageClass
 ↓
CSI Provisioner
 ↓
Storage Backend
```

Check each layer.

Commands:

```bash
kubectl describe pvc <pvc>
kubectl get storageclass
kubectl get pv
```

Then inspect CSI components and events.

---

## Scenario 2 — Pod Cannot Mount Volume

Flow:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
CSI
 ↓
Node
 ↓
Storage
```

Check:

```bash
kubectl describe pod <pod>
kubectl describe pvc <pvc>
kubectl describe pv <pv>
```

Look at Events.

---

## Scenario 3 — Database Pod Moved to Another Node

Ask:

```text
What storage backend?
What access mode?
Can volume attach to new node?
Are topology constraints satisfied?
Is CSI healthy?
```

For block storage, volume attachment may be limited by backend topology or node attachment rules.

---

## Scenario 4 — Application Requires RWX

Requirement:

```text
Pod A ─┐
Pod B ─┼──→ Shared filesystem
Pod C ─┘
```

Do not assume RWO will work.

Select a storage backend/CSI driver that supports the required shared access model, such as a suitable file-storage system.

---

## Scenario 5 — Production Database Lost Data

Do not immediately assume Kubernetes caused the issue.

Investigate:

```text
Application
 ↓
PVC
 ↓
PV
 ↓
Storage backend
 ↓
Backup
 ↓
Restore
```

Determine:

- What data was lost?
- Was the PVC deleted?
- What was the reclaim policy?
- Were backups available?
- When was the last successful backup?
- Was restore tested?
- What is the RPO/RTO?

---

## Scenario 6 — StatefulSet Has 3 Replicas but Application Is Not Highly Available

Important distinction:

```text
3 Pods
   ≠
3 independent healthy database replicas
```

The database/application must support:

- Replication
- Leader election
- Failure handling
- Data consistency
- Recovery
- Client routing

Kubernetes provides infrastructure primitives; it does not automatically make the application stateful and highly available.

---

# 67. Production Storage Design Checklist

Before deploying a stateful workload, answer:

### Storage

- [ ] What type of storage is required?
- [ ] Block or file?
- [ ] RWO, RWX, or RWOP?
- [ ] Required capacity?
- [ ] IOPS/throughput?
- [ ] Latency requirement?
- [ ] Encryption?

### Availability

- [ ] What happens if a node fails?
- [ ] What happens if a zone fails?
- [ ] Does the storage support required topology?
- [ ] Is the application replicated?

### Backup

- [ ] What is the RPO?
- [ ] What is the RTO?
- [ ] How often are backups taken?
- [ ] Where are backups stored?
- [ ] Are backups encrypted?
- [ ] Has restore been tested?

### Security

- [ ] RBAC?
- [ ] Encryption?
- [ ] Secret management?
- [ ] NetworkPolicy?
- [ ] Least privilege?

### Operations

- [ ] Monitoring?
- [ ] Storage capacity alerts?
- [ ] CSI health monitoring?
- [ ] Backup monitoring?
- [ ] Restore runbook?

---

# 68. Day 8 Checklist

- [ ] I understand ephemeral container storage.
- [ ] I understand Kubernetes Volumes.
- [ ] I understand `emptyDir`.
- [ ] I understand `hostPath` risks.
- [ ] I understand PV.
- [ ] I understand PVC.
- [ ] I understand StorageClass.
- [ ] I understand dynamic provisioning.
- [ ] I understand CSI.
- [ ] I understand access modes.
- [ ] I understand RWO.
- [ ] I understand RWX.
- [ ] I understand RWOP.
- [ ] I understand reclaim policies.
- [ ] I understand StatefulSets.
- [ ] I understand stable Pod identity.
- [ ] I understand Headless Services.
- [ ] I understand `volumeClaimTemplates`.
- [ ] I understand ConfigMaps.
- [ ] I understand Secrets.
- [ ] I understand external secret management.
- [ ] I understand Azure Disk vs Azure Files.
- [ ] I understand EBS vs EFS.
- [ ] I understand persistence vs backup.
- [ ] I understand RPO/RTO.
- [ ] I completed the PVC lab.
- [ ] I completed the StatefulSet lab.
- [ ] I tested persistent data.
- [ ] I troubleshot a Pending PVC.
- [ ] I answered at least 25 interview questions aloud.

---

# 69. Homework

- [ ] Create a PVC.
- [ ] Mount it into a Pod.
- [ ] Write data.
- [ ] Delete and recreate the Pod.
- [ ] Verify the data remains.
- [ ] Create a Headless Service.
- [ ] Deploy a StatefulSet with 3 replicas.
- [ ] Inspect its PVCs.
- [ ] Delete `pod-1` and observe recreation.
- [ ] Intentionally create a broken StorageClass reference.
- [ ] Troubleshoot the Pending PVC.
- [ ] Compare Azure Disk/Azure Files.
- [ ] Compare AWS EBS/EFS.
- [ ] Draw a production database storage architecture.
- [ ] Explain why persistence does not equal backup.

---

# 70. Final Day 8 Challenge

Without looking at your notes, explain:

```text
                    Application
                         ↓
                        Pod
                         ↓
                       PVC
                         ↓
                        PV
                         ↓
                  StorageClass / CSI
                         ↓
              Cloud Storage Backend
```

Then explain:

1. Why is container storage ephemeral?
2. What is `emptyDir`?
3. PV vs PVC?
4. What is StorageClass?
5. What is dynamic provisioning?
6. What does CSI do?
7. RWO vs RWX vs RWOP?
8. What is a reclaim policy?
9. Why can deleting a PVC be dangerous?
10. What is a StatefulSet?
11. Deployment vs StatefulSet?
12. Why does a StatefulSet need stable identity?
13. What is a Headless Service?
14. How does `volumeClaimTemplates` work?
15. Does StatefulSet automatically provide database replication?
16. How do you troubleshoot a Pending PVC?
17. How do you troubleshoot a volume mount failure?
18. Azure Disk vs Azure Files?
19. EBS vs EFS?
20. Persistence vs backup?
21. What are RPO and RTO?
22. How would you design storage for PostgreSQL?
23. How would you design storage for a shared file-processing application?
24. How would you handle a zone failure?
25. How would you prove that backups are actually recoverable?

---

# 71. Success Criteria

You are ready for **Day 9 — Kubernetes Security & RBAC** when you can confidently explain:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
StorageClass
 ↓
CSI
 ↓
Cloud Storage
```

and:

```text
StatefulSet
   ↓
Stable Pod Identity
   ↓
Persistent Per-Pod Storage
   ↓
Headless Service
   ↓
Stable Network Identity
```

You should also be able to troubleshoot a storage problem systematically:

```text
PVC
 ↓
PV
 ↓
StorageClass
 ↓
CSI
 ↓
Node
 ↓
Storage Backend
```

---

# Next Day

## Day 9 — Kubernetes Security & RBAC

Topics:

- Kubernetes security model
- Authentication vs Authorization
- RBAC
- Roles
- ClusterRoles
- RoleBindings
- ClusterRoleBindings
- ServiceAccounts
- Least privilege
- SecurityContext
- Pod Security Standards
- NetworkPolicies
- Secrets security
- Workload Identity
- Azure Managed Identity / Workload Identity
- AWS IAM Roles for Service Accounts
- Image security
- Admission controls
- Security troubleshooting
- Complete RBAC practical lab
- 30+ security interview questions
