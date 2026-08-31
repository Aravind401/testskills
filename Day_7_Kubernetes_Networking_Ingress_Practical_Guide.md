# Day 7 — Kubernetes Networking & Ingress

> **30-Day DevOps → MLOps Study Plan**  
> **Focus:** Kubernetes networking, Services, DNS/CoreDNS, CNI, Ingress, Ingress Controllers, Gateway API, NetworkPolicies, Minikube lab, troubleshooting, and senior-level interview preparation  
> **Recommended time:** 4 hours  
> **Target:** ~6 years DevOps experience

---

# 1. Objectives

By the end of Day 7, you should be able to:

- Explain the Kubernetes networking model.
- Explain Pod-to-Pod communication.
- Understand Pod IPs and why Pods are ephemeral.
- Understand Services and service discovery.
- Understand ClusterIP, NodePort, and LoadBalancer.
- Understand `port`, `targetPort`, and `nodePort`.
- Understand Kubernetes DNS and CoreDNS.
- Understand CNI at a practical level.
- Understand Ingress and Ingress Controllers.
- Understand host/path-based routing.
- Understand TLS termination.
- Understand Gateway API at a high level.
- Understand NetworkPolicies.
- Troubleshoot DNS, Services, Ingress, and NetworkPolicy issues.
- Complete a Minikube networking lab.
- Answer advanced Kubernetes networking interview questions.

---

# 2. Schedule

| Time | Topic |
|---|---|
| 00:00–00:40 | Kubernetes networking model |
| 00:40–01:20 | Services and service discovery |
| 01:20–02:00 | DNS, CoreDNS, ports |
| 02:00–02:40 | Ingress and Ingress Controllers |
| 02:40–03:10 | NetworkPolicies |
| 03:10–03:50 | Minikube networking lab |
| 03:50–04:30 | Troubleshooting + interview Q&A |

---

# 3. Kubernetes Networking Model

A simplified Kubernetes networking model expects:

1. Every Pod receives an IP address.
2. Pods can communicate with other Pods through the cluster networking implementation.
3. Nodes can communicate with Pods.
4. Services provide stable virtual endpoints for dynamic Pods.

Concept:

```text
Pod A
10.x.x.10
   |
   | Pod Network
   ↓
Pod B
10.x.x.20
```

The exact implementation depends on the cluster's CNI.

---

# 4. What Is CNI?

CNI means **Container Network Interface**.

CNI plugins implement Pod networking.

Examples:

- Cilium
- Calico
- Azure CNI
- Amazon VPC CNI
- Flannel

Concept:

```text
Kubernetes
    ↓
CNI
    ↓
Pod Networking
```

The CNI provides/configures network connectivity for Pods and may also implement network policy capabilities.

---

# 5. Pod IP

Example:

```text
frontend Pod
10.244.1.10

backend Pod
10.244.2.15
```

Do not use Pod IPs as permanent application endpoints.

Bad:

```text
Frontend
   ↓
10.244.2.15
   ↓
Backend Pod
```

If the backend Pod is replaced:

```text
Old Pod ❌
10.244.2.15

New Pod
10.244.3.20
```

The client would have stale information.

Use a Service.

---

# 6. Service

A Service provides a stable virtual endpoint for a set of Pods.

```text
Client
   ↓
Service
   ↓
Pods
```

Example:

```text
backend-service
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
Pod A Pod B Pod C
```

Pods can change while the Service remains stable.

---

# 7. Service Selector

```yaml
apiVersion: v1
kind: Service

metadata:
  name: backend

spec:
  selector:
    app: backend

  ports:
  - port: 80
    targetPort: 8080
```

The Service selects Pods with:

```yaml
labels:
  app: backend
```

Flow:

```text
Service selector
      ↓
Matching Pod labels
      ↓
EndpointSlices
      ↓
Traffic
```

---

# 8. EndpointSlices

EndpointSlices represent the network endpoints associated with Services.

Check:

```bash
kubectl get endpointslices
```

Namespace:

```bash
kubectl get endpointslices -n day7
```

Concept:

```text
Service
   ↓
EndpointSlices
   ↓
Pod IPs
```

If a Service has no usable endpoints, traffic cannot reach a backend through that Service.

---

# 9. ClusterIP

ClusterIP is the default Service type.

It exposes a Service internally within the cluster.

```yaml
apiVersion: v1
kind: Service

metadata:
  name: backend

spec:
  type: ClusterIP

  selector:
    app: backend

  ports:
  - port: 80
    targetPort: 8080
```

Flow:

```text
Frontend Pod
      ↓
backend:80
      ↓
ClusterIP
      ↓
Backend Pods
```

---

# 10. NodePort

NodePort exposes a Service through a port on cluster nodes.

```yaml
spec:
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
Pods
```

Example:

```text
Node:30080
    ↓
Service:80
    ↓
Pod:8080
```

Useful for labs and specific architectures.

---

# 11. LoadBalancer

Cloud Kubernetes environments can integrate `LoadBalancer` Services with external/cloud load balancers.

```yaml
spec:
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

Examples:

```text
AKS → Azure load-balancing integration
EKS → AWS load-balancing integration
```

Exact behavior depends on cloud provider and cluster configuration.

---

# 12. Service Type Comparison

| Type | Main Use |
|---|---|
| ClusterIP | Internal cluster communication |
| NodePort | Exposure through node ports |
| LoadBalancer | External/cloud load-balancer exposure |

Easy memory:

```text
ClusterIP → Inside cluster
NodePort → Node port
LoadBalancer → External/cloud LB
```

---

# 13. `port` vs `targetPort` vs `nodePort`

Example:

```yaml
ports:
- port: 80
  targetPort: 8080
  nodePort: 30080
```

Flow:

```text
Client
  ↓
Node:30080
  ↓
Service:80
  ↓
Pod:8080
```

Remember:

```text
nodePort   → Node exposure
port       → Service port
targetPort → Backend Pod port
```

---

# 14. Kubernetes DNS

Kubernetes provides DNS-based service discovery.

For a Service named:

```text
backend
```

inside the same namespace, applications can usually use:

```text
backend
```

Other forms:

```text
backend.day7
```

Fully qualified:

```text
backend.day7.svc.cluster.local
```

Use DNS names instead of hard-coded Service IPs.

---

# 15. CoreDNS

CoreDNS commonly provides Kubernetes DNS service discovery.

Concept:

```text
Pod
 ↓
DNS Query
 ↓
CoreDNS
 ↓
Service DNS
 ↓
Service Address
```

Check:

```bash
kubectl get pods -n kube-system
```

Look for CoreDNS Pods.

---

# 16. DNS Troubleshooting

Check Services:

```bash
kubectl get svc -n day7
```

Check CoreDNS:

```bash
kubectl get pods -n kube-system
```

Test from a Pod:

```bash
kubectl exec -it <pod> -n day7 -- nslookup backend
```

If `nslookup` is unavailable, use a temporary network-debugging Pod/image.

Check CoreDNS logs:

```bash
kubectl logs -n kube-system <coredns-pod>
```

---

# 17. Service Discovery Flow

```text
Frontend
   ↓
http://backend:80
   ↓
CoreDNS
   ↓
Service
   ↓
EndpointSlices
   ↓
Backend Pod
```

This is one of the most important Kubernetes networking patterns.

---

# 18. kube-proxy

In many Kubernetes networking setups, kube-proxy programs node-level networking rules for Services.

Conceptually:

```text
Service
   ↓
Service Rules
   ↓
Backend Pod
```

Traditional implementations may use mechanisms such as iptables or IPVS.

Modern eBPF-based networking implementations can provide Service handling without kube-proxy.

Interview answer:

> kube-proxy traditionally implements Kubernetes Service virtual-IP behavior on nodes. Its exact role depends on the networking implementation.

---

# 19. Ingress

Ingress is a Kubernetes API resource for HTTP/HTTPS routing to Services.

```text
Internet
   ↓
Ingress
   ↓
Service
   ↓
Pods
```

It can route based on:

- Hostname
- URL path

Example:

```text
api.example.com
      ↓
backend-service

www.example.com
      ↓
frontend-service
```

---

# 20. Why Use Ingress?

Without centralized HTTP routing:

```text
App A → LoadBalancer
App B → LoadBalancer
App C → LoadBalancer
```

With an ingress layer:

```text
                Load Balancer
                     ↓
              Ingress Controller
              /       |                    ↓        ↓        ↓
        Frontend    Backend     API
```

One external entry point can route to multiple Services.

---

# 21. Ingress Resource

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: app-ingress
  namespace: day7

spec:
  rules:

  - host: app.example.com

    http:

      paths:

      - path: /
        pathType: Prefix

        backend:

          service:
            name: frontend
            port:
              number: 80
```

Important:

> An Ingress object alone does not necessarily provide a working HTTP endpoint. An Ingress Controller must implement the resource.

---

# 22. Ingress Controller

The Ingress resource describes desired routing.

The Ingress Controller implements it.

```text
Ingress YAML
     ↓
Ingress Controller
     ↓
Proxy / Load Balancer
     ↓
Service
     ↓
Pods
```

Examples include:

- NGINX-based controllers
- Traefik
- HAProxy
- Cloud-provider ingress/load-balancer controllers

---

# 23. Host-Based Routing

```text
api.example.com
      ↓
backend-service

web.example.com
      ↓
frontend-service
```

Example:

```yaml
rules:

- host: api.example.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: backend
          port:
            number: 80

- host: web.example.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: frontend
          port:
            number: 80
```

---

# 24. Path-Based Routing

Example:

```text
example.com/
      ↓
frontend

example.com/api
      ↓
backend
```

Ingress:

```yaml
rules:

- host: example.com

  http:

    paths:

    - path: /api
      pathType: Prefix

      backend:
        service:
          name: backend
          port:
            number: 80

    - path: /
      pathType: Prefix

      backend:
        service:
          name: frontend
          port:
            number: 80
```

Path rewriting behavior is controller-specific. Do not assume `/api` is automatically removed before reaching the backend.

---

# 25. TLS with Ingress

Ingress can reference a TLS Secret.

```yaml
tls:

- hosts:
  - app.example.com

  secretName: app-tls
```

Concept:

```text
Client
  ↓ HTTPS
Ingress
  ↓
Service
  ↓
Pods
```

TLS termination can occur at different layers depending on the architecture.

---

# 26. Ingress vs Service

Service:

```text
Stable endpoint
 ↓
Routes to Pods
```

Ingress:

```text
HTTP/HTTPS routing
 ↓
Host/path rules
 ↓
Services
```

Simplified:

```text
Service → Expose application
Ingress → Route web traffic
```

---

# 27. Ingress vs Gateway API

Gateway API is a newer Kubernetes networking API family designed for a more expressive and extensible traffic-management model.

Simplified:

```text
Ingress
 ↓
Basic HTTP routing
```

Gateway API:

```text
Gateway
 ↓
HTTPRoute / other Route types
 ↓
More expressive routing model
```

Exact features depend on the implementation/controller.

---

# 28. Gateway API Architecture

Concept:

```text
GatewayClass
      ↓
Gateway
      ↓
HTTPRoute
      ↓
Service
      ↓
Pods
```

Learn this at a conceptual level first.

---

# 29. NetworkPolicy

NetworkPolicy controls allowed network traffic to/from selected Pods when supported by the cluster networking implementation.

Without policy:

```text
Pod A → Pod B
Pod C → Pod B
Pod D → Pod B
```

With policy:

```text
Pod A ─────→ Pod B
Pod C ──X──→ Pod B
Pod D ──X──→ Pod B
```

---

# 30. Default-Deny Pattern

A common security approach:

```text
Namespace
   ↓
Default Deny
   ↓
Explicitly Allow Required Traffic
```

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: default-deny
  namespace: day7

spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

Be careful: default-deny egress can also block DNS and external dependencies until appropriate allow rules are added.

---

# 31. Allow Frontend → Backend

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: allow-frontend-to-backend
  namespace: day7

spec:

  podSelector:
    matchLabels:
      app: backend

  policyTypes:
  - Ingress

  ingress:

  - from:

    - podSelector:
        matchLabels:
          app: frontend

    ports:

    - protocol: TCP
      port: 8080
```

Concept:

```text
Frontend
   ↓ TCP/8080
Backend

Other Pods
   X
Backend
```

---

# 32. NetworkPolicy Caveat

NetworkPolicy behavior depends on the CNI/network implementation.

Before production use:

- Confirm required policy support.
- Test ingress and egress.
- Test DNS.
- Test application dependencies.
- Test monitoring/observability traffic.
- Test external API access.

---

# 33. North-South vs East-West Traffic

### North-South

Traffic entering or leaving the cluster:

```text
Internet
   ↓
Cluster
```

### East-West

Traffic between internal workloads:

```text
Frontend
   ↔
Backend
   ↔
Database
```

Typical controls:

```text
North-South
↓
Load Balancer / Ingress / Gateway

East-West
↓
Service / NetworkPolicy / Service Mesh
```

---

# 34. Complete Networking Architecture

```text
                    Internet
                       ↓
                Cloud Load Balancer
                       ↓
                 Ingress / Gateway
                  /                            ↓             ↓
          frontend-service   api-service
                 ↓             ↓
             Frontend Pods   Backend Pods
                                ↓
                           db-service
                                ↓
                             Database
```

NetworkPolicies can restrict:

```text
Frontend → Backend
Backend → Database
```

---

# 35. Day 7 Hands-On Lab

## Objective

Build:

```text
Client
  ↓
Ingress
  ↓
Frontend Service
  ↓
Frontend Pods

Backend Service
  ↓
Backend Pods
```

Then practice:

- Service DNS
- ClusterIP
- NodePort
- Ingress
- EndpointSlices
- NetworkPolicy
- Troubleshooting

---

# 36. Step 1 — Start Minikube

```bash
minikube start
```

Verify:

```bash
kubectl get nodes
```

---

# 37. Step 2 — Enable Ingress

```bash
minikube addons enable ingress
```

Check:

```bash
kubectl get pods -A
```

On many Minikube setups, the controller appears under:

```bash
kubectl get pods -n ingress-nginx
```

---

# 38. Step 3 — Namespace

```bash
kubectl create namespace day7
```

---

# 39. Step 4 — Frontend Deployment

`frontend.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: frontend
  namespace: day7

spec:

  replicas: 2

  selector:
    matchLabels:
      app: frontend

  template:

    metadata:
      labels:
        app: frontend

    spec:

      containers:

      - name: frontend
        image: nginx:1.27

        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f frontend.yaml
```

---

# 40. Step 5 — Frontend Service

`frontend-service.yaml`:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: frontend
  namespace: day7

spec:

  selector:
    app: frontend

  ports:

  - port: 80
    targetPort: 80

  type: ClusterIP
```

Apply:

```bash
kubectl apply -f frontend-service.yaml
```

---

# 41. Step 6 — Backend Deployment

`backend.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: backend
  namespace: day7

spec:

  replicas: 2

  selector:
    matchLabels:
      app: backend

  template:

    metadata:
      labels:
        app: backend

    spec:

      containers:

      - name: backend
        image: nginx:1.27

        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f backend.yaml
```

---

# 42. Step 7 — Backend Service

`backend-service.yaml`:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: backend
  namespace: day7

spec:

  selector:
    app: backend

  ports:

  - port: 80
    targetPort: 80

  type: ClusterIP
```

Apply:

```bash
kubectl apply -f backend-service.yaml
```

---

# 43. Step 8 — Test Service DNS

Get Pods:

```bash
kubectl get pods -n day7
```

Enter a Pod:

```bash
kubectl exec -it <frontend-pod> -n day7 -- /bin/sh
```

Inside, test the Service:

```bash
wget -qO- http://backend
```

If the application image lacks debugging tools, use a temporary network-debugging Pod.

Concept:

```text
Frontend Pod
    ↓
backend DNS name
    ↓
backend Service
    ↓
Backend Pods
```

---

# 44. Step 9 — Inspect EndpointSlices

```bash
kubectl get endpointslices -n day7
```

Also:

```bash
kubectl get endpoints -n day7
```

The selected backend Pods should appear as endpoints.

---

# 45. Step 10 — Create Ingress

`ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: app-ingress
  namespace: day7

spec:

  rules:

  - host: day7.local

    http:

      paths:

      - path: /
        pathType: Prefix

        backend:

          service:
            name: frontend
            port:
              number: 80

      - path: /api
        pathType: Prefix

        backend:

          service:
            name: backend
            port:
              number: 80
```

Apply:

```bash
kubectl apply -f ingress.yaml
```

Check:

```bash
kubectl get ingress -n day7
```

---

# 46. Step 11 — Access Minikube Ingress

Get the Minikube IP:

```bash
minikube ip
```

For local testing, map:

```text
day7.local
```

to the Minikube IP in your local hosts configuration.

Then:

```bash
curl http://day7.local/
```

and:

```bash
curl http://day7.local/api
```

The exact access method can vary with the local Minikube/Ingress setup.

---

# 47. Step 12 — Inspect Ingress

```bash
kubectl describe ingress app-ingress -n day7
```

Check:

```text
Rules
Backends
Events
Address
```

---

# 48. Step 13 — Test NodePort

Change a Service to:

```yaml
type: NodePort
```

Check:

```bash
kubectl get svc -n day7
```

Then:

```bash
minikube service frontend -n day7
```

Observe:

```text
NodePort
   ↓
Service
   ↓
Pod
```

---

# 49. Step 14 — NetworkPolicy

Create a backend ingress restriction:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: backend-default-deny
  namespace: day7

spec:

  podSelector:
    matchLabels:
      app: backend

  policyTypes:
  - Ingress
```

Apply:

```bash
kubectl apply -f backend-policy.yaml
```

Then create an explicit allow policy for frontend traffic.

---

# 50. Step 15 — Allow Frontend → Backend

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: allow-frontend-backend
  namespace: day7

spec:

  podSelector:
    matchLabels:
      app: backend

  policyTypes:
  - Ingress

  ingress:

  - from:

    - podSelector:
        matchLabels:
          app: frontend

    ports:

    - protocol: TCP
      port: 80
```

Apply:

```bash
kubectl apply -f allow-frontend-backend.yaml
```

---

# 51. Step 16 — Verify NetworkPolicy

Expected concept:

```text
Frontend → Backend
      ✓

Other Pod → Backend
      ✗
```

This depends on the CNI enforcing NetworkPolicy.

---

# 52. Troubleshooting — Service Has No Endpoints

Run:

```bash
kubectl get svc -n day7
kubectl get endpointslices -n day7
kubectl get pods --show-labels -n day7
```

Check:

```text
Service selector
       ↓
Pod labels
```

Example:

Service:

```yaml
selector:
  app: backend
```

Pod:

```yaml
labels:
  app: api
```

Result:

```text
No matching Pods
       ↓
No endpoints
```

---

# 53. Troubleshooting — DNS Failure

Check:

```bash
kubectl get svc -n day7
kubectl get pods -n kube-system
```

Test:

```bash
kubectl exec -it <pod> -n day7 -- nslookup backend
```

Check CoreDNS logs:

```bash
kubectl logs -n kube-system <coredns-pod>
```

---

# 54. Troubleshooting — Ingress 404

Check:

```bash
kubectl get ingress -n day7
kubectl describe ingress app-ingress -n day7
kubectl get svc -n day7
kubectl get endpointslices -n day7
```

Common causes:

- Host header mismatch
- Wrong path
- Wrong Service name
- Wrong Service port
- No backend endpoints
- Ingress Controller issue

---

# 55. Troubleshooting — Ingress Has No Address

Check:

```bash
kubectl get ingress -n day7
kubectl get pods -A
```

Then identify the Ingress Controller.

For Minikube, often:

```bash
kubectl get pods -n ingress-nginx
```

Also:

```bash
kubectl describe ingress app-ingress -n day7
```

---

# 56. Troubleshooting — Connection Refused

Check:

```text
Pod running?
      ↓
Application listening?
      ↓
Correct container port?
      ↓
Service targetPort?
      ↓
EndpointSlices?
      ↓
Readiness?
      ↓
NetworkPolicy?
```

A Service can exist while the application behind it is not accepting traffic.

---

# 57. Troubleshooting Flow

Use:

```text
1. kubectl get
        ↓
2. kubectl describe
        ↓
3. kubectl logs
        ↓
4. Check events
        ↓
5. Check endpoints
        ↓
6. Check DNS
        ↓
7. Check NetworkPolicy
        ↓
8. Check Ingress Controller
```

---

# 58. Common Networking Mistakes

### Mistake 1 — Pod IPs

Bad:

```text
Frontend → Pod IP
```

Better:

```text
Frontend → Service DNS
```

### Mistake 2 — localhost

Bad:

```text
Backend → localhost:5432
```

when the database is another Pod.

Better:

```text
database:5432
```

### Mistake 3 — Wrong `targetPort`

### Mistake 4 — Selector/label mismatch

### Mistake 5 — NetworkPolicy blocks required traffic

### Mistake 6 — Ingress exists but no Controller implements it

---

# 59. Production Networking Architecture

```text
                    Internet
                       ↓
                Cloud Load Balancer
                       ↓
                  Gateway / Ingress
                  /                              ↓               ↓
          frontend-service    api-service
                 ↓               ↓
          Frontend Pods      Backend Pods
                                ↓
                           db-service
                                ↓
                             Database
```

Security:

```text
NetworkPolicy
      ↓
Limit East-West Traffic
```

Observability:

```text
Logs
Metrics
Traces
      ↓
Monitoring
```

---

# 60. Azure vs AWS Kubernetes Networking

Azure:

```text
Internet
   ↓
Azure Load Balancer / Application Gateway
   ↓
AKS
   ↓
Services / Ingress / Gateway
   ↓
Pods
```

AWS:

```text
Internet
   ↓
AWS Load Balancer
   ↓
EKS
   ↓
Services / Ingress / Gateway
   ↓
Pods
```

The exact design depends on the cloud networking mode and controllers selected.

---

# 61. Interview Q&A

## Q1. What is Kubernetes networking?

> It provides connectivity between Pods, Nodes, Services, and external clients. The underlying Pod network is implemented by the cluster's networking/CNI solution.

## Q2. What is CNI?

> Container Network Interface is the standard interface used to configure container networking. CNI plugins implement the actual networking behavior.

## Q3. Why don't we use Pod IPs directly?

> Pod IPs can change when Pods are replaced. Services provide stable endpoints.

## Q4. What is a Service?

> A Service provides a stable virtual endpoint and routes traffic to a selected set of Pods.

## Q5. ClusterIP vs NodePort vs LoadBalancer?

> ClusterIP is internal, NodePort exposes through node ports, and LoadBalancer integrates with an external/cloud load balancer where supported.

## Q6. What is `targetPort`?

> The port on the selected backend Pod that receives Service traffic.

## Q7. What is CoreDNS?

> CoreDNS commonly provides Kubernetes DNS-based service discovery.

## Q8. How does a Pod resolve another Service?

> The Pod sends a DNS query to the cluster DNS service, commonly CoreDNS, which resolves the Service name.

## Q9. What is an EndpointSlice?

> It represents network endpoints associated with a Service and provides a scalable endpoint-tracking mechanism.

## Q10. What is Ingress?

> Ingress is a Kubernetes API resource for HTTP/HTTPS routing to Services based on host and path rules.

## Q11. Is Ingress itself a load balancer?

> The Ingress resource is configuration. An Ingress Controller implements the actual routing and may integrate with a load balancer or proxy.

## Q12. What is an Ingress Controller?

> It watches Ingress resources and configures the actual traffic-routing mechanism.

## Q13. Why is Ingress useful?

> It centralizes HTTP/HTTPS routing and can expose multiple applications through shared entry points.

## Q14. What is Gateway API?

> Gateway API is a newer Kubernetes networking API family with a more expressive and extensible model for traffic management.

## Q15. What is NetworkPolicy?

> It defines allowed ingress and/or egress traffic for selected Pods, provided the cluster networking implementation supports it.

## Q16. What is east-west traffic?

> Traffic between internal workloads or services.

## Q17. What is north-south traffic?

> Traffic entering or leaving the cluster.

## Q18. Why does a Service have no endpoints?

> Usually because its selector does not match Pod labels, or selected Pods are not eligible endpoints.

## Q19. How do you troubleshoot DNS?

> Check the Service, CoreDNS Pods/logs, DNS configuration, and perform a lookup from a test Pod.

## Q20. How do you troubleshoot an Ingress 404?

> Verify host/path rules, Ingress Controller, Service name/port, EndpointSlices, and the request Host/path.

## Q21. How do you troubleshoot connection refused?

> Verify the application listening port/address, container port, Service targetPort, endpoints, readiness, NetworkPolicy, and ingress/load-balancer configuration.

## Q22. Can Pods on different nodes communicate?

> Yes, assuming the cluster networking implementation correctly provides Pod-to-Pod connectivity.

## Q23. Why use Service DNS?

> It decouples clients from changing Pod IPs and provides stable service discovery.

## Q24. How do NetworkPolicies improve security?

> They restrict unnecessary east-west communication and allow explicit communication paths.

## Q25. Most common Kubernetes networking mistake?

> Incorrect selectors, ports, DNS assumptions, or network policies. Use a systematic Pods → Services → EndpointSlices → DNS → Policy → Ingress troubleshooting flow.

---

# 62. Senior-Level Scenarios

## Scenario 1 — Service Exists but Application Is Down

```text
Service
  ↓
EndpointSlices?
  ↓
Pod Ready?
  ↓
Application Listening?
```

A Service existing does not guarantee that the backend is healthy.

---

## Scenario 2 — Ingress `/` Works but `/api` Fails

Check:

```text
Path rule
 ↓
PathType
 ↓
Backend Service
 ↓
Service port
 ↓
Endpoint
```

Also check controller-specific path rewriting.

---

## Scenario 3 — DNS Works for One Service but Not Another

Compare:

```text
Working
 ↓
Namespace
 ↓
Service name
 ↓
Endpoints

Broken
 ↓
Namespace
 ↓
Service name
 ↓
Endpoints
```

Commands:

```bash
kubectl get svc -A
kubectl get endpointslices -A
```

---

## Scenario 4 — NetworkPolicy Breaks the Application

Check:

```text
NetworkPolicy
 ↓
Ingress rules
 ↓
Egress rules
 ↓
DNS
 ↓
Database/API dependencies
```

Default-deny policies require explicit rules for required traffic.

---

## Scenario 5 — NodePort Works but Ingress Does Not

If:

```text
NodePort → Works
```

but:

```text
Ingress → Fails
```

focus on:

```text
Ingress Resource
 ↓
Ingress Controller
 ↓
Host/Path
 ↓
Controller Logs
 ↓
External/Local Routing
```

---

## Scenario 6 — 100 Microservices

Avoid exposing every Service individually.

Prefer:

```text
Internet
   ↓
Central Gateway / Ingress
   ↓
Services
   ↓
Microservices
```

Use:

- Standardized routing
- TLS
- Authentication/authorization
- NetworkPolicies
- Observability
- Rate limiting where appropriate
- Clear service ownership

---

# 63. Day 7 Checklist

- [ ] I understand the Kubernetes networking model.
- [ ] I understand CNI.
- [ ] I understand Pod IPs.
- [ ] I understand why Pod IPs are ephemeral.
- [ ] I understand Services.
- [ ] I understand Service selectors.
- [ ] I understand EndpointSlices.
- [ ] I understand ClusterIP.
- [ ] I understand NodePort.
- [ ] I understand LoadBalancer.
- [ ] I understand `port`.
- [ ] I understand `targetPort`.
- [ ] I understand `nodePort`.
- [ ] I understand Kubernetes DNS.
- [ ] I understand CoreDNS.
- [ ] I understand kube-proxy at a high level.
- [ ] I understand Ingress.
- [ ] I understand Ingress Controllers.
- [ ] I understand host-based routing.
- [ ] I understand path-based routing.
- [ ] I understand TLS with Ingress.
- [ ] I understand Gateway API at a high level.
- [ ] I understand NetworkPolicy.
- [ ] I understand east-west traffic.
- [ ] I understand north-south traffic.
- [ ] I completed the Minikube networking lab.
- [ ] I can troubleshoot Service failures.
- [ ] I can troubleshoot DNS.
- [ ] I can troubleshoot Ingress.
- [ ] I can troubleshoot NetworkPolicy issues.
- [ ] I answered at least 25 interview questions aloud.

---

# 64. Homework

- [ ] Create a Minikube cluster.
- [ ] Deploy frontend and backend.
- [ ] Create ClusterIP Services.
- [ ] Test Service DNS.
- [ ] Inspect EndpointSlices.
- [ ] Create a NodePort Service.
- [ ] Enable Minikube Ingress.
- [ ] Create host/path-based Ingress.
- [ ] Test `/` and `/api`.
- [ ] Break a Service selector intentionally.
- [ ] Troubleshoot missing endpoints.
- [ ] Create a NetworkPolicy.
- [ ] Test allowed/blocked traffic.
- [ ] Inspect CoreDNS.
- [ ] Explain north-south vs east-west traffic.
- [ ] Explain Ingress vs Gateway API.
- [ ] Draw your production Kubernetes networking architecture.

---

# 65. Final Day 7 Challenge

Explain this without notes:

```text
                         Internet
                            ↓
                    Cloud Load Balancer
                            ↓
                       Ingress/Gateway
                       /                                  ↓              ↓
              frontend-service   api-service
                      ↓              ↓
                Frontend Pods    Backend Pods
                                     ↓
                                db-service
                                     ↓
                                  Database
```

Then explain:

1. How external traffic enters the cluster.
2. What an Ingress Controller does.
3. What happens after an Ingress rule matches.
4. How a Service finds backend Pods.
5. What EndpointSlices are.
6. How DNS resolves `backend`.
7. Difference between `port` and `targetPort`.
8. ClusterIP vs NodePort vs LoadBalancer.
9. Why clients should avoid Pod IPs.
10. What CNI does.
11. What CoreDNS does.
12. What kube-proxy does at a high level.
13. What NetworkPolicy does.
14. How you would implement default-deny networking.
15. East-west vs north-south traffic.
16. How you would troubleshoot a Service with no endpoints.
17. How you would troubleshoot DNS.
18. How you would troubleshoot Ingress 404.
19. How you would troubleshoot NetworkPolicy issues.
20. How you would design networking for 100 microservices.

---

# 66. Success Criteria

You are ready for **Day 8 — Kubernetes Storage, Config, Secrets & Stateful Workloads** when you can explain:

```text
Client
  ↓
Load Balancer
  ↓
Ingress / Gateway
  ↓
Service
  ↓
EndpointSlices
  ↓
Pods
  ↓
Containers
```

and:

```text
Pod
 ↓
CNI
 ↓
Pod Network
 ↓
Service Discovery
 ↓
CoreDNS
```

You should also be able to troubleshoot a basic Kubernetes networking problem systematically instead of changing YAML randomly.

---

# Next Day

## Day 8 — Kubernetes Storage, Config, Secrets & Stateful Workloads

Topics:

- Kubernetes volumes
- `emptyDir`
- PersistentVolumes
- PersistentVolumeClaims
- StorageClasses
- Dynamic provisioning
- CSI
- StatefulSets
- Headless Services
- Stateful workload architecture
- ConfigMaps
- Secrets
- External secret management
- Azure Disk / Azure Files concepts
- AWS EBS / EFS concepts
- Backup and recovery
- Storage troubleshooting
- StatefulSet troubleshooting
- 25+ interview questions
