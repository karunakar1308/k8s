Kubernetes Advanced Deployment Strategies: Rolling, Blue-Green, Canary, A/B & GitOps

Deploying applications in Kubernetes is easy—but deploying safely and efficiently is a different challenge. Advanced deployment strategies ensure zero downtime, controlled rollouts, and fast rollbacks. This guide covers the most common strategies used in production.

1️⃣ Rolling Updates

Rolling updates are Kubernetes’ default deployment strategy. Pods are updated incrementally, ensuring service availability.

YAML Example: Rolling Update

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: dev
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
        version: v2
    spec:
      containers:
      - name: web
        image: myregistry/web-app:v2
        ports:
        - containerPort: 80
```

kubectl Commands
# Apply rolling update
kubectl apply -f web-app-deployment.yaml

# Check rollout status
kubectl rollout status deployment/web-app -n dev

# Rollback
kubectl rollout undo deployment/web-app -n dev

Best Practices

✅ Use maxUnavailable and maxSurge to control pod replacement
✅ Monitor pods during rollout
❌ Don’t deploy large changes without testing

Real-world Scenario

Updating a web app from v1 to v2 without downtime by gradually replacing pods.

2️⃣ Blue-Green Deployments

Blue-Green deployment runs two identical environments, only switching traffic when the new version is validated.

Workflow

Blue environment serves current traffic

Deploy Green environment with new version

Switch traffic to Green when ready

Keep Blue as a rollback option

YAML Example with Service Switching
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
    version: green
  ports:
  - port: 80
    targetPort: 80


Switching the selector from version: blue → version: green routes traffic to the new pods.

Best Practices

✅ Use for critical production deployments
✅ Keep both environments fully functional
❌ Avoid keeping idle environments long-term (costly)

Real-world Scenario

A payment gateway updates its API without downtime by switching traffic from blue → green environment.

3️⃣ Canary Deployments

Canary deployments release new versions gradually, routing only a portion of traffic to the new pods.

YAML Example (Istio VirtualService)
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: web-vs
spec:
  hosts:
  - web-app
  http:
  - route:
    - destination:
        host: web-app
        subset: v1
      weight: 90
    - destination:
        host: web-app
        subset: v2
      weight: 10

Best Practices

✅ Start with small percentages (5–20%)
✅ Monitor metrics, logs, and traces
✅ Increase traffic gradually if stable

Real-world Scenario

A recommendation service tests new algorithms with 10% of traffic before scaling to 100%.

4️⃣ A/B Testing

A/B testing is feature-based deployment, routing traffic to different versions to compare performance or user behavior.

Key Points

Often uses canary deployment techniques

Uses metrics like click-through rate or conversion

Can be implemented via Istio VirtualService routing by headers

Example: Header-based Routing
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: web-ab-test
spec:
  hosts:
  - web-app
  http:
  - match:
    - headers:
        user-type:
          exact: beta
    route:
    - destination:
        host: web-app
        subset: v2
  - route:
    - destination:
        host: web-app
        subset: v1

Best Practices

✅ Use A/B testing for UX and feature validation
✅ Track metrics to decide which version wins
❌ Avoid exposing untested features to all users

Real-world Scenario

An e-commerce site shows a new checkout flow to 10% of users to measure conversion improvement.

5️⃣ GitOps Principles

GitOps is a methodology for declarative infrastructure and application deployments via Git repositories.

Key Principles
Principle	Description
Git as Source of Truth	All configs stored in Git
Automated Deployment	CI/CD pipeline applies changes automatically
Observability	Changes in Git are reflected in cluster state
Rollback	Revert changes via Git commit
Popular Tools

ArgoCD

FluxCD

Example: ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/web-app-config.git
    path: manifests
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

Best Practices

✅ Use GitOps for consistent, auditable deployments
✅ Enable automated sync and self-healing
✅ Monitor application state via ArgoCD dashboard

✅ Summary Table
Deployment Strategy	Type	Use Case	Key Tools
Rolling Update	Incremental	Default deployment	kubectl
Blue-Green	Dual environment	Zero-downtime release	Service selector switch
Canary	Gradual rollout	Testing new version	Istio VirtualService
A/B Testing	Feature-based	UX testing	Istio header routing
GitOps	Declarative	CI/CD automation	ArgoCD, FluxCD
Interview Questions

Q: Difference between Rolling Update and Canary deployment?
A: Rolling Update replaces pods gradually without traffic splitting. Canary deployment routes only a portion of traffic to new version.

Q: How do Blue-Green deployments enable fast rollback?
A: The previous environment (Blue) remains intact; switching back is as simple as updating the service selector.

Q: What is GitOps?
A: A declarative deployment model using Git as the single source of truth, with automated sync to Kubernetes clusters.

Q: How can Istio help with A/B testing?
A: Using VirtualService header-based routing to send a subset of users to the new version.

Official Docs:

Kubernetes Deployments

Istio Traffic Management

ArgoCD GitOps
