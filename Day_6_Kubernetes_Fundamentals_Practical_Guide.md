# Day 6 — Kubernetes Fundamentals

> **30-Day DevOps → MLOps Study Plan**  
> **Focus:** Kubernetes architecture, control plane, worker nodes, Pods, ReplicaSets, Deployments, Namespaces, Labels, Services, ConfigMaps, Secrets, Requests/Limits, Probes, kubectl, Minikube practical lab, troubleshooting, and senior-level interview preparation  
> **Recommended time:** 4 hours  
> **Target level:** ~6 years DevOps experience

---

# 1. Day 6 Objectives

By the end of Day 6, you should be able to:

- Explain Kubernetes architecture.
- Explain control plane and worker node components.
- Understand API Server, etcd, Scheduler, Controller Manager, Kubelet and container runtime.
- Explain Pods and why Pods are the basic workload unit.
- Create Deployments and understand ReplicaSets.
- Understand Namespaces, Labels and Selectors.
- Create and expose Services.
- Understand ClusterIP, NodePort and LoadBalancer.
- Use ConfigMaps and Secrets.
- Understand resource requests and limits.
- Configure liveness, readiness and startup probes.
- Use common `kubectl` commands.
- Deploy an application using Minikube.
- Troubleshoot common Kubernetes failures.
- Answer senior-level Kubernetes interview questions.

---

# 2. Recommended 4-Hour Schedule

| Time | Duration | Topic |
|---|---:|---|
| 00:00–00:40 | 40 min | Kubernetes architecture |
| 00:40–01:20 | 40 min | Pods, ReplicaSets, Deployments |
| 01:20–02:00 | 40 min | Namespaces, Labels, Services |
| 02:00–02:35 | 35 min | ConfigMaps, Secrets, Resources |
| 02:35–03:05 | 30 min | Probes and application health |
| 03:05–03:40 | 35 min | kubectl practical |
| 03:40–04:20 | 40 min | Minikube lab + troubleshooting |

---

# 3. What Is Kubernetes?

Kubernetes is a container orchestration platform.

Docker primarily helps you:

```text
Build
 ↓
Package
 ↓
Run
```

Kubernetes helps you:

```text
Schedule
 ↓
Deploy
 ↓
Scale
 ↓
Heal
 ↓
Expose
 ↓
Roll out
 ↓
Manage containers
```

Typical architecture:

```text
Developer
    ↓
Container Image
    ↓
Registry
    ↓
Kubernetes
    ↓
Pods
    ↓
Containers
```

---

# 4. Why Kubernetes?

Without orchestration:

```text
Server 1
 ├── Container A
 ├── Container B

Server 2
 ├── Container C

Server 3
 ├── Container D
```

You need to manually handle:

- Scheduling
- Restarts
- Scaling
- Service discovery
- Rollouts
- Networking

Kubernetes automates these operations.

---

# 5. Kubernetes Architecture

High-level:

```text
                 Control Plane
        ┌────────────────────────────┐
        │ API Server                 │
        │ Scheduler                  │
        │ Controller Manager         │
        │ etcd                       │
        └──────────────┬─────────────┘
                       │
             Kubernetes API
                       │
       ┌───────────────┼───────────────┐
       ↓               ↓               ↓
   Worker Node     Worker Node     Worker Node
       │               │               │
    Kubelet         Kubelet         Kubelet
       │               │               │
  Containers       Containers       Containers
```

---

# 6. Control Plane

The control plane manages the Kubernetes cluster.

Main components:

```text
API Server
Scheduler
Controller Manager
etcd
```

---

# 7. API Server

The API Server is the main entry point into Kubernetes.

Commands such as:

```bash
kubectl get pods
```

communicate with the Kubernetes API.

Concept:

```text
kubectl
   ↓
API Server
   ↓
Kubernetes Objects
```

The API Server also handles:

- Authentication
- Authorization
- Admission
- API validation
- API requests

---

# 8. etcd

`etcd` is the distributed key-value store used by Kubernetes for cluster state.

Concept:

```text
API Server
    ↓
etcd
    ↓
Cluster State
```

It stores information about Kubernetes objects and cluster configuration/state.

Important:

> Protect etcd carefully. Losing or corrupting etcd can severely affect the cluster's ability to recover its state.

---

# 9. Scheduler

The Scheduler decides which node should run a newly scheduled Pod.

Concept:

```text
Pod Pending
    ↓
Scheduler
    ↓
Evaluate Nodes
    ↓
Select Suitable Node
    ↓
Pod Assigned
```

It considers factors such as:

- Resource availability
- Node constraints
- Taints and tolerations
- Affinity/anti-affinity
- Topology requirements
- Scheduling policies

---

# 10. Controller Manager

Controllers continuously compare:

```text
Desired State
      vs
Current State
```

Example:

Desired:

```text
replicas: 3
```

Current:

```text
2 Pods
```

Controller:

```text
Detect difference
      ↓
Create another Pod
```

This is one of the core Kubernetes principles:

> Kubernetes continuously works to move the actual state toward the desired state.

---

# 11. Worker Node

A worker node runs workloads.

Important components:

```text
Kubelet
Container Runtime
Networking Components
Pods
```

Concept:

```text
Worker Node
│
├── Kubelet
├── Container Runtime
├── Network
└── Pods
```

---

# 12. Kubelet

Kubelet is the node-level agent.

It:

- Receives workload instructions through the Kubernetes control plane.
- Ensures assigned Pods are running.
- Reports node/Pod status.
- Works with the container runtime.

Concept:

```text
API Server
    ↓
Kubelet
    ↓
Container Runtime
    ↓
Container
```

---

# 13. Container Runtime

Kubernetes needs a runtime capable of running containers.

Modern Kubernetes commonly uses runtimes such as:

```text
containerd
CRI-O
```

The runtime is responsible for running containers on the node.

---

# 14. Pod

A Pod is the smallest deployable unit in Kubernetes.

A Pod can contain:

```text
Pod
├── Container
└── Container
```

Containers in the same Pod share:

- Network namespace
- Pod IP
- Volumes that are mounted into them

Most applications use one main application container per Pod.

---

# 15. Why Pods Instead of Containers?

Kubernetes does not directly schedule a Docker container as its basic unit.

It schedules a Pod.

```text
Deployment
     ↓
ReplicaSet
     ↓
Pod
     ↓
Container
```

A Pod represents a tightly coupled group of one or more containers.

---

# 16. Create a Pod

Example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-pod

spec:
  containers:

  - name: nginx
    image: nginx:1.27
    ports:
    - containerPort: 80
```

Apply:

```bash
kubectl apply -f pod.yaml
```

Check:

```bash
kubectl get pods
```

---

# 17. Pod Lifecycle

Typical lifecycle:

```text
Pending
   ↓
Running
   ↓
Succeeded / Failed
```

A Pod can also be restarted depending on its restart policy and container state.

Check:

```bash
kubectl get pod
kubectl describe pod <pod-name>
```

---

# 18. Why We Usually Don't Create Pods Directly

You can create a Pod directly, but production workloads usually use controllers such as:

```text
Deployment
StatefulSet
DaemonSet
Job
CronJob
```

For stateless applications, Deployment is usually the starting point.

---

# 19. Deployment

A Deployment manages a set of Pods for a stateless application.

Concept:

```text
Deployment
     ↓
ReplicaSet
     ↓
Pods
     ↓
Containers
```

Example:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx

spec:

  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:

    metadata:
      labels:
        app: nginx

    spec:

      containers:

      - name: nginx
        image: nginx:1.27
        ports:
        - containerPort: 80
```

---

# 20. ReplicaSet

ReplicaSet maintains the requested number of matching Pods.

Desired:

```text
replicas: 3
```

Current:

```text
Pod 1
Pod 2
Pod 3
```

If Pod 2 dies:

```text
Pod 1
Pod 2 ❌
Pod 3
```

ReplicaSet creates another:

```text
Pod 1
Pod 4
Pod 3
```

---

# 21. Deployment vs ReplicaSet

| Deployment | ReplicaSet |
|---|---|
| Higher-level controller | Lower-level controller |
| Manages application rollout | Maintains Pod replicas |
| Supports rolling updates | Maintains replica count |
| Supports rollback | Usually managed by Deployment |

Typical hierarchy:

```text
Deployment
     ↓
ReplicaSet
     ↓
Pods
```

---

# 22. Scale a Deployment

```bash
kubectl scale deployment nginx --replicas=5
```

Check:

```bash
kubectl get deployment
kubectl get pods
```

Expected:

```text
5 Pods
```

---

# 23. Update a Deployment

Change image:

```bash
kubectl set image deployment/nginx \
  nginx=nginx:1.28
```

Check rollout:

```bash
kubectl rollout status deployment/nginx
```

History:

```bash
kubectl rollout history deployment/nginx
```

Rollback:

```bash
kubectl rollout undo deployment/nginx
```

---

# 24. Rolling Update

Kubernetes can gradually replace old Pods.

```text
Old Version
Pod 1
Pod 2
Pod 3

       ↓

New Version
Pod 1 → New
Pod 2 → New
Pod 3 → New
```

This reduces downtime compared with replacing everything at once.

---

# 25. Namespaces

Namespaces logically separate resources within a cluster.

Example:

```text
Cluster
│
├── dev
├── qa
├── staging
└── production
```

Create:

```bash
kubectl create namespace dev
```

Use:

```bash
kubectl get pods -n dev
```

---

# 26. Why Use Namespaces?

Namespaces help with:

- Organization
- Resource isolation
- RBAC
- Resource quotas
- Environment separation

Important:

> Namespaces are logical isolation boundaries, not a replacement for all security isolation mechanisms.

---

# 27. Labels

Labels attach metadata to Kubernetes objects.

Example:

```yaml
metadata:
  labels:
    app: nginx
    environment: dev
```

Labels are heavily used for:

- Selection
- Service routing
- Organization
- Queries

---

# 28. Selectors

A selector finds objects matching labels.

Example:

```yaml
selector:
  matchLabels:
    app: nginx
```

If Pods have:

```yaml
labels:
  app: nginx
```

the Deployment manages them.

---

# 29. Label/Selector Mismatch

A common problem:

Deployment:

```yaml
selector:
  matchLabels:
    app: nginx
```

Pod template:

```yaml
labels:
  app: web
```

This mismatch can break controller behavior.

Always verify:

```text
Selector
   =
Pod Labels
```

---

# 30. Services

Pods are ephemeral.

Their IP addresses can change.

A Service provides a stable network endpoint.

```text
Clients
   ↓
Service
   ↓
Pods
```

---

# 31. ClusterIP

Default Service type:

```text
ClusterIP
```

Example:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:

  selector:
    app: nginx

  ports:

  - port: 80
    targetPort: 80

  type: ClusterIP
```

This exposes the application inside the cluster.

---

# 32. NodePort

NodePort exposes a Service through a port on cluster nodes.

```yaml
type: NodePort
```

Concept:

```text
Client
  ↓
Node IP:NodePort
  ↓
Service
  ↓
Pod
```

NodePort is useful for labs and some specific use cases, but production internet exposure often uses a LoadBalancer or Ingress/Gateway architecture.

---

# 33. LoadBalancer

Cloud Kubernetes platforms can integrate Services with cloud load balancers.

```yaml
type: LoadBalancer
```

Concept:

```text
Internet
   ↓
Cloud Load Balancer
   ↓
Service
   ↓
Pods
```

---

# 34. Service Selector

Example:

```yaml
selector:
  app: nginx
```

The Service routes traffic to matching Pods.

Check:

```bash
kubectl get endpoints
```

or, on newer Kubernetes versions:

```bash
kubectl get endpointslices
```

---

# 35. ConfigMap

ConfigMaps store non-sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  APP_ENV: "dev"
  LOG_LEVEL: "INFO"
```

Use as environment variables:

```yaml
envFrom:
- configMapRef:
    name: app-config
```

---

# 36. Secret

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

> Kubernetes Secret objects are not automatically equivalent to a full enterprise secret-management system. Configure encryption at rest and RBAC appropriately, and consider external secret managers for sensitive production environments.

---

# 37. ConfigMap vs Secret

| ConfigMap | Secret |
|---|---|
| Non-sensitive config | Sensitive config |
| URLs/settings | Passwords/tokens |
| Log levels | Credentials |
| Feature flags | Certificates/keys |

Neither should be used carelessly.

---

# 38. Resource Requests

Requests tell Kubernetes the amount of resource a container needs for scheduling.

Example:

```yaml
resources:

  requests:
    cpu: "250m"
    memory: "256Mi"
```

Concept:

```text
Scheduler
   ↓
Find node with enough requested resources
```

---

# 39. Resource Limits

Limits define a maximum resource boundary.

```yaml
resources:

  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

Meaning:

```text
Request → Scheduling requirement
Limit   → Runtime ceiling
```

---

# 40. Why Requests and Limits Matter

Without resource controls:

```text
Bad Pod
   ↓
Consumes excessive CPU/memory
   ↓
Impacts other workloads
```

With appropriate limits:

```text
Pod
 ↓
Controlled Resource Usage
```

However, limits must be selected based on workload behavior rather than blindly copied.

---

# 41. Liveness Probe

Liveness asks:

> Is the container still healthy enough to continue running?

Example:

```yaml
livenessProbe:

  httpGet:
    path: /health
    port: 8080

  initialDelaySeconds: 10
  periodSeconds: 10
```

If liveness repeatedly fails, Kubernetes may restart the container.

---

# 42. Readiness Probe

Readiness asks:

> Is this Pod ready to receive traffic?

Example:

```yaml
readinessProbe:

  httpGet:
    path: /ready
    port: 8080

  initialDelaySeconds: 5
  periodSeconds: 5
```

If readiness fails:

```text
Pod remains running
       ↓
Removed from Service endpoints
```

This is different from liveness.

---

# 43. Startup Probe

Startup probes are useful for slow-starting applications.

```yaml
startupProbe:

  httpGet:
    path: /health
    port: 8080

  failureThreshold: 30
  periodSeconds: 10
```

Concept:

```text
Application Starting
       ↓
Startup Probe
       ↓
Application Ready
       ↓
Liveness / Readiness
```

---

# 44. Probe Comparison

| Probe | Question |
|---|---|
| Startup | Has the application finished starting? |
| Liveness | Should the container be restarted? |
| Readiness | Should traffic be sent to this Pod? |

Remember:

```text
Startup → Start
Liveness → Restart
Readiness → Traffic
```

---

# 45. kubectl Basics

Check cluster:

```bash
kubectl cluster-info
```

Nodes:

```bash
kubectl get nodes
```

Pods:

```bash
kubectl get pods
```

Detailed Pods:

```bash
kubectl get pods -o wide
```

All namespaces:

```bash
kubectl get pods -A
```

Deployments:

```bash
kubectl get deployments
```

Services:

```bash
kubectl get svc
```

---

# 46. kubectl Describe

Use:

```bash
kubectl describe pod <pod>
```

This is one of the most useful troubleshooting commands.

Look for:

```text
Events
Container State
Image
Mounts
Probes
Environment
Scheduling
```

---

# 47. Logs

```bash
kubectl logs <pod>
```

Follow:

```bash
kubectl logs -f <pod>
```

For a specific container:

```bash
kubectl logs <pod> -c <container>
```

Previous crashed container:

```bash
kubectl logs <pod> --previous
```

---

# 48. Execute Inside Pod

```bash
kubectl exec -it <pod> -- /bin/sh
```

If multiple containers:

```bash
kubectl exec -it <pod> -c <container> -- /bin/sh
```

Use this for controlled troubleshooting, not as the primary operational model.

---

# 49. YAML Workflow

Typical workflow:

```text
Write YAML
   ↓
kubectl apply
   ↓
kubectl get
   ↓
kubectl describe
   ↓
kubectl logs
   ↓
Test
```

Example:

```bash
kubectl apply -f deployment.yaml
kubectl get deployment
kubectl get pods
kubectl describe pod <pod>
```

---

# 50. Minikube

Minikube creates a local Kubernetes cluster for learning.

Start:

```bash
minikube start
```

Check:

```bash
minikube status
kubectl get nodes
```

Dashboard:

```bash
minikube dashboard
```

Check profile:

```bash
minikube profile list
```

---

# 51. Day 6 Hands-On Lab

## Objective

Deploy a complete application:

```text
Deployment
   ↓
Pods
   ↓
Service
   ↓
ConfigMap
   ↓
Secret
   ↓
Probes
   ↓
Resource Requests/Limits
```

---

# 52. Step 1 — Start Minikube

```bash
minikube start
```

Verify:

```bash
kubectl get nodes
```

Expected concept:

```text
NAME       STATUS   ROLE
minikube   Ready    control-plane
```

---

# 53. Step 2 — Create Namespace

```bash
kubectl create namespace day6
```

Verify:

```bash
kubectl get namespaces
```

---

# 54. Step 3 — Create ConfigMap

`configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config
  namespace: day6

data:
  APP_ENV: "dev"
  LOG_LEVEL: "INFO"
```

Apply:

```bash
kubectl apply -f configmap.yaml
```

---

# 55. Step 4 — Create Secret

`secret.yaml`:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: app-secret
  namespace: day6

type: Opaque

stringData:
  DB_USER: appuser
  DB_PASSWORD: change-me
```

Apply:

```bash
kubectl apply -f secret.yaml
```

For real production credentials, use a proper secret-management strategy rather than committing this example value to Git.

---

# 56. Step 5 — Create Deployment

`deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx
  namespace: day6

spec:

  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:

    metadata:
      labels:
        app: nginx

    spec:

      containers:

      - name: nginx
        image: nginx:1.27

        ports:
        - containerPort: 80

        envFrom:

        - configMapRef:
            name: app-config

        - secretRef:
            name: app-secret

        resources:

          requests:
            cpu: "100m"
            memory: "128Mi"

          limits:
            cpu: "250m"
            memory: "256Mi"

        readinessProbe:

          httpGet:
            path: /
            port: 80

          initialDelaySeconds: 5
          periodSeconds: 10

        livenessProbe:

          httpGet:
            path: /
            port: 80

          initialDelaySeconds: 10
          periodSeconds: 10
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

---

# 57. Step 6 — Verify Deployment

```bash
kubectl get deployment -n day6
```

Check ReplicaSets:

```bash
kubectl get rs -n day6
```

Check Pods:

```bash
kubectl get pods -n day6
```

Detailed:

```bash
kubectl get pods -n day6 -o wide
```

---

# 58. Step 7 — Create Service

`service.yaml`:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx
  namespace: day6

spec:

  selector:
    app: nginx

  ports:

  - port: 80
    targetPort: 80

  type: NodePort
```

Apply:

```bash
kubectl apply -f service.yaml
```

Check:

```bash
kubectl get svc -n day6
```

---

# 59. Step 8 — Access Application

With Minikube:

```bash
minikube service nginx -n day6
```

Or inspect:

```bash
kubectl get svc -n day6
```

You can also use:

```bash
minikube ip
```

and the assigned NodePort where appropriate.

---

# 60. Step 9 — Scale

```bash
kubectl scale deployment nginx \
  --replicas=5 \
  -n day6
```

Check:

```bash
kubectl get pods -n day6
```

---

# 61. Step 10 — Rolling Update

Change image:

```bash
kubectl set image deployment/nginx \
  nginx=nginx:1.28 \
  -n day6
```

Check:

```bash
kubectl rollout status deployment/nginx -n day6
```

History:

```bash
kubectl rollout history deployment/nginx -n day6
```

---

# 62. Step 11 — Rollback

```bash
kubectl rollout undo deployment/nginx -n day6
```

Verify:

```bash
kubectl rollout status deployment/nginx -n day6
```

---

# 63. Step 12 — Troubleshooting Practice

Delete a Pod:

```bash
kubectl delete pod <pod-name> -n day6
```

Watch:

```bash
kubectl get pods -n day6 -w
```

Observe:

```text
Pod deleted
   ↓
ReplicaSet notices
   ↓
Replacement Pod created
```

This demonstrates Kubernetes reconciliation.

---

# 64. Troubleshooting Scenario — ImagePullBackOff

Check:

```bash
kubectl get pods -n day6
kubectl describe pod <pod> -n day6
```

Look at Events.

Common causes:

- Wrong image name
- Wrong tag
- Registry authentication failure
- Private registry access problem
- Network problem

---

# 65. Troubleshooting Scenario — CrashLoopBackOff

Check:

```bash
kubectl logs <pod> -n day6
kubectl logs <pod> --previous -n day6
kubectl describe pod <pod> -n day6
```

Possible causes:

- Application crash
- Invalid configuration
- Missing secret
- Bad command
- Dependency unavailable
- Probe failure

---

# 66. Troubleshooting Scenario — Pod Pending

Check:

```bash
kubectl describe pod <pod> -n day6
```

Common causes:

- Insufficient resources
- Node selector mismatch
- Taints/tolerations
- PVC not available
- Scheduling constraints

---

# 67. Troubleshooting Scenario — Service Has No Traffic

Check:

```bash
kubectl get svc -n day6
kubectl get endpoints -n day6
kubectl get endpointslices -n day6
kubectl get pods --show-labels -n day6
```

Common cause:

```text
Service selector
       ≠
Pod labels
```

Also check:

```text
port
targetPort
container listening port
readiness probe
```

---

# 68. Troubleshooting Flow

Use this order:

```text
1. kubectl get
       ↓
2. kubectl describe
       ↓
3. kubectl logs
       ↓
4. Check events
       ↓
5. Check configuration
       ↓
6. Check networking
       ↓
7. Check resources
       ↓
8. Check dependencies
```

Avoid randomly changing YAML before understanding the failure.

---

# 69. Kubernetes Desired State

Kubernetes is declarative.

You describe:

```yaml
replicas: 3
```

Kubernetes continuously tries to make reality:

```text
Current State = Desired State
```

Example:

```text
Desired: 3
Current: 2
       ↓
Controller
       ↓
Create Pod
       ↓
Current: 3
```

---

# 70. Declarative vs Imperative

### Declarative

```bash
kubectl apply -f deployment.yaml
```

You define the desired state.

### Imperative

```bash
kubectl create deployment nginx --image=nginx
```

You issue a direct command.

For production configuration, declarative YAML and GitOps are generally easier to review, version, and reproduce.

---

# 71. Kubernetes Security Basics

Important controls:

```text
RBAC
 ↓
Namespaces
 ↓
Network Policies
 ↓
Pod Security
 ↓
Secrets
 ↓
Non-root Containers
 ↓
Resource Limits
 ↓
Image Scanning
```

Do not assume that putting something in a namespace automatically makes it secure.

---

# 72. ResourceQuota

A namespace can have a resource quota.

Example:

```yaml
apiVersion: v1
kind: ResourceQuota

metadata:
  name: day6-quota
  namespace: day6

spec:

  hard:
    pods: "10"
    requests.cpu: "2"
    requests.memory: 2Gi
```

Concept:

```text
Namespace
   ↓
ResourceQuota
   ↓
Limit total consumption
```

---

# 73. LimitRange

LimitRange can define default/min/max resource settings.

Example:

```yaml
apiVersion: v1
kind: LimitRange

metadata:
  name: defaults
  namespace: day6

spec:

  limits:

  - type: Container

    default:
      cpu: "500m"
      memory: "512Mi"

    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
```

This is useful for enforcing sensible resource configuration.

---

# 74. Kubernetes Interview Q&A

## Q1. What is Kubernetes?

> Kubernetes is a container orchestration platform that automates deployment, scheduling, scaling, networking, and lifecycle management of containerized workloads.

## Q2. What is the control plane?

> The control plane manages cluster state and scheduling through components such as API Server, Scheduler, Controller Manager, and etcd.

## Q3. What is the API Server?

> It is the central API endpoint through which clients and cluster components interact with Kubernetes.

## Q4. What is etcd?

> etcd is the distributed key-value store that holds Kubernetes cluster state.

## Q5. What does the Scheduler do?

> It selects a suitable node for a Pod based on resource and scheduling constraints.

## Q6. What does the Controller Manager do?

> It runs controllers that continuously reconcile current state with desired state.

## Q7. What is Kubelet?

> Kubelet is the node agent responsible for ensuring Pods assigned to the node are running and reporting their status.

## Q8. What is a Pod?

> A Pod is Kubernetes' smallest deployable unit and can contain one or more tightly coupled containers sharing networking and selected storage.

## Q9. Why use a Deployment?

> A Deployment manages stateless Pods through ReplicaSets and provides controlled rolling updates and rollback.

## Q10. Deployment vs ReplicaSet?

> A Deployment is the higher-level controller responsible for rollout management; ReplicaSet maintains the desired number of matching Pods.

## Q11. Why are Pods ephemeral?

> Pods are replaceable workload instances. Kubernetes controllers create replacements when Pods disappear or become unsuitable.

## Q12. What is a Service?

> A Service provides a stable network endpoint for a dynamic set of Pods.

## Q13. ClusterIP vs NodePort vs LoadBalancer?

> ClusterIP is internal cluster access, NodePort exposes a port on nodes, and LoadBalancer integrates with a cloud/external load-balancing mechanism where supported.

## Q14. What is a Namespace?

> A Namespace logically organizes and scopes Kubernetes resources and can be used with RBAC, quotas, and policies.

## Q15. What are Labels?

> Labels are key-value metadata used to organize objects and select groups of resources.

## Q16. What is a Selector?

> A selector identifies objects matching specific labels.

## Q17. ConfigMap vs Secret?

> ConfigMap is intended for non-sensitive configuration; Secret is intended for sensitive values. Production Secret handling should include appropriate RBAC and encryption/secret-management controls.

## Q18. What is a resource request?

> A request is the resource amount Kubernetes uses primarily for scheduling decisions.

## Q19. What is a resource limit?

> A limit defines an upper runtime boundary for a container's resource usage.

## Q20. Liveness vs readiness?

> Liveness determines whether a container should be restarted. Readiness determines whether the Pod should receive traffic.

## Q21. What is a startup probe?

> It gives slow-starting applications time to initialize before liveness/readiness checks become active.

## Q22. What is CrashLoopBackOff?

> It indicates that a container is repeatedly failing and Kubernetes is backing off between restart attempts.

## Q23. What is ImagePullBackOff?

> Kubernetes cannot currently pull the specified image and is backing off before retrying.

## Q24. How do you troubleshoot Pending Pods?

> Start with `kubectl describe pod` and inspect Events, resource availability, taints/tolerations, selectors, affinity, and other scheduling constraints.

## Q25. How do you troubleshoot a Service?

> Verify Service selector, Pod labels, endpoints/EndpointSlices, ports, targetPort, readiness, and whether the application is listening correctly.

## Q26. What is reconciliation?

> Controllers continuously compare desired state with actual state and take actions to reduce the difference.

## Q27. Why use GitOps?

> GitOps stores desired cluster configuration in Git, enabling version control, review, auditability, and automated reconciliation.

## Q28. What is a rolling update?

> A rolling update gradually replaces old Pods with new Pods while maintaining application availability according to configured rollout constraints.

## Q29. How do you secure a Kubernetes workload?

> Use RBAC, least privilege, network policies, Pod security controls, non-root containers, image scanning, resource controls, secret management, and restricted service accounts.

## Q30. How would you troubleshoot a production Pod?

> Check workload state, events, logs, previous logs, probes, resources, configuration, networking, dependencies, image/version, and recent deployment changes before taking corrective action.

---

# 75. Senior-Level Scenarios

## Scenario 1 — Pod Is Running but Application Is Unreachable

Investigate:

```text
Pod Running?
 ↓
Readiness?
 ↓
Service Selector?
 ↓
Endpoints?
 ↓
Service Port?
 ↓
TargetPort?
 ↓
Application Listening?
 ↓
Network Policy?
```

A `Running` Pod does not automatically mean the application is reachable.

---

## Scenario 2 — Pod Keeps Restarting

Check:

```bash
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

Investigate:

- Application crash
- OOMKilled
- Bad configuration
- Secret/config issue
- Liveness failure
- Dependency failure

---

## Scenario 3 — Deployment Stuck During Rollout

Check:

```bash
kubectl rollout status deployment/<name>
kubectl describe deployment/<name>
kubectl get rs
kubectl get pods
```

Look for:

- Failed scheduling
- Image pull failure
- Readiness failure
- Resource constraints
- Invalid configuration

---

## Scenario 4 — Pod OOMKilled

Check:

```bash
kubectl describe pod <pod>
```

If memory usage exceeds the container's configured limit, the container may be killed.

Investigate:

```text
Application memory behavior
 ↓
Request
 ↓
Limit
 ↓
Node capacity
```

Do not simply increase limits without understanding why memory is growing.

---

## Scenario 5 — Service Selector Is Wrong

Deployment:

```yaml
labels:
  app: backend
```

Service:

```yaml
selector:
  app: api
```

Result:

```text
Service
  ↓
No matching Pods
  ↓
No endpoints
```

Fix the selector/label mismatch.

---

## Scenario 6 — Production Pod Runs as Root

Improve:

```yaml
securityContext:
  runAsNonRoot: true
```

Also consider:

```yaml
allowPrivilegeEscalation: false
readOnlyRootFilesystem: true
capabilities:
  drop:
  - ALL
```

Only use settings that are compatible with the application. Some workloads require specific filesystem or capability access.

---

# 76. Day 6 Practical Checklist

- [ ] I understand Kubernetes architecture.
- [ ] I understand the control plane.
- [ ] I understand worker nodes.
- [ ] I understand API Server.
- [ ] I understand etcd.
- [ ] I understand Scheduler.
- [ ] I understand Controller Manager.
- [ ] I understand Kubelet.
- [ ] I understand container runtime.
- [ ] I understand Pods.
- [ ] I understand Deployments.
- [ ] I understand ReplicaSets.
- [ ] I understand rolling updates.
- [ ] I understand rollbacks.
- [ ] I understand Namespaces.
- [ ] I understand Labels and Selectors.
- [ ] I understand Services.
- [ ] I understand ClusterIP.
- [ ] I understand NodePort.
- [ ] I understand LoadBalancer.
- [ ] I understand ConfigMaps.
- [ ] I understand Secrets.
- [ ] I understand requests and limits.
- [ ] I understand liveness/readiness/startup probes.
- [ ] I can use kubectl.
- [ ] I completed the Minikube lab.
- [ ] I can troubleshoot CrashLoopBackOff.
- [ ] I can troubleshoot ImagePullBackOff.
- [ ] I can troubleshoot Pending Pods.
- [ ] I can troubleshoot Services.
- [ ] I answered at least 25 interview questions aloud.

---

# 77. Homework

- [ ] Create a Minikube cluster.
- [ ] Create a namespace.
- [ ] Deploy nginx with 3 replicas.
- [ ] Expose it using a Service.
- [ ] Add ConfigMap.
- [ ] Add Secret.
- [ ] Add resource requests/limits.
- [ ] Add liveness/readiness probes.
- [ ] Scale to 5 replicas.
- [ ] Perform a rolling update.
- [ ] Perform a rollback.
- [ ] Delete one Pod and observe reconciliation.
- [ ] Intentionally use a wrong image and troubleshoot ImagePullBackOff.
- [ ] Intentionally break a Service selector and troubleshoot it.
- [ ] Practice `kubectl describe`, `kubectl logs`, and `kubectl get`.
- [ ] Explain Kubernetes architecture without notes.

---

# 78. Final Day 6 Challenge

Without looking at your notes, explain this:

```text
                    Control Plane
              ┌─────────────────────┐
              │ API Server           │
              │ Scheduler            │
              │ Controller Manager   │
              │ etcd                 │
              └──────────┬──────────┘
                         │
                 Kubernetes API
                         │
       ┌─────────────────┼─────────────────┐
       ↓                 ↓                 ↓
   Worker Node       Worker Node       Worker Node
       │                 │                 │
    Kubelet           Kubelet           Kubelet
       │                 │                 │
      Pods              Pods              Pods
       │                 │                 │
 Containers          Containers        Containers
```

Then explain:

1. What happens when you run `kubectl apply`?
2. How does a Deployment create Pods?
3. What does a ReplicaSet do?
4. What happens if a Pod dies?
5. How does the Scheduler choose a node?
6. What does etcd store?
7. Why do Pods need Services?
8. Difference between ClusterIP, NodePort and LoadBalancer?
9. ConfigMap vs Secret?
10. Requests vs limits?
11. Liveness vs readiness?
12. What does a startup probe solve?
13. How do you troubleshoot CrashLoopBackOff?
14. How do you troubleshoot ImagePullBackOff?
15. How do you troubleshoot a Service with no endpoints?
16. How does Kubernetes reconciliation work?
17. Why are containers normally run as non-root?
18. How would you secure a production namespace?
19. How would you design Kubernetes for 50 microservices?
20. How would you safely roll out a new production version?

---

# 79. Success Criteria

You are ready for **Day 7 — Kubernetes Networking & Ingress** when you can explain:

```text
kubectl
  ↓
API Server
  ↓
Scheduler / Controllers
  ↓
Deployment
  ↓
ReplicaSet
  ↓
Pod
  ↓
Container
  ↓
Service
  ↓
Traffic
```

You should also be able to deploy, scale, update, rollback, and troubleshoot a basic Kubernetes application using Minikube.

---

# Next Day

## Day 7 — Kubernetes Networking & Ingress

Topics:

- Kubernetes networking model
- Pod-to-Pod communication
- Services and DNS
- ClusterIP internals
- NodePort
- LoadBalancer
- Ingress
- Ingress Controller
- NGINX Ingress
- Gateway API overview
- CoreDNS
- Network Policies
- Service discovery
- Internal vs external traffic
- Minikube networking lab
- Ingress troubleshooting
- 25+ Kubernetes networking interview questions
