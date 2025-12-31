Part3: Kubernetes Workloads – A Practical, Interview-Ready Guide

Kubernetes workloads are the core building blocks that define how your applications run, scale, heal, and evolve inside a cluster. Whether you’re deploying a stateless web app, a stateful database, a background job, or an agent running on every node — Kubernetes has a workload type designed for it.

This guide covers Pods, ReplicaSets, Deployments, StatefulSets, DaemonSets, Jobs, and CronJobs with:

Real-world scenarios

YAML examples

CLI commands

Interview questions

Best practices (✅/❌)

Official documentation links

What Are Kubernetes Workloads?

A workload in Kubernetes represents an application or task running on the cluster. Workloads describe:

What to run (container image)

How many instances

How they scale and recover

Where they run

Common workload resources:

Pod

ReplicaSet

Deployment

StatefulSet

DaemonSet

Job / CronJob

📘 Official Docs:
https://kubernetes.io/docs/concepts/workloads/

Pods – The Smallest Deployable Unit

A Pod is the smallest unit in Kubernetes. It can run:

One container (most common)

Multiple tightly coupled containers (sidecars)

Key Characteristics

Pods are ephemeral

Each Pod has one IP

Containers in a Pod share:

Network namespace

Volumes

Example Pod YAML
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80

Run & Inspect a Pod
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod nginx-pod
kubectl logs nginx-pod

Real-World Use Case

Debugging

One-off experiments

Learning Kubernetes basics

⚠️ Pods are rarely used directly in production

Best Practices

✅ Use Pods only for learning or debugging
❌ Do not manage Pods manually in production
❌ Do not rely on Pods for self-healing

Interview Q&A

Q: Why are Pods not used directly in production?
A: Pods don’t self-heal or scale. If a Pod dies, it’s gone unless managed by a higher-level controller.

📘 Docs: https://kubernetes.io/docs/concepts/workloads/pods/

ReplicaSets – Ensuring Desired State

A ReplicaSet ensures a specified number of identical Pods are always running.

What ReplicaSets Do

Maintain desired Pod count

Replace failed Pods automatically

Use label selectors

ReplicaSet YAML Example
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
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
        image: nginx:1.25

Commands
kubectl apply -f rs.yaml
kubectl get rs
kubectl get pods -l app=web

Real-World Use Case

Foundation for Deployments

Rarely used directly

Best Practices

✅ Let Deployments manage ReplicaSets
❌ Do not update ReplicaSets manually
❌ Avoid direct production usage

Interview Q&A

Q: Can ReplicaSets perform rolling updates?
A: No. ReplicaSets only maintain pod count. Rolling updates are handled by Deployments.

📘 Docs: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

Deployments – Production Standard for Stateless Apps

A Deployment is the most common workload for stateless applications.

What Deployments Provide

Rolling updates

Rollbacks

Scaling

Self-healing

Deployment YAML Example
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
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
        image: nginx:1.25
        ports:
        - containerPort: 80

Common Deployment Commands
kubectl apply -f deployment.yaml
kubectl rollout status deployment/nginx-deployment
kubectl scale deployment nginx-deployment --replicas=5
kubectl rollout undo deployment/nginx-deployment

Real-World Use Case

Web applications

APIs

Microservices

Best Practices

✅ Use Deployments for stateless apps
✅ Define resource requests/limits
❌ Don’t use Deployments for databases

Interview Q&A

Q: What happens during a rolling update?
A: Kubernetes gradually replaces old Pods with new ones, maintaining availability.

📘 Docs: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

StatefulSets – Managing Stateful Applications

StatefulSets are designed for applications that need:

Stable network identity

Stable storage

Ordered deployment

Key Features

Predictable Pod names (app-0, app-1)

Persistent Volumes per Pod

Ordered scaling and termination

StatefulSet YAML Example
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi

Real-World Use Case

Databases (MySQL, PostgreSQL)

Kafka

Elasticsearch

Best Practices

✅ Use StatefulSets for databases
✅ Pair with Headless Services
❌ Don’t treat StatefulSets like Deployments

Interview Q&A

Q: StatefulSet vs Deployment?
A: Deployments are for stateless apps; StatefulSets preserve identity and storage.

📘 Docs: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

DaemonSets – One Pod per Node

A DaemonSet ensures a Pod runs on every node (or selected nodes).

Common Use Cases

Log collectors

Monitoring agents

Security scanners

DaemonSet YAML Example
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd

Commands
kubectl get daemonsets
kubectl describe daemonset fluentd

Best Practices

✅ Use node-level agents
❌ Don’t use DaemonSets for app workloads

Interview Q&A

Q: Does DaemonSet respect node autoscaling?
A: Yes, new nodes automatically get DaemonSet Pods.

📘 Docs: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

Jobs & CronJobs – Run-to-Completion Tasks
Jobs

A Job runs a task until completion.

apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  completions: 1
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: job
        image: busybox
        command: ["echo", "Hello Kubernetes"]

CronJobs

A CronJob runs Jobs on a schedule.

apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: busybox
            command: ["echo", "Running backup"]

Real-World Use Cases

Database backups

Report generation

Cleanup tasks

Best Practices

✅ Use Jobs for batch processing
✅ Use CronJobs for scheduled automation
❌ Don’t run long-running services as Jobs

Interview Q&A

Q: Job vs CronJob?
A: Job runs once; CronJob runs on a schedule.

📘 Docs:
https://kubernetes.io/docs/concepts/workloads/controllers/job/

https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

Final Comparison Summary
Workload	Use Case
Pod	Basic execution unit
ReplicaSet	Pod replication
Deployment	Stateless apps
StatefulSet	Stateful apps
DaemonSet	Node-level agents
Job	One-time tasks
CronJob	Scheduled tasks
Final Interview Tip 🎯

“In Kubernetes, you almost never deploy Pods directly — you deploy controllers that manage Pods for you.”
