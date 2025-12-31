PART 7: KUBERNETES SCALING & AUTOSCALING

Kubernetes Scaling & Autoscaling: HPA, VPA, Cluster Autoscaler, and KEDA

Scaling is one of Kubernetes’ most powerful features. With proper autoscaling, your workloads can handle spikes in demand efficiently, save resources, and maintain performance. This guide covers all major scaling strategies.

1️⃣ Horizontal Pod Autoscaler (HPA)

HPA automatically scales the number of pod replicas based on metrics like CPU, memory, or custom metrics.

How It Works

Monitors metrics (CPU, memory, custom metrics) for a deployment/replicaset.

Adjusts replicas in the deployment dynamically.

Uses the Metrics API (metrics-server must be installed).

YAML Example
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
  namespace: dev
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50

kubectl Commands
# View HPA status
kubectl get hpa -n dev

# Describe HPA
kubectl describe hpa web-hpa -n dev

# Test scaling manually
kubectl scale deployment web-app --replicas=5 -n dev

Best Practices

✅ Use HPA for workloads with predictable metrics like CPU
✅ Monitor scaling behavior in real-time
❌ Don’t rely solely on HPA for critical workloads

Real-world Scenario

An e-commerce site experiences traffic spikes during flash sales. HPA automatically scales pods up to handle the load and scales down during off-peak hours.

2️⃣ Vertical Pod Autoscaler (VPA)

VPA adjusts resource requests and limits for pods dynamically, unlike HPA which changes replica count.

How It Works

Observes pod resource usage (CPU/memory)

Recommends or automatically updates resource requests

Ideal for workloads with variable memory/CPU usage

YAML Example
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-vpa
  namespace: dev
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: web-app
  updatePolicy:
    updateMode: "Auto"

kubectl Commands
kubectl get vpa -n dev
kubectl describe vpa web-vpa -n dev

Best Practices

✅ Use VPA for stateful workloads or memory-heavy applications
✅ Avoid VPA + HPA on the same deployment for CPU metrics (can conflict)

Real-world Scenario

A data processing app has unpredictable memory usage. VPA ensures pods get enough memory, reducing OOMKilled errors.

3️⃣ Cluster Autoscaler

Cluster Autoscaler automatically adjusts the number of nodes in your cluster based on pod demands.

How It Works

Scales up when pods cannot be scheduled due to resource constraints.

Scales down idle nodes with no critical pods.

Works with cloud providers (AWS, GCP, Azure).

Example: AWS EKS
# Cluster Autoscaler is deployed as a Deployment
# For EKS, you usually annotate your node group:
serviceAccount:
  name: cluster-autoscaler
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/<ClusterAutoscalerRole>

kubectl Commands
kubectl get deployment cluster-autoscaler -n kube-system
kubectl logs deployment/cluster-autoscaler -n kube-system

Best Practices

✅ Configure node group scaling limits
✅ Tag nodes correctly for proper scaling
❌ Don’t forget resource requests/limits on pods (CA uses them to scale nodes)

Real-world Scenario

During peak traffic, HPA scales pods beyond current node capacity. Cluster Autoscaler adds new nodes automatically to schedule these pods.

Official Docs: Cluster Autoscaler

4️⃣ KEDA (Kubernetes Event-Driven Autoscaling)

KEDA allows autoscaling based on external events, not just CPU/memory. It’s perfect for serverless or event-driven workloads.

Key Features

Scales workloads based on metrics from Kafka, RabbitMQ, Azure Queue, Prometheus, etc.

Integrates with HPA seamlessly.

Can scale down to zero.

YAML Example: Scale Deployment Based on Kafka Queue
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: kafka-consumer
  namespace: dev
spec:
  scaleTargetRef:
    name: kafka-consumer-deployment
  minReplicaCount: 1
  maxReplicaCount: 10
  triggers:
  - type: kafka
    metadata:
      bootstrapServers: my-kafka:9092
      consumerGroup: my-group
      topic: my-topic
      lagThreshold: "10"

kubectl Commands
kubectl get scaledobject -n dev
kubectl describe scaledobject kafka-consumer -n dev

Best Practices

✅ Use KEDA for event-driven workloads
✅ Combine with HPA for hybrid metrics
✅ Monitor scaling events for anomalies

Real-world Scenario

A message-processing microservice consumes messages from Kafka. KEDA scales pods dynamically based on queue backlog.

Official Docs: KEDA

✅ Summary Table
Autoscaler	Scales	Metrics	Use Case
HPA	Pods	CPU, Memory, Custom	Web apps with fluctuating load
VPA	Resources (CPU/Memory)	Pod usage	Stateful or memory-heavy workloads
Cluster Autoscaler	Nodes	Pod scheduling & resource requests	Cloud clusters with variable workload
KEDA	Pods	External events (Kafka, Queue, Prometheus)	Event-driven microservices
Interview Questions

Q: Difference between HPA and VPA?
A: HPA scales replicas, VPA scales resources (CPU/memory) per pod.

Q: Can HPA and VPA be used together?
A: Only if they don’t target the same metrics (e.g., HPA CPU + VPA memory).

Q: How does Cluster Autoscaler decide to scale down a node?
A: If all pods on a node can be scheduled elsewhere and the node is idle, CA scales it down.

Q: What’s unique about KEDA?
A: It scales workloads based on external event sources and can scale to zero.

This guide covers all major Kubernetes autoscaling options, with real-world examples and best practices.
