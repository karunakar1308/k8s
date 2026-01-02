Part 5: Kubernetes Configuration & Storage

Decoupling configuration, securing secrets, and managing state the Kubernetes way

Kubernetes’ power lies in how it cleanly separates application code, configuration, and data. This separation allows:

Immutable deployments

Secure handling of secrets

Stateful workloads at scale

In this guide, we’ll explore Kubernetes’ approach to:

Application configuration

Sensitive data

Persistent storage

Stateful workloads

1️⃣ ConfigMaps
What is a ConfigMap?

A ConfigMap stores non-sensitive configuration data in key-value format, decoupled from container images.

Use Cases:

Application configs

Environment variables

Property files

Feature flags

⚠️ ConfigMaps are not for secrets—use Secrets instead.

Creating a ConfigMap from YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"

kubectl apply -f configmap.yaml
kubectl get configmap

Using ConfigMaps in Pods

As environment variables:

env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV


As files via volume mount:

volumes:
  - name: config-volume
    configMap:
      name: app-config


Real-World Uses:

✅ Externalize config for multiple environments
✅ Update config without rebuilding images

❌ Don’t store passwords (use Secrets)

2️⃣ Secrets
What is a Secret?

Secrets store sensitive information such as:

Passwords

API keys

Tokens

TLS certificates

Note: Secrets are Base64-encoded by default—not encrypted.

Creating a Secret from YAML
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_USER: YWRtaW4=
  DB_PASSWORD: c2VjdXJl

3️⃣ Persistent Volumes (PV)
What is a Persistent Volume?

A Persistent Volume is a cluster-wide storage resource.

Key Points:

Acts like a disk in the cluster

Independent of Pods

Persists beyond Pod lifecycle

PV Lifecycle

Available → Bound → Released → Deleted

Available: Ready to be claimed

Bound: Attached to a PVC

Released: Claim deleted

Deleted: Storage reclaimed

Example PV YAML
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-manual
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data

kubectl get pv
kubectl describe pv pv-manual


Use Cases:

✅ On-prem clusters
✅ Legacy storage systems

❌ Not ideal for cloud-native dynamic environments

4️⃣ Persistent Volume Claims (PVC)
What is a PVC?

A PVC is a request for storage by a Pod. Pods never talk directly to PVs—they interact through PVCs.

PVC YAML Example
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

kubectl get pvc
kubectl describe pvc app-pvc

Using PVC in a Pod
volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: app-pvc

volumeMounts:
  - mountPath: "/usr/share/nginx/html"
    name: storage


Use Cases:

✅ Databases
✅ File uploads
✅ Logs that survive restarts

5️⃣ Storage Classes
What is a StorageClass?

A StorageClass defines how storage is dynamically provisioned, eliminating the need to manually create PVs—especially in cloud environments.

StorageClass YAML Example (AWS EBS)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard-ebs
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer

kubectl get storageclass

Dynamic PVC with StorageClass
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  storageClassName: standard-ebs
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi


Benefits:

✅ Automatic provisioning
✅ Cloud-native
✅ Scalable

❌ Cloud-provider dependent

6️⃣ StatefulSet Storage
What is a StatefulSet?

StatefulSets manage stateful applications requiring:

Stable network identity

Stable storage

Ordered deployment & scaling

Examples: Databases (MySQL, PostgreSQL), Kafka, Elasticsearch

StatefulSet vs Deployment
Feature	Deployment	StatefulSet
Pod identity	Random	Stable
Storage	Shared	Per-pod
Scaling	Parallel	Ordered
Use case	Stateless apps	Stateful apps
StatefulSet with VolumeClaimTemplates
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
          image: mysql:8
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: DB_PASSWORD
          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 10Gi


Each Pod gets its own disk:

mysql-0 → pvc-mysql-data-mysql-0

mysql-1 → pvc-mysql-data-mysql-1

mysql-2 → pvc-mysql-data-mysql-2

7️⃣ Best Practices

✅ Do This

Use ConfigMaps for non-sensitive configs

Use Secrets for credentials

Use StorageClasses for dynamic provisioning

Use StatefulSets for databases

❌ Avoid This

Hardcoding configs in images

Storing secrets in ConfigMaps

Using hostPath in production

Using Deployments for stateful apps

8️⃣ Interview Questions & Answers

Q1: Difference between PV and PVC?
A: PV is the actual storage resource; PVC is the request made by a Pod.

Q2: Why StorageClasses?
A: Enable dynamic provisioning—no manual PV management.

Q3: Can multiple Pods share a PVC?
A: Only if access mode supports it (e.g., ReadWriteMany).

Q4: Why StatefulSet for databases?
A: Provides stable identity, persistent storage, and ordered deployment.

📌 Official Documentation

ConfigMaps

Secrets

Persistent Volumes

Storage Classes

StatefulSets
