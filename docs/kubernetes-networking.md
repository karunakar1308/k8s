KUBERNETES NETWORKING

Connecting Pods, Services, and Users — the backbone of Kubernetes

Kubernetes networking is one of the most critical (and most interviewed) areas of Kubernetes. Unlike Docker, Kubernetes follows a flat networking model, where every Pod can talk to every other Pod by default — no NAT, no port conflicts.

This section explains how traffic flows inside and outside the cluster, how it’s secured, and how modern platforms use service meshes for advanced traffic control.

1️⃣ Kubernetes Networking Model (High Level)

Kubernetes networking is built on three core guarantees:

All Pods can communicate with all other Pods without NAT

Pods can communicate with Services

External traffic can reach cluster workloads

📌 Kubernetes itself does not implement networking — it relies on CNI plugins (Calico, Cilium, Flannel).

2️⃣ Kubernetes Services

A Service provides a stable virtual IP and DNS name to access Pods, even when Pods are recreated.

🔹 Service Types Overview
Service Type	Scope	Use Case
ClusterIP	Internal only	Internal microservices
NodePort	External (basic)	Dev / testing
LoadBalancer	External (cloud)	Production traffic
2.1 ClusterIP (Default)
What is ClusterIP?

Exposes service inside the cluster only

Not accessible from the internet

Ideal for backend services

YAML Example
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080

Access Pattern
curl http://backend-svc.default.svc.cluster.local

Real-World Use Case

✅ Frontend → Backend
✅ Backend → Database
❌ Direct internet access

2.2 NodePort
What is NodePort?

Exposes service on each node’s IP

Port range: 30000–32767

Simple but not production-friendly

YAML Example
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080

Access
curl http://<NodeIP>:30080

Pros / Cons

✅ Easy to test
❌ No TLS
❌ No load balancing across nodes

2.3 LoadBalancer
What is LoadBalancer?

Cloud-provider managed

Creates external LB (ELB, ALB, Azure LB)

Production-ready

YAML Example
apiVersion: v1
kind: Service
metadata:
  name: web-lb
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80

kubectl get svc web-lb

Real-World Use Case

✅ Public APIs
✅ Web applications
✅ SaaS platforms

📌 Cloud cost applies!

3️⃣ Ingress & Ingress Controllers
Why Ingress?

Using multiple LoadBalancers is expensive.
Ingress allows HTTP/HTTPS routing using a single entry point.

3.1 Ingress Controller

An Ingress resource alone does nothing — you need a controller.

Popular Controllers:

NGINX Ingress

Traefik

HAProxy

AWS ALB Ingress

3.2 Ingress YAML Example
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-svc
                port:
                  number: 80

Flow
Client → LoadBalancer → Ingress Controller → Service → Pod

4️⃣ Network Policies
What is a Network Policy?

Acts like a firewall for Pods

Controls ingress & egress traffic

Enforced by CNI plugin

📌 Default Kubernetes = allow all traffic

4.1 Deny All Ingress Example
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress

Allow Frontend → Backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend

Real-World Use Case

✅ Zero-trust networking
✅ Compliance (PCI, HIPAA)
❌ Not supported by all CNIs

5️⃣ CNI Plugins

Kubernetes delegates networking to CNI plugins.

5.1 Flannel

Simple overlay networking

No network policies

✅ Easy setup
❌ No security controls

5.2 Calico

Layer 3 routing

Powerful NetworkPolicies

✅ Production-grade
✅ Used in EKS, AKS

5.3 Cilium

eBPF-based networking

L7 visibility and security

✅ High performance
✅ Advanced observability
❌ Steeper learning curve

6️⃣ DNS in Kubernetes

Kubernetes uses CoreDNS.

DNS Pattern
<service>.<namespace>.svc.cluster.local

Example
nslookup backend-svc.default.svc.cluster.local

CoreDNS Use Cases

✅ Service discovery
✅ Decoupled microservices
✅ No hard-coded IPs

7️⃣ Service Mesh Fundamentals
What is a Service Mesh?

A dedicated infrastructure layer for:

Traffic management

Security

Observability

Uses sidecar proxies (Envoy).

7.1 Popular Service Meshes
Mesh	Use Case
Istio	Full-featured enterprise mesh
Linkerd	Lightweight, simple
Consul	Hybrid environments
7.2 What Service Mesh Solves

Without Mesh:
❌ App code handles retries, TLS, metrics

With Mesh:
✅ mTLS
✅ Canary deployments
✅ Traffic splitting
✅ Distributed tracing

8️⃣ Best Practices
✅ Do This

✅ Use Ingress instead of multiple LoadBalancers
✅ Implement NetworkPolicies early
✅ Use DNS names, not IPs
✅ Choose CNI based on security needs

❌ Avoid This

❌ NodePort in production
❌ Exposing databases publicly
❌ Skipping network policies
❌ Hard-coding service IPs

9️⃣ Interview Questions & Answers
Q1: How does Kubernetes networking differ from Docker?

A: Kubernetes provides a flat networking model where every Pod gets a unique IP and can communicate directly without NAT, unlike Docker bridge networking.

Q2: Difference between Service and Ingress?

A: A Service exposes Pods, while Ingress provides HTTP routing, TLS, and host-based access on top of Services.

Q3: What enforces Network Policies?

A: Network Policies are enforced by the CNI plugin (e.g., Calico, Cilium), not Kubernetes itself.

Q4: Why use a Service Mesh?

A: To handle cross-cutting concerns like security, traffic control, retries, and observability without modifying application code.

📌 Official Documentation

Kubernetes Services
https://kubernetes.io/docs/concepts/services-networking/service/

Ingress
https://kubernetes.io/docs/concepts/services-networking/ingress/

Network Policies
https://kubernetes.io/docs/concepts/services-networking/network-policies/

CNI
https://www.cni.dev/

CoreDNS
https://kubernetes.io/docs/tasks/administer-cluster/coredns/

Istio
https://istio.io/latest/docs/
