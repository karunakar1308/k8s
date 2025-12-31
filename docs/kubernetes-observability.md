Kubernetes Observability: Logging, Monitoring, Metrics & Tracing

Kubernetes makes deploying applications easy—but observing them in production is equally critical. Observability ensures you can monitor health, detect anomalies, and troubleshoot issues efficiently. This guide covers all key observability practices in Kubernetes.

1️⃣ Logging

Logs capture what happens inside your cluster and applications. Centralized logging is essential for debugging and auditing.

Popular Tools
Tool	Use Case
Fluentd	Log collector and aggregator
Fluent Bit	Lightweight log forwarder
EFK Stack (Elasticsearch + Fluentd/Bit + Kibana)	Centralized logging, search, and visualization
Example: Fluent Bit DaemonSet
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: fluent-bit
  template:
    metadata:
      labels:
        app: fluent-bit
    spec:
      serviceAccountName: fluent-bit
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:2.2
        resources:
          limits:
            memory: 200Mi
            cpu: 200m
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log

kubectl Commands
# Check logs for a pod
kubectl logs <pod-name> -n <namespace>

# Tail logs for all pods with a label
kubectl logs -l app=web -n dev --follow

Best Practices

✅ Centralize logs to EFK or Loki
✅ Set retention policies to avoid storage overload
✅ Use structured logging (JSON) for easier parsing

2️⃣ Monitoring

Monitoring tracks cluster and application metrics, enabling proactive alerting.

Popular Tools
Tool	Purpose
Prometheus	Metrics collection and storage
Grafana	Metrics visualization and dashboards
Example: Prometheus Deployment
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: web-app-monitor
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: web
  endpoints:
  - port: http
    interval: 30s

kubectl Commands
# Check Prometheus targets
kubectl get servicemonitor -n monitoring

# View metrics
kubectl port-forward svc/prometheus-k8s 9090:9090 -n monitoring

Best Practices

✅ Monitor CPU, memory, disk, and custom app metrics
✅ Set up alerts for critical metrics
✅ Visualize dashboards in Grafana for operational awareness

3️⃣ Metrics Server

Metrics Server provides resource metrics (CPU/memory) for HPA and ad-hoc queries.

Deployment
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl Commands
# Check node and pod metrics
kubectl top nodes
kubectl top pods -n dev

Best Practices

✅ Metrics Server is lightweight; use it for autoscaling and quick insights
✅ For long-term storage, integrate Prometheus

4️⃣ Distributed Tracing

Distributed tracing helps track requests across multiple microservices.

Popular Tools
Tool	Description
Jaeger	Tracing system for monitoring and troubleshooting
Zipkin	Distributed tracing for microservices
Example: Jaeger All-in-One Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger
  namespace: observability
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jaeger
  template:
    metadata:
      labels:
        app: jaeger
    spec:
      containers:
      - name: jaeger
        image: jaegertracing/all-in-one:1.36
        ports:
        - containerPort: 16686

kubectl Commands
kubectl port-forward deployment/jaeger 16686:16686 -n observability


Open http://localhost:16686 to view traces.

Best Practices

✅ Trace high-latency requests to detect bottlenecks
✅ Use sampling to avoid overload
✅ Tag traces with user/context information

5️⃣ Debugging Pods & Troubleshooting

Debugging is inevitable. Kubernetes provides several built-in tools.

Useful Commands
# Inspect pod details
kubectl describe pod <pod-name> -n <namespace>

# Open interactive shell in a pod
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash

# Copy files from pod
kubectl cp <pod-name>:/path/to/file ./local-path -n <namespace>

# Get events in namespace
kubectl get events -n dev --sort-by='.lastTimestamp'

# View node conditions
kubectl describe node <node-name>

Real-world Scenarios

CrashLoopBackOff: Inspect logs, check resource limits, or misconfigurations.

Pod not scheduled: Check node resources, taints, tolerations, and Network Policies.

High latency: Use Prometheus + Jaeger to identify bottlenecks.

Best Practices

✅ Always check kubectl describe pod and events first
✅ Use sidecar containers for logging/debugging if needed
✅ Set resource requests and limits to prevent node starvation

✅ Summary Table
Observability Area	Tool/Method	Purpose
Logging	Fluentd, Fluent Bit, EFK	Centralize and analyze logs
Monitoring	Prometheus, Grafana	Track metrics, create dashboards
Metrics	Metrics Server	Provides CPU/Memory metrics for HPA
Tracing	Jaeger, Zipkin	Track requests across microservices
Debugging	kubectl exec/logs/describe	Troubleshoot pod and cluster issues
Interview Questions

Q: Difference between Metrics Server and Prometheus?
A: Metrics Server is lightweight for short-term metrics (HPA), Prometheus stores long-term metrics with query/alert capabilities.

Q: How do you debug a CrashLoopBackOff pod?
A: Check kubectl describe pod, inspect logs with kubectl logs, verify resource requests/limits, and check events.

Q: Why use distributed tracing?
A: To trace requests across microservices, identify bottlenecks, and troubleshoot performance issues.

Official Docs:

Kubernetes Logging

Prometheus

Metrics Server

Jaeger
