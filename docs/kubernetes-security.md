# PART 6: Kubernetes Security

Kubernetes is a powerful platform for container orchestration, but with that power comes responsibility for securing clusters, workloads, and data. This guide walks through core security primitives you will use in real clusters and interviews alike.

---

## 1. RBAC (Role-Based Access Control)

Kubernetes RBAC controls who can do what in your cluster. It is central to enforcing least privilege and avoiding over-privileged users and workloads.

### Key concepts

| Resource type      | Scope        | Description                                                 |
|--------------------|-------------|-------------------------------------------------------------|
| Role               | Namespace   | Grants permissions within a single namespace               |
| ClusterRole        | Cluster     | Grants permissions cluster-wide or across namespaces       |
| RoleBinding        | Namespace   | Binds a Role to a user or service account in a namespace   |
| ClusterRoleBinding | Cluster     | Binds a ClusterRole to a user or service account globally  |

### YAML examples

**Role (namespace-scoped)**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]

Role Binding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
  - kind: User
    name: jane
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io



ClusterRole Example

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]


ClusterRoleBinding Example

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-binding
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io

kubectl Commands
# Test access
kubectl auth can-i get pods --namespace=dev --as=jane

# List roles
kubectl get roles -n dev
kubectl get clusterroles

# Describe bindings
kubectl describe rolebinding read-pods-binding -n dev
kubectl describe clusterrolebinding admin-binding

Best Practices

✅ Use least privilege principle
✅ Prefer namespace-scoped Roles over ClusterRoles
❌ Avoid giving cluster-admin to service accounts unnecessarily
✅ Regularly audit RBAC bindings

Real-world Scenario

A team wants developers to read pods in dev namespace but not delete them. RBAC roles and bindings can enforce this access without giving cluster-wide admin rights.

Interview Questions

Q: Difference between Role and ClusterRole?
A: Role is namespace-scoped, ClusterRole is cluster-scoped and can also be used across namespaces.

Q: How do you test RBAC permissions?
A: kubectl auth can-i <verb> <resource> --namespace <ns> --as <user>

Official Docs: RBAC Authorization

2️⃣ Service Accounts

Service accounts are identities for pods, not humans.

Key Points
Feature	Description
User Accounts	Represent humans logging in
Service Accounts	Represent workloads (pods)
Token Mounting	Pods mount a JWT token to authenticate to API server
IRSA (AWS)	IAM Roles for Service Accounts, map AWS IAM to k8s SA
Workload Identity (GCP)	Map GCP service accounts to k8s SA
YAML Example
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: dev
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-role-binding
  namespace: dev
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

kubectl Commands
kubectl get serviceaccounts -n dev
kubectl describe sa app-sa -n dev

Best Practices

✅ Use service accounts for automation and workloads
✅ Do not use default service account for sensitive workloads
✅ Rotate tokens regularly when manually managed

Official Docs: Service Accounts

3️⃣ Pod Security Standards (PSS)

PSS define allowed security profiles for pods. Kubernetes supports Privileged, Baseline, and Restricted policies.

Profiles
Profile	Description
Privileged	Full access, like root (avoid if possible)
Baseline	Default least-risk constraints for common workloads
Restricted	Maximum restrictions, no root, limited capabilities
Pod Security Admission (PSA) Example
apiVersion: policy/v1
kind: PodSecurity
metadata:
  name: default
  namespace: dev
spec:
  enforce: "restricted"
  enforce-version: "latest"

Migration Guide

Identify pods violating restricted or baseline

Update YAML with securityContext

Enable PSA admission controller

YAML Example (Restricted Pod)
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
  - name: app
    image: nginx

Best Practices

✅ Enforce restricted for prod workloads
✅ Use baseline for dev/test
❌ Never run privileged pods unless absolutely needed

Official Docs: Pod Security Standards

4️⃣ Network Policies for Security

Network Policies control pod-to-pod communication. By default, pods can communicate freely.

Key Concepts
Type	Description
Ingress	Control incoming traffic to pods
Egress	Control outgoing traffic from pods
Default Deny	Block all traffic unless explicitly allowed
Namespace Isolation	Segregate network traffic between namespaces
Example: Deny All & Allow HTTP
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: dev
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-http
  namespace: dev
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - ports:
    - protocol: TCP
      port: 80

kubectl Commands
kubectl get networkpolicy -n dev
kubectl describe networkpolicy allow-http -n dev

Best Practices

✅ Implement default deny policies
✅ Use namespace isolation for multi-tenant clusters
✅ Test policies using kubectl exec or netcat

Official Docs: Network Policies

5️⃣ Admission Controllers

Admission controllers intercept API requests before they’re persisted.

Types
Type	Description
ValidatingWebhook	Rejects invalid requests
MutatingWebhook	Modifies requests
PodSecurityAdmission	Enforces PSS
OPA/Gatekeeper	Policy as code enforcement
Example: PodSecurity Admission
apiVersion: policy/v1
kind: PodSecurity
metadata:
  name: dev
  namespace: dev
spec:
  enforce: "baseline"

Best Practices

✅ Use admission controllers to enforce compliance
✅ Combine PSA with OPA/Gatekeeper for complex policies
❌ Don’t skip webhooks in production clusters

Official Docs: Admission Controllers

6️⃣ Security Contexts

Security contexts define security settings for pods and containers.

Key Options
Option	Description
runAsNonRoot	Ensure container doesn’t run as root
runAsUser	Specify UID
capabilities	Add/drop Linux capabilities
seccomp	System call filtering
AppArmor	Mandatory access control
YAML Example
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: nginx
    securityContext:
      capabilities:
        drop: ["ALL"]
      seccompProfile:
        type: RuntimeDefault

Best Practices

✅ Always run non-root
✅ Drop unnecessary capabilities
✅ Use seccomp and AppArmor profiles

Official Docs: Security Context

7️⃣ Secrets Management

Secrets are sensitive info (passwords, API keys). Avoid storing them in plaintext.

Options
Tool	Description
External Secrets Operator	Sync secrets from AWS/HashiCorp/GCP
HashiCorp Vault	Centralized secret storage
CSI driver	Mount secrets as volumes
Example: External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: db-password
  data:
  - secretKey: password
    remoteRef:
      key: database/password

Best Practices

✅ Don’t store secrets in Git
✅ Use RBAC to restrict access
✅ Rotate secrets regularly

Official Docs:

- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [External Secrets Operator](https://external-secrets.io/latest/)
- [Vault Integration](https://developer.hashicorp.com/vault/docs/platform/k8s)


✅ Conclusion

Kubernetes security is multi-layered. RBAC, Pod Security Standards, Network Policies, Admission Controllers, Security Contexts, and Secrets management together provide a strong defense-in-depth strategy. Implementing these best practices reduces attack surface and ensures compliance.
