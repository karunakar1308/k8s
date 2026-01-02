# PART 7: KUBERNETES SCALING & AUTOSCALING

## Overview

**Kubernetes Scaling & Autoscaling: HPA, VPA, Cluster Autoscaler, and KEDA**  

Scaling is one of Kubernetes’ most powerful features. With proper autoscaling, workloads can handle demand spikes efficiently, save resources, and maintain performance. This guide covers the major scaling strategies.

---

## 1. Horizontal Pod Autoscaler (HPA)

HPA automatically scales the number of pod replicas based on metrics like CPU, memory, or custom metrics.

### How It Works

- Monitors metrics (CPU, memory, custom metrics) for a Deployment/ReplicaSet.  
- Adjusts replicas in the Deployment dynamically.  
- Uses the Metrics API (metrics-server must be installed).

### YAML Example

```yaml
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
```

### kubectl Commands

```bash
# View HPA status
kubectl get hpa -n dev

# Describe HPA
kubectl describe hpa web-hpa -n dev

# Test scaling manually
kubectl scale deployment web-app --replicas=5 -n dev
```

### Best Practices

- ✅ Use HPA for workloads with predictable metrics like CPU.  
- ✅ Monitor scaling behavior in real time.  
- ❌ Do not rely solely on HPA for mission-critical workloads.

### Real-world Scenario

An e-commerce site sees traffic spikes during flash sales. HPA automatically scales pods up to handle the load and scales down during off-peak hours. 

---

## 2. Vertical Pod Autoscaler (VPA)

VPA adjusts resource requests and limits for pods dynamically, unlike HPA which changes replica count. 

### How It Works

- Observes pod resource usage (CPU/memory).  
- Recommends or automatically updates resource requests.  
- Ideal for workloads with variable memory/CPU usage. 

### YAML Example

```yaml
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
``` 

### kubectl Commands

```bash
kubectl get vpa -n dev
kubectl describe vpa web-vpa -n dev
``` 

### Best Practices

- ✅ Use VPA for stateful or memory-heavy applications.  
- ✅ Use VPA for workloads with unpredictable resource usage.  
- ⚠️ Avoid using VPA and HPA on the same Deployment for CPU metrics (they can conflict). 

### Real-world Scenario

A data-processing app has unpredictable memory usage. VPA ensures pods receive sufficient memory, reducing OOMKilled events. 

---

## 3. Cluster Autoscaler

Cluster Autoscaler (CA) automatically adjusts the number of nodes in a cluster based on pod demands. 

### How It Works

- Scales up when pods cannot be scheduled due to insufficient resources.  
- Scales down nodes that are underutilized and do not run critical pods.  
- Integrates with major cloud providers (AWS, GCP, Azure). 

### Example: AWS EKS

Cluster Autoscaler is typically deployed as a Deployment in `kube-system`, and node groups are annotated: 

```yaml
serviceAccount:
  name: cluster-autoscaler
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/<ClusterAutoscalerRole>
``` 

### kubectl Commands

```bash
kubectl get deployment cluster-autoscaler -n kube-system
kubectl logs deployment/cluster-autoscaler -n kube-system
``` 

### Best Practices

- ✅ Configure node group min/max scaling limits.  
- ✅ Ensure nodes are tagged correctly for CA.  
- ❌ Do not forget resource requests/limits on pods (CA uses them for scheduling decisions).

### Real-world Scenario

During peak traffic, HPA increases pod replicas beyond current node capacity. Cluster Autoscaler adds new nodes to schedule these additional pods.

Official docs: Cluster Autoscaler (provider-specific guides for EKS, GKE, AKS).

---

## 4. KEDA (Kubernetes Event-Driven Autoscaling)

KEDA enables autoscaling based on external events, not just CPU/memory, making it ideal for event-driven or serverless-style workloads.

### Key Features

- Scales workloads based on metrics from Kafka, RabbitMQ, Azure Queue, Prometheus, and more.  
- Integrates with HPA to drive pod scaling.  
- Can scale deployments down to zero replicas.

### YAML Example: Kafka-based Scaling

```yaml
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
```

### kubectl Commands

```bash
kubectl get scaledobject -n dev
kubectl describe scaledobject kafka-consumer -n dev
```

### Best Practices

- ✅ Use KEDA for event-driven workloads (queues, streams, events).  
- ✅ Combine with HPA for hybrid metric-based scaling.  
- ✅ Monitor scaling events and queue backlogs carefully.

### Real-world Scenario

A message-processing microservice consumes messages from Kafka. KEDA scales pods based on queue lag, scaling up when backlog grows and down when it drains.

Official docs: https://keda.sh/

---

## Summary Table

```md
| Autoscaler         | Scales         | Metrics                               | Use Case                                   |
|--------------------|---------------|----------------------------------------|--------------------------------------------|
| HPA                | Pods          | CPU, memory, custom metrics            | Web apps with fluctuating load             |
| VPA                | Resources     | Pod resource usage (CPU/memory)        | Stateful or memory-heavy workloads         |
| Cluster Autoscaler | Nodes         | Pod scheduling & resource requests     | Cloud clusters with variable workloads     |
| KEDA               | Pods          | External events (Kafka, queues, etc.)  | Event-driven microservices                 |
```

---

## Interview Questions

**Q:** Difference between HPA and VPA?  
**A:** HPA scales the number of replicas, while VPA scales resource requests and limits (CPU/memory) per pod. 

**Q:** Can HPA and VPA be used together?  
**A:** Yes, but only if they do not target the same metric (for example, HPA on CPU and VPA on memory). 

**Q:** How does Cluster Autoscaler decide to scale down a node?  
**A:** If all pods on a node can be rescheduled elsewhere and the node is not running critical pods, the node can be removed.

**Q:** What’s unique about KEDA?  
**A:** It scales workloads based on external event sources (for example, Kafka, queues, Prometheus) and can scale down to zero replicas.
