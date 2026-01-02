# PART 7: KUBERNETES SCALING & AUTOSCALING

## Overview

**Kubernetes Scaling & Autoscaling: HPA, VPA, Cluster Autoscaler, and KEDA**  

Scaling is one of Kubernetes’ most powerful features. With proper autoscaling, workloads can handle demand spikes efficiently, save resources, and maintain performance. This guide covers the major scaling strategies. [page:6]

---

## 1. Horizontal Pod Autoscaler (HPA)

HPA automatically scales the number of pod replicas based on metrics like CPU, memory, or custom metrics. [page:6]

### How It Works

- Monitors metrics (CPU, memory, custom metrics) for a Deployment/ReplicaSet.  
- Adjusts replicas in the Deployment dynamically.  
- Uses the Metrics API (metrics-server must be installed). [page:6]

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
``` [page:6]

### kubectl Commands

```bash
# View HPA status
kubectl get hpa -n dev

# Describe HPA
kubectl describe hpa web-hpa -n dev

# Test scaling manually
kubectl scale deployment web-app --replicas=5 -n dev
``` [page:6]

### Best Practices

- ✅ Use HPA for workloads with predictable metrics like CPU.  
- ✅ Monitor scaling behavior in real time.  
- ❌ Do not rely solely on HPA for mission-critical workloads. [page:6]

### Real-world Scenario

An e-commerce site sees traffic spikes during flash sales. HPA automatically scales pods up to handle the load and scales down during off-peak hours. [page:6]

---

## 2. Vertical Pod Autoscaler (VPA)

VPA adjusts resource requests and limits for pods dynamically, unlike HPA which changes replica count. [page:6]

### How It Works

- Observes pod resource usage (CPU/memory).  
- Recommends or automatically updates resource requests.  
- Ideal for workloads with variable memory/CPU usage. [page:6]

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
``` [page:6]

### kubectl Commands

```bash
kubectl get vpa -n dev
kubectl describe vpa web-vpa -n dev
``` [page:6]

### Best Practices

- ✅ Use VPA for stateful or memory-heavy applications.  
- ✅ Use VPA for workloads with unpredictable resource usage.  
- ⚠️ Avoid using VPA and HPA on the same Deployment for CPU metrics (they can conflict). [page:6]

### Real-world Scenario

A data-processing app has unpredictable memory usage. VPA ensures pods receive sufficient memory, reducing OOMKilled events. [page:6]

---

## 3. Cluster Autoscaler

Cluster Autoscaler (CA) automatically adjusts the number of nodes in a cluster based on pod demands. [page:6]

### How It Works

- Scales up when pods cannot be scheduled due to insufficient resources.  
- Scales down nodes that are underutilized and do not run critical pods.  
- Integrates with major cloud providers (AWS, GCP, Azure). [page:6]

### Example: AWS EKS

Cluster Autoscaler is typically deployed as a Deployment in `kube-system`, and node groups are annotated: [page:6]

```yaml
serviceAccount:
  name: cluster-autoscaler
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/<ClusterAutoscalerRole>
``` [page:6]

### kubectl Commands

```bash
kubectl get deployment cluster-autoscaler -n kube-system
kubectl logs deployment/cluster-autoscaler -n kube-system
``` [page:6]

### Best Practices

- ✅ Confi
