# Day 9 — Kubernetes Security & RBAC

> **30-Day DevOps → MLOps Study Plan**  
> **Focus:** Kubernetes security model, Authentication vs Authorization, RBAC, Roles, ClusterRoles, RoleBindings, ClusterRoleBindings, ServiceAccounts, least privilege, SecurityContext, Pod Security Standards, NetworkPolicies, Secrets security, Workload Identity, Azure Workload Identity, AWS IRSA, image security, admission controls, security troubleshooting, practical RBAC lab, and senior-level interview preparation  
> **Recommended time:** 4–4.5 hours  
> **Target:** ~6 years DevOps experience

---

# 1. Day 9 Objectives

By the end of Day 9, you should be able to:

- Explain the Kubernetes security model.
- Understand Authentication vs Authorization.
- Understand RBAC.
- Understand Roles and ClusterRoles.
- Understand RoleBindings and ClusterRoleBindings.
- Understand ServiceAccounts.
- Apply least-privilege access.
- Understand Kubernetes SecurityContext.
- Understand Pod Security Standards.
- Understand NetworkPolicy as a security control.
- Understand Secret security.
- Understand image security.
- Understand admission control.
- Understand workload identity.
- Understand Azure Workload Identity at a high level.
- Understand AWS IAM Roles for Service Accounts at a high level.
- Troubleshoot `Forbidden` and RBAC errors.
- Build a practical RBAC lab.
- Answer senior-level Kubernetes security interview questions.

---

# 2. Recommended Schedule

| Time | Topic |
|---|---|
| 00:00–00:35 | Kubernetes security model |
| 00:35–01:25 | Authentication, Authorization & RBAC |
| 01:25–02:05 | Roles, Bindings & ServiceAccounts |
| 02:05–02:40 | SecurityContext & Pod Security |
| 02:40–03:10 | Secrets, Images & Admission |
| 03:10–03:50 | Workload Identity + cloud security |
| 03:50–04:30 | Hands-on RBAC lab |
| 04:30–05:00 | Troubleshooting + interview Q&A |

---

# 3. Kubernetes Security Model

Think about Kubernetes security as multiple layers:

```text
                    Kubernetes Security
                           |
       ┌───────────────────┼───────────────────┐
       ↓                   ↓                   ↓
 Authentication       Authorization        Admission
       ↓                   ↓                   ↓
 Who are you?         What can you do?    Should request be
                                         allowed/modified?
       |
       └───────────────────────────────────────────
                           ↓
                    Workload Security
                           ↓
              ┌────────────┼────────────┐
              ↓            ↓            ↓
         Containers     Network       Secrets
              ↓            ↓            ↓
        SecurityContext  Policies    Secret Manager
```

A strong production security model uses multiple layers instead of depending on one mechanism.

---

# 4. Authentication vs Authorization

This is one of the most important interview concepts.

### Authentication

> Who are you?

Examples:

```text
User
ServiceAccount
OIDC identity
Certificate
Cloud identity
```

### Authorization

> What are you allowed to do?

Examples:

```text
get Pods
create Deployments
delete Services
read Secrets
```

Easy memory:

```text
Authentication
= Who are you?

Authorization
= What can you do?
```

---

# 5. Kubernetes API Security Flow

When a request reaches the Kubernetes API:

```text
Client
  ↓
Authentication
  ↓
Authorization
  ↓
Admission
  ↓
API Server
  ↓
Resource
```

For a request such as:

```bash
kubectl get pods
```

the cluster needs to determine:

1. Who is making the request?
2. Is that identity authorized?
3. Do admission controls allow/modify the request?
4. If allowed, perform the operation.

---

# 6. Authentication

Kubernetes can authenticate identities using mechanisms such as:

- Client certificates
- Bearer tokens
- OIDC
- ServiceAccount credentials
- Cloud/provider integrations

Important:

> Kubernetes authentication identifies the caller; it does not decide what that caller can do.

Authorization is a separate step.

---

# 7. Authorization

Kubernetes supports authorization mechanisms including:

- RBAC
- Node authorization
- Webhook authorization
- Other configured authorization modes

RBAC is the most important mechanism for day-to-day Kubernetes administration and workload permissions.

---

# 8. RBAC

RBAC means:

> Role-Based Access Control

It answers:

```text
WHO
 ↓
CAN DO WHAT
 ↓
WHERE
```

Example:

```text
developer
   ↓
get/list Pods
   ↓
namespace: development
```

---

# 9. RBAC Components

Four objects are fundamental:

```text
Role
ClusterRole
RoleBinding
ClusterRoleBinding
```

Think:

```text
Role
   ↓
Permissions

RoleBinding
   ↓
Who receives permissions?
```

---

# 10. Role

A Role grants permissions within a namespace.

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role

metadata:
  name: pod-reader
  namespace: dev

rules:

- apiGroups:
  - ""

  resources:
  - pods

  verbs:
  - get
  - list
  - watch
```

This allows the holder to:

```text
get Pods
list Pods
watch Pods
```

inside:

```text
dev
```

---

# 11. Verbs

Common RBAC verbs:

```text
get
list
watch
create
update
patch
delete
deletecollection
```

Examples:

```text
get
→ Read one object

list
→ List resources

watch
→ Watch changes

create
→ Create resources

delete
→ Delete resources
```

---

# 12. Least Privilege

Bad:

```yaml
verbs:
- "*"
```

Bad:

```yaml
resources:
- "*"
```

Better:

```yaml
resources:
- pods

verbs:
- get
- list
```

Principle:

> Give identities only the permissions they actually need.

---

# 13. ClusterRole

ClusterRole can define permissions for cluster-scoped resources or reusable permissions that may be bound to namespaces.

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole

metadata:
  name: pod-reader

rules:

- apiGroups:
  - ""

  resources:
  - pods

  verbs:
  - get
  - list
  - watch
```

---

# 14. Role vs ClusterRole

| Role | ClusterRole |
|---|---|
| Namespace-scoped object | Cluster-scoped RBAC object |
| Commonly used for namespace permissions | Can cover cluster-scoped resources |
| Permissions are applied within namespace through RoleBinding | Can be used with RoleBinding or ClusterRoleBinding |

Important:

> A ClusterRole is not automatically "cluster-wide access." Its effective scope depends on what it contains and how it is bound.

---

# 15. RoleBinding

RoleBinding grants permissions from a Role or ClusterRole to subjects in a namespace.

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding

metadata:
  name: read-pods
  namespace: dev

subjects:

- kind: User
  name: dev-user

roleRef:

  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Flow:

```text
User
 ↓
RoleBinding
 ↓
Role
 ↓
Permissions
```

---

# 16. ClusterRoleBinding

ClusterRoleBinding grants a ClusterRole to subjects at cluster scope.

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding

metadata:
  name: platform-reader

subjects:

- kind: User
  name: platform-user

roleRef:

  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Be careful:

> ClusterRoleBindings can grant broad access across the cluster.

Use them carefully.

---

# 17. RoleBinding with ClusterRole

A ClusterRole can also be referenced by a RoleBinding.

Example:

```text
ClusterRole
     ↓
RoleBinding
     ↓
Namespace: dev
```

This is useful when you want reusable permissions but namespace-limited access.

---

# 18. ServiceAccount

A ServiceAccount represents an identity for workloads running inside Kubernetes.

Example:

```yaml
apiVersion: v1
kind: ServiceAccount

metadata:
  name: app-sa
  namespace: dev
```

Pod:

```yaml
spec:
  serviceAccountName: app-sa
```

Concept:

```text
Pod
 ↓
ServiceAccount
 ↓
Kubernetes API permissions
```

---

# 19. ServiceAccount vs User

### User

Usually represents a human or external identity.

```text
Developer
 ↓
kubectl
 ↓
API Server
```

### ServiceAccount

Represents a workload identity.

```text
Pod
 ↓
ServiceAccount
 ↓
API Server
```

---

# 20. ServiceAccount Does Not Automatically Give Permissions

Creating:

```yaml
kind: ServiceAccount
```

does not automatically mean:

```text
Can access everything
```

Permissions come from bindings.

Example:

```text
ServiceAccount
     ↓
RoleBinding
     ↓
Role
     ↓
Permissions
```

---

# 21. Inspect ServiceAccounts

```bash
kubectl get serviceaccounts -A
```

Namespace:

```bash
kubectl get serviceaccounts -n dev
```

Describe:

```bash
kubectl describe serviceaccount app-sa -n dev
```

---

# 22. `kubectl auth can-i`

One of the most useful security troubleshooting commands:

```bash
kubectl auth can-i get pods
```

Specific namespace:

```bash
kubectl auth can-i get pods -n dev
```

Specific user:

```bash
kubectl auth can-i get pods   --as=dev-user   -n dev
```

Check negative permission:

```bash
kubectl auth can-i delete pods   --as=dev-user   -n dev
```

Expected:

```text
yes
```

or:

```text
no
```

---

# 23. Resource and API Group

RBAC rules use:

```yaml
apiGroups:
resources:
verbs:
```

For core resources:

```yaml
apiGroups:
- ""
```

For Deployments:

```yaml
apiGroups:
- apps
```

Example:

```yaml
resources:
- deployments

apiGroups:
- apps
```

---

# 24. Resource Names

You can restrict permissions to specific resource names.

Example concept:

```yaml
resourceNames:
- my-config
```

This can be useful when an identity should access only specific resources.

Always verify the exact API behavior for the resource and verb you are restricting.

---

# 25. RBAC Wildcards

Avoid broad rules such as:

```yaml
apiGroups:
- "*"

resources:
- "*"

verbs:
- "*"
```

This effectively grants very broad privileges.

Better:

```yaml
apiGroups:
- apps

resources:
- deployments

verbs:
- get
- list
- watch
```

---

# 26. SecurityContext

SecurityContext controls security-related settings for Pods and containers.

Examples:

```yaml
securityContext:
  runAsNonRoot: true
```

Other important controls include:

```text
runAsUser
runAsGroup
fsGroup
allowPrivilegeEscalation
readOnlyRootFilesystem
capabilities
seccompProfile
```

---

# 27. Example SecurityContext

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secure-app

spec:

  securityContext:
    runAsNonRoot: true

  containers:

  - name: app
    image: nginx:1.27

    securityContext:

      allowPrivilegeEscalation: false

      capabilities:
        drop:
        - ALL

      seccompProfile:
        type: RuntimeDefault
```

Important:

> The exact security settings must be compatible with the container image and application.

---

# 28. Why Run as Non-Root?

Running as root inside a container increases the impact of certain application/container compromises.

Prefer:

```text
Non-root user
+
Minimal privileges
```

instead of:

```text
Root
+
Full capabilities
```

---

# 29. Linux Capabilities

Linux capabilities divide some root privileges into smaller units.

Avoid unnecessary capabilities.

Example:

```yaml
capabilities:
  drop:
  - ALL
```

Then add only required capabilities.

Principle:

```text
No capability
   ↓
Add only what is required
```

---

# 30. Privileged Containers

Avoid:

```yaml
securityContext:
  privileged: true
```

unless there is a specific, reviewed requirement.

A privileged container can have significantly more access to the underlying host.

For most application workloads:

```text
privileged: false
```

is preferable.

---

# 31. Read-Only Root Filesystem

Example:

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

This can reduce the ability of an attacker or compromised application to write arbitrary files into the container filesystem.

If the application needs temporary write space, provide an appropriate writable volume such as:

```yaml
emptyDir: {}
```

---

# 32. Pod Security Standards

Kubernetes defines Pod Security Standards with profiles such as:

```text
Privileged
Baseline
Restricted
```

### Privileged

Very permissive.

### Baseline

Blocks many known privilege-escalation patterns.

### Restricted

Strong hardening with more security requirements.

Production application namespaces should generally aim for a strong security posture appropriate to the workload.

---

# 33. Pod Security Admission

Kubernetes can enforce Pod Security Standards using Pod Security Admission.

Concept:

```text
Namespace
   ↓
Pod Security Admission
   ↓
Pod manifest
   ↓
Allowed / Warn / Audit
```

Namespace labels can be used to configure enforcement behavior.

Example:

```text
pod-security.kubernetes.io/enforce=restricted
```

Do not apply restrictive policies blindly to existing workloads; first test compatibility.

---

# 34. NetworkPolicy as Security

NetworkPolicy provides network-level segmentation.

Example:

```text
Frontend
   ↓
Backend
   ↓
Database
```

Security objective:

```text
Frontend → Backend     ✓
Backend → Database     ✓
Frontend → Database    ✗
```

Combine:

```text
RBAC
+
NetworkPolicy
+
SecurityContext
```

---

# 35. Kubernetes Secrets

Secrets are intended for sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: db-secret

type: Opaque

stringData:
  username: appuser
  password: change-me
```

Use:

```yaml
envFrom:

- secretRef:
    name: db-secret
```

or mount as a file.

---

# 36. Important Secret Security Fact

Kubernetes Secret objects are not automatically a complete enterprise secret-management solution.

Production considerations:

- Encrypt Secrets at rest in etcd.
- Restrict RBAC access.
- Avoid putting credentials in Git.
- Avoid exposing Secret values in logs.
- Use short-lived credentials where possible.
- Consider external secret managers.
- Use workload identity instead of static cloud credentials.

---

# 37. Encryption at Rest

Concept:

```text
Secret
 ↓
API Server
 ↓
etcd
```

If etcd data is not appropriately protected/encrypted, sensitive data can be exposed.

Production clusters should use appropriate encryption-at-rest controls for sensitive resources.

---

# 38. Image Security

Before running an image:

```text
Source
 ↓
Build
 ↓
Scan
 ↓
Sign / Verify
 ↓
Registry
 ↓
Deploy
```

Security checks can include:

- Known vulnerabilities
- Base image vulnerabilities
- Malicious packages
- Secrets accidentally included in image
- Image provenance
- Image signatures

---

# 39. Image Best Practices

Prefer:

```text
Minimal base image
Non-root
Pinned/controlled versions
Regular scanning
Trusted registry
SBOM
Image signing
```

Avoid:

```text
latest
```

for production release processes where immutable versioning is required.

Prefer:

```text
myapp:1.4.2
```

or an immutable digest.

---

# 40. Admission Control

Admission happens after authentication/authorization and before the object is persisted.

Concept:

```text
Request
 ↓
Authentication
 ↓
Authorization
 ↓
Admission
 ↓
Persist
```

Admission can:

- Reject requests
- Modify requests
- Enforce policies
- Validate security requirements

---

# 41. Admission Examples

Common ecosystem tools/patterns include:

- Pod Security Admission
- Validating admission policies
- Kyverno
- OPA Gatekeeper
- Image policy/signature verification integrations

Example policy:

```text
Reject image:
latest
```

or:

```text
Reject privileged containers
```

---

# 42. Workload Identity

Applications often need access to cloud services.

Bad:

```text
Pod
 ↓
Hard-coded AWS/Azure credentials
```

Better:

```text
Pod
 ↓
Kubernetes / Cloud Workload Identity
 ↓
Cloud IAM
 ↓
Cloud Service
```

Benefits:

- No long-lived static credentials in Pods.
- Better rotation.
- Better auditability.
- Least privilege.
- Easier credential lifecycle management.

---

# 43. Azure Workload Identity

High-level:

```text
AKS Pod
  ↓
Kubernetes ServiceAccount
  ↓
Federated Identity
  ↓
Microsoft Entra ID
  ↓
Azure Resource
```

Example:

```text
Pod
 ↓
ServiceAccount
 ↓
Workload Identity
 ↓
Azure Key Vault
```

The application can obtain an Azure identity without storing a long-lived client secret in the container.

---

# 44. AWS IAM Roles for Service Accounts

Common EKS pattern:

```text
Pod
 ↓
Kubernetes ServiceAccount
 ↓
IAM Role
 ↓
AWS API
```

This is commonly referred to as:

> IRSA — IAM Roles for Service Accounts

Modern AWS architectures may also use EKS Pod Identity. Know the concept and understand which mechanism your organization uses.

---

# 45. Least Privilege Cloud Access

Bad:

```text
Pod
 ↓
Admin role
 ↓
Entire cloud account
```

Better:

```text
Application
 ↓
Specific ServiceAccount
 ↓
Specific IAM/Azure role
 ↓
Only required resources
```

Example:

```text
Image processor
 ↓
Read:
s3://images/input
 ↓
Write:
s3://images/output
```

rather than full storage-account access.

---

# 46. Security Architecture

A strong Kubernetes security architecture looks like:

```text
                    User
                     ↓
              Authentication
                     ↓
                API Server
                     ↓
                Authorization
                     ↓
                Admission
                     ↓
              Kubernetes Objects
                     ↓
        ┌────────────┼────────────┐
        ↓            ↓            ↓
      RBAC      NetworkPolicy  SecurityContext
        ↓            ↓            ↓
   API access    Network      Container
                 access       hardening
```

Cloud identity:

```text
Pod
 ↓
Workload Identity
 ↓
Cloud IAM
 ↓
Azure/AWS resources
```

---

# 47. Day 9 Hands-On Lab — RBAC

## Objective

Create:

```text
Namespace: dev
User: dev-user
Role: pod-reader
RoleBinding
```

The user should:

```text
get Pods
list Pods
watch Pods
```

but should not:

```text
delete Pods
create Deployments
read Secrets
```

---

# 48. Step 1 — Create Namespace

```bash
kubectl create namespace dev
```

---

# 49. Step 2 — Create Role

`pod-reader-role.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role

metadata:
  name: pod-reader
  namespace: dev

rules:

- apiGroups:
  - ""

  resources:
  - pods

  verbs:
  - get
  - list
  - watch
```

Apply:

```bash
kubectl apply -f pod-reader-role.yaml
```

---

# 50. Step 3 — Create ServiceAccount

```bash
kubectl create serviceaccount dev-reader -n dev
```

Check:

```bash
kubectl get serviceaccount -n dev
```

---

# 51. Step 4 — Create RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding

metadata:
  name: dev-reader-binding
  namespace: dev

subjects:

- kind: ServiceAccount
  name: dev-reader
  namespace: dev

roleRef:

  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f rolebinding.yaml
```

---

# 52. Step 5 — Test Permissions

Check:

```bash
kubectl auth can-i get pods   --as=system:serviceaccount:dev:dev-reader   -n dev
```

Expected:

```text
yes
```

Check:

```bash
kubectl auth can-i delete pods   --as=system:serviceaccount:dev:dev-reader   -n dev
```

Expected:

```text
no
```

Check:

```bash
kubectl auth can-i get secrets   --as=system:serviceaccount:dev:dev-reader   -n dev
```

Expected:

```text
no
```

---

# 53. Step 6 — Test Deployment Permission

```bash
kubectl auth can-i create deployments   --as=system:serviceaccount:dev:dev-reader   -n dev
```

Expected:

```text
no
```

This proves least privilege.

---

# 54. Step 7 — Create a Pod Using ServiceAccount

Example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: rbac-test
  namespace: dev

spec:

  serviceAccountName: dev-reader

  containers:

  - name: app
    image: nginx:1.27
```

Apply:

```bash
kubectl apply -f rbac-test.yaml
```

---

# 55. Step 8 — Verify ServiceAccount

```bash
kubectl get pod rbac-test -n dev   -o jsonpath='{.spec.serviceAccountName}'
```

Expected:

```text
dev-reader
```

---

# 56. Step 9 — Test API Access from Pod

Enter:

```bash
kubectl exec -it rbac-test -n dev -- /bin/sh
```

For actual API testing, use a purpose-built Kubernetes client/debugging image rather than assuming the NGINX image contains `curl`, `wget`, or Kubernetes tooling.

The important concept is:

```text
Pod
 ↓
ServiceAccount identity
 ↓
RBAC
 ↓
API Server
```

---

# 57. Step 10 — Inspect RBAC

```bash
kubectl get role -n dev
```

```bash
kubectl get rolebinding -n dev
```

Detailed:

```bash
kubectl describe role pod-reader -n dev
```

```bash
kubectl describe rolebinding dev-reader-binding -n dev
```

---

# 58. Step 11 — Break the Binding

Temporarily change:

```yaml
roleRef:
  name: wrong-role
```

Observe the failure.

Then restore:

```yaml
roleRef:
  name: pod-reader
```

Test again:

```bash
kubectl auth can-i get pods   --as=system:serviceaccount:dev:dev-reader   -n dev
```

---

# 59. SecurityContext Lab

Create:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secure-pod
  namespace: dev

spec:

  securityContext:
    runAsNonRoot: true

  containers:

  - name: app
    image: nginx:1.27

    securityContext:
      allowPrivilegeEscalation: false

      capabilities:
        drop:
        - ALL

      seccompProfile:
        type: RuntimeDefault
```

Apply:

```bash
kubectl apply -f secure-pod.yaml
```

If the image does not support the chosen non-root configuration, Kubernetes may reject or fail the workload. This is an intentional lesson: security hardening must be compatible with the application image.

---

# 60. Troubleshooting — Forbidden

Error:

```text
Error from server (Forbidden)
```

Troubleshooting:

```text
Who?
 ↓
Which namespace?
 ↓
Which resource?
 ↓
Which verb?
 ↓
Which Role?
 ↓
Which Binding?
```

Use:

```bash
kubectl auth can-i <verb> <resource>   --as=<identity>   -n <namespace>
```

---

# 61. Troubleshooting — RBAC

Check:

```bash
kubectl get role -n dev
kubectl get rolebinding -n dev
kubectl describe role <role> -n dev
kubectl describe rolebinding <binding> -n dev
```

Then verify:

```bash
kubectl auth can-i ...
```

Do not randomly add:

```text
cluster-admin
```

just to make the error disappear.

---

# 62. Troubleshooting — Pod Security

Symptoms:

```text
Pod rejected
```

Check:

- Namespace Pod Security labels
- `securityContext`
- Privileged settings
- Host networking
- HostPath
- Linux capabilities
- Run-as user
- Seccomp profile

Use:

```bash
kubectl describe pod <pod> -n dev
```

and inspect API/admission error messages.

---

# 63. Troubleshooting — Secret Access

If an application cannot access a Secret:

Check:

```bash
kubectl get secret -n dev
kubectl auth can-i get secrets   --as=system:serviceaccount:dev:dev-reader   -n dev
```

Remember:

```text
Secret exists
≠
Application can read Secret
```

RBAC determines API access.

---

# 64. Troubleshooting — Cloud Access

If:

```text
Pod → Azure/AWS API
```

fails:

Check:

```text
ServiceAccount
 ↓
Workload Identity configuration
 ↓
Cloud role
 ↓
Cloud policy
 ↓
Target resource permissions
```

Do not immediately add administrator privileges.

---

# 65. Security Review Checklist

Before production deployment:

### Identity

- [ ] Workloads use dedicated ServiceAccounts.
- [ ] Human access uses appropriate enterprise identity.
- [ ] Cloud access uses workload identity where possible.

### RBAC

- [ ] Least privilege.
- [ ] No unnecessary `cluster-admin`.
- [ ] No broad `*` permissions.
- [ ] Namespace access is restricted.

### Containers

- [ ] Run as non-root where possible.
- [ ] No unnecessary privileged containers.
- [ ] Drop unnecessary Linux capabilities.
- [ ] Disable privilege escalation where possible.
- [ ] Use seccomp.
- [ ] Use read-only root filesystem where practical.

### Network

- [ ] NetworkPolicies are defined.
- [ ] East-west access is restricted.
- [ ] Only required ingress/egress is allowed.

### Images

- [ ] Images are scanned.
- [ ] Images come from trusted registries.
- [ ] Versions/digests are controlled.
- [ ] Images are signed/verified where required.
- [ ] SBOM/provenance is available where required.

### Secrets

- [ ] No secrets in Git.
- [ ] Secrets encrypted at rest.
- [ ] RBAC restricts Secret access.
- [ ] External secret management considered.
- [ ] Credentials are rotated.

### Admission

- [ ] Security policies are enforced.
- [ ] Privileged workloads are controlled.
- [ ] Unsafe images/configurations are rejected.

---

# 66. Day 9 Interview Q&A

## Q1. What is Kubernetes RBAC?

> RBAC controls which authenticated identities can perform which actions on Kubernetes resources.

## Q2. Authentication vs Authorization?

> Authentication identifies who the caller is; authorization determines what the caller is allowed to do.

## Q3. What is a Role?

> A namespace-scoped RBAC object that defines permissions.

## Q4. What is a ClusterRole?

> An RBAC object that can define permissions for cluster-scoped resources or reusable permissions that can be bound within namespaces or cluster-wide.

## Q5. Role vs ClusterRole?

> Role is namespace-scoped; ClusterRole is cluster-scoped as an RBAC object and can also be referenced by namespace RoleBindings.

## Q6. What is RoleBinding?

> It grants a Role or ClusterRole's permissions to subjects within a namespace.

## Q7. What is ClusterRoleBinding?

> It grants a ClusterRole to subjects at cluster scope.

## Q8. What is a ServiceAccount?

> A Kubernetes identity intended for workloads.

## Q9. Does creating a ServiceAccount grant permissions?

> No. Permissions come from RBAC bindings and other configured authorization mechanisms.

## Q10. What is least privilege?

> Grant only the minimum permissions required to perform the workload's function.

## Q11. How do you test RBAC?

> Use `kubectl auth can-i` with the relevant identity, resource, verb, and namespace.

## Q12. What does `*` in RBAC mean?

> A wildcard matching broad sets of API groups, resources, or verbs. It should be avoided unless there is a justified requirement.

## Q13. What is SecurityContext?

> It defines security-related settings for a Pod or container, such as user/group IDs, privilege escalation, capabilities, seccomp, and filesystem settings.

## Q14. Why run containers as non-root?

> It reduces the privileges available to a compromised application/container.

## Q15. What is Pod Security Standards?

> Kubernetes security profiles describing different levels of workload security: Privileged, Baseline, and Restricted.

## Q16. What is Pod Security Admission?

> A built-in admission controller that can enforce, audit, or warn on Pod Security Standards at namespace boundaries.

## Q17. What is NetworkPolicy?

> A resource for controlling allowed network traffic to and from selected Pods when supported by the CNI.

## Q18. Are Kubernetes Secrets encrypted automatically?

> Kubernetes supports encryption at rest, but production clusters must be configured appropriately. Base64 encoding itself is not encryption.

## Q19. Why shouldn't secrets be stored in Git?

> Git history is durable and widely replicated, making accidental exposure difficult to eliminate. Use appropriate secret-management systems instead.

## Q20. What is admission control?

> A stage after authentication and authorization where Kubernetes can validate or mutate API requests before persistence.

## Q21. What is workload identity?

> A mechanism that gives workloads an identity they can use to access external/cloud resources without embedding long-lived credentials.

## Q22. What is Azure Workload Identity?

> An AKS pattern using Kubernetes ServiceAccounts and federated identity with Microsoft Entra ID to obtain Azure permissions without static application credentials.

## Q23. What is IRSA?

> IAM Roles for Service Accounts, a common EKS pattern that maps Kubernetes ServiceAccounts to AWS IAM roles.

## Q24. What is EKS Pod Identity?

> An AWS mechanism for associating EKS Pods with IAM permissions without requiring the older IRSA configuration model.

## Q25. Why should we avoid `cluster-admin`?

> It grants extremely broad privileges and increases the blast radius of compromised credentials or workloads.

## Q26. What is a privileged container?

> A container configured with elevated privileges that can significantly weaken isolation from the host. It should be avoided unless required.

## Q27. What is `allowPrivilegeEscalation: false`?

> It prevents a process from gaining more privileges than its parent process, subject to the container/runtime/security configuration.

## Q28. What is seccomp?

> A Linux security mechanism that restricts the system calls a process can make.

## Q29. How do you troubleshoot `Forbidden`?

> Identify the caller, namespace, resource, verb, Role/ClusterRole, and Binding, then verify with `kubectl auth can-i`.

## Q30. How would you secure a production Kubernetes cluster?

> Use strong identity integration, least-privilege RBAC, workload identity, Pod hardening, NetworkPolicies, image security, Secret protection, admission policies, logging/auditing, regular upgrades, and tested incident/backup procedures.

---

# 67. Senior-Level Scenarios

## Scenario 1 — Developer Needs Pod Logs but Nothing Else

Requirement:

```text
Developer
 ↓
Only read Pod information/logs
 ↓
Namespace: development
```

Do not give:

```text
cluster-admin
```

Create a narrowly scoped Role and RoleBinding.

Consider exactly which API resources/verbs are needed for the operational task.

---

## Scenario 2 — CI/CD Pipeline Needs Deployment Access

Bad:

```text
CI/CD
 ↓
cluster-admin
```

Better:

```text
CI/CD ServiceAccount
 ↓
Dedicated Role
 ↓
Only required namespaces/resources
 ↓
Only required verbs
```

For example, limit access to:

```text
deployments
services
configmaps
pods
```

only if the pipeline genuinely needs those resources.

---

## Scenario 3 — Application Needs Key Vault Access

Bad:

```text
Pod
 ↓
Client secret stored in Kubernetes Secret
 ↓
Key Vault
```

Better:

```text
Pod
 ↓
Dedicated ServiceAccount
 ↓
Azure Workload Identity
 ↓
Entra ID
 ↓
Key Vault
```

Grant only the required Key Vault permissions.

---

## Scenario 4 — Pod Can Access Kubernetes API Unexpectedly

Investigate:

```text
ServiceAccount
 ↓
RoleBindings
 ↓
ClusterRoleBindings
 ↓
Default ServiceAccount usage
 ↓
Application token handling
```

Use:

```bash
kubectl auth can-i --list   --as=system:serviceaccount:<namespace>:<serviceaccount>
```

Review all bindings associated with the identity.

---

## Scenario 5 — Security Team Wants No Privileged Containers

Implement:

```text
Admission Policy
       ↓
Reject privileged: true
```

Then validate workloads in CI before deployment.

Do not rely only on developers remembering the rule.

---

## Scenario 6 — Compromised Pod

A good defense-in-depth architecture limits blast radius:

```text
Compromised Pod
      ↓
Non-root
      ↓
Dropped capabilities
      ↓
Read-only filesystem
      ↓
NetworkPolicy
      ↓
Least-privilege ServiceAccount
      ↓
Limited cloud identity
```

No single security control should be considered sufficient.

---

# 68. Practical Security Commands

List RBAC:

```bash
kubectl get roles -A
kubectl get clusterroles
kubectl get rolebindings -A
kubectl get clusterrolebindings
```

Check permissions:

```bash
kubectl auth can-i get pods -n dev
```

List all permissions:

```bash
kubectl auth can-i --list
```

For a ServiceAccount:

```bash
kubectl auth can-i --list   --as=system:serviceaccount:dev:dev-reader   -n dev
```

Inspect a Role:

```bash
kubectl describe role pod-reader -n dev
```

Inspect a RoleBinding:

```bash
kubectl describe rolebinding dev-reader-binding -n dev
```

---

# 69. Useful Security Context Commands

Inspect Pod:

```bash
kubectl get pod <pod> -o yaml
```

Inspect SecurityContext:

```bash
kubectl get pod <pod>   -o jsonpath='{.spec.securityContext}'
```

Inspect container SecurityContext:

```bash
kubectl get pod <pod>   -o jsonpath='{.spec.containers[*].securityContext}'
```

---

# 70. Day 9 Practical Checklist

- [ ] I understand Authentication.
- [ ] I understand Authorization.
- [ ] I understand RBAC.
- [ ] I understand Roles.
- [ ] I understand ClusterRoles.
- [ ] I understand RoleBindings.
- [ ] I understand ClusterRoleBindings.
- [ ] I understand ServiceAccounts.
- [ ] I understand least privilege.
- [ ] I can use `kubectl auth can-i`.
- [ ] I understand API groups.
- [ ] I understand RBAC verbs.
- [ ] I understand SecurityContext.
- [ ] I understand non-root containers.
- [ ] I understand Linux capabilities.
- [ ] I understand seccomp.
- [ ] I understand Pod Security Standards.
- [ ] I understand Pod Security Admission.
- [ ] I understand NetworkPolicy.
- [ ] I understand Secret security.
- [ ] I understand image security.
- [ ] I understand admission control.
- [ ] I understand workload identity.
- [ ] I understand Azure Workload Identity.
- [ ] I understand AWS IRSA/EKS Pod Identity concepts.
- [ ] I completed the RBAC lab.
- [ ] I tested a ServiceAccount.
- [ ] I tested `kubectl auth can-i`.
- [ ] I troubleshot a Forbidden error.
- [ ] I answered at least 25 interview questions aloud.

---

# 71. Homework

- [ ] Create a namespace.
- [ ] Create a ServiceAccount.
- [ ] Create a Role.
- [ ] Create a RoleBinding.
- [ ] Verify allowed permissions.
- [ ] Verify denied permissions.
- [ ] Create a Pod using the ServiceAccount.
- [ ] Inspect the ServiceAccount.
- [ ] Run `kubectl auth can-i --list`.
- [ ] Create a restricted SecurityContext.
- [ ] Research your organization's cloud workload identity pattern.
- [ ] Review how Secrets are encrypted at rest in your Kubernetes platform.
- [ ] Scan a container image.
- [ ] Identify three workloads that should not have `cluster-admin`.
- [ ] Design a least-privilege RBAC model for a DevOps team.
- [ ] Draw a production Kubernetes security architecture.

---

# 72. Final Day 9 Challenge

Without looking at your notes, explain:

```text
                    User / Pod
                        ↓
                 Authentication
                        ↓
                  Kubernetes API
                        ↓
                  Authorization
                        ↓
                       RBAC
                        ↓
                    Admission
                        ↓
                Kubernetes Resource
```

Then explain:

1. Authentication vs Authorization.
2. Role vs ClusterRole.
3. RoleBinding vs ClusterRoleBinding.
4. ServiceAccount vs human user.
5. How a Pod gets Kubernetes API permissions.
6. What least privilege means.
7. How `kubectl auth can-i` works.
8. Why `cluster-admin` should be avoided.
9. What SecurityContext does.
10. Why containers should run as non-root.
11. What Linux capabilities are.
12. What seccomp is.
13. What Pod Security Standards are.
14. What Pod Security Admission does.
15. What NetworkPolicy does.
16. How to protect Kubernetes Secrets.
17. Why image scanning is important.
18. What admission control does.
19. What workload identity solves.
20. How Azure Workload Identity works at a high level.
21. What IRSA is.
22. How to troubleshoot a `Forbidden` error.
23. How to secure a CI/CD ServiceAccount.
24. How to secure a compromised Pod.
25. How you would design security for a production Kubernetes platform.

---

# 73. Success Criteria

You are ready for **Day 10 — Terraform Fundamentals & Infrastructure as Code** when you can explain:

```text
Identity
   ↓
Authentication
   ↓
Authorization / RBAC
   ↓
Admission
   ↓
Secure Workload
   ↓
NetworkPolicy
   ↓
Cloud Workload Identity
```

and troubleshoot:

```text
Forbidden
   ↓
Who?
   ↓
What resource?
   ↓
Which verb?
   ↓
Which namespace?
   ↓
Which Role?
   ↓
Which Binding?
```

Most importantly, you should understand this production principle:

> **Security is defense in depth.**

Use:

```text
Strong Identity
+
Least-Privilege RBAC
+
Workload Identity
+
Secure Containers
+
NetworkPolicies
+
Image Security
+
Secret Management
+
Admission Controls
+
Monitoring/Auditing
```

rather than relying on one control.

---

# Next Day

## Day 10 — Terraform Fundamentals & Infrastructure as Code

Topics:

- Infrastructure as Code
- Terraform architecture
- Providers
- Resources
- Data sources
- Variables
- Outputs
- Locals
- Terraform state
- Remote state
- State locking
- Modules
- `terraform init`
- `terraform plan`
- `terraform apply`
- `terraform destroy`
- Dependency graph
- `count` vs `for_each`
- `lifecycle`
- Import
- Drift
- Secrets in Terraform
- Azure Terraform
- AWS Terraform
- Terraform with CI/CD
- Practical Azure/AWS lab
- Terraform troubleshooting
- 30+ interview questions
