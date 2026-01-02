PART 5: KUBERNETES CONFIGURATION & STORAGE

Decoupling configuration, securing secrets, and managing state the Kubernetes way

One of Kubernetes’ biggest strengths is how cleanly it separates application code, configuration, and data.
This separation enables immutable deployments, secure secret handling, and stateful workloads at scale.

In this section, we’ll cover how Kubernetes manages:

Application configuration

Sensitive data

Persistent storage

Stateful workloads

1️⃣ ConfigMaps
What is a ConfigMap?

A ConfigMap stores non-sensitive configuration data in key-value format, decoupled from container images.

📌 Think of ConfigMaps as:

Application configs

Environment variables

Property files

Feature flags

1.1 Creating a ConfigMap From YAML

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "info"

kubectl apply -f configmap.yaml
kubectl get configmap

1.2 Using ConfigMaps in Pods As Environment Variables

apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV

As Files (Volume Mount)
volumes:
  - name: config-volume
    configMap:
      name: app-config

Real-World Use Case

✅ Externalizing config for multiple environments
✅ Updating config without rebuilding images
❌ Storing passwords (use Secrets instead)

2️⃣ Secrets
What is a Secret?

A Secret stores sensitive information, such as:

Passwords

API keys

Tokens

TLS certificates

📌 Secrets are Base64-encoded, not encrypted by default.

2.1 Creating a Secret From YAML

apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_USER: YWRtaW4=
  DB_PASSWORD:

3️⃣ Persistent Volumes (PV)
What is a Persistent Volume?

A Persistent Volume (PV) is a cluster-wide storage resource provisioned by an administrator or dynamically via a StorageClass.

📌 Think of a PV as:

A disk in the cluster

Independent of Pods

Lives beyond Pod lifecycle

3.1 PV Lifecycle
Available → Bound → Released → Deleted


Available – Ready to be claimed

Bound – Attached to a PVC

Released – Claim deleted

Deleted – Storage reclaimed

3.2 Persistent Volume YAML Example

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

Real-World Use Case

✅ On-prem clusters
✅ Legacy storage systems
❌ Not recommended for cloud-native dynamic environments

4️⃣ Persistent Volume Claims (PVC)
What is a PVC?

A Persistent Volume Claim is a request for storage by a Pod.

📌 Pods do not talk to PVs directly — they talk to PVCs.

4.1 PVC YAML Example

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

4.2 Using PVC in a Pod

apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: storage
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: app-pvc

Real-World Use Case

✅ Databases
✅ File uploads
✅ Logs that must survive restarts

5️⃣ Storage Classes
What is a StorageClass?

A StorageClass defines how storage is dynamically provisioned.

📌 No need to manually create PVs in cloud environments.

5.1 StorageClass YAML Example (AWS EBS)

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

5.2 Dynamic PVC with StorageClass

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

Benefits

✅ Automatic provisioning
✅ Cloud-native
✅ Scalable
❌ Cloud-provider dependent

6️⃣ StatefulSet Storage
What is a StatefulSet?

A StatefulSet manages stateful applications requiring:

Stable network identity

Stable storage

Ordered deployment & scaling

📌 Examples:

Databases (MySQL, PostgreSQL)

Kafka

Elasticsearch

### 6.1 StatefulSet vs Deployment

| Feature      | Deployment     | StatefulSet   |
|-------------|----------------|---------------|
| Pod identity| Random         | Stable        |
| Storage     | Shared         | Per-pod       |
| Scaling     | Parallel       | Ordered       |
| Use case    | Stateless apps | Stateful apps |

6.2 StatefulSet with VolumeClaimTemplates

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

6.3 What Happens Behind the Scenes?

For 3 replicas:

mysql-0 → pvc-mysql-data-mysql-0
mysql-1 → pvc-mysql-data-mysql-1
mysql-2 → pvc-mysql-data-mysql-2


Each Pod gets its own disk — guaranteed.

7️⃣ Best Practices
✅ Do This

✅ Use ConfigMaps for non-sensitive config
✅ Use Secrets for credentials
✅ Use StorageClasses for dynamic provisioning
✅ Use StatefulSets for databases

❌ Avoid This

❌ Hardcoding configs in images
❌ Storing secrets in ConfigMaps
❌ Using hostPath in production
❌ Deployments for stateful apps

8️⃣ Interview Questions & Answers
Q1: Difference between PV and PVC?

A: PV is the actual storage resource, while PVC is a request for storage made by a Pod.

Q2: Why StorageClasses?

A: StorageClasses allow dynamic provisioning of storage, eliminating the need for manually managing PVs.

Q3: Can multiple Pods share a PVC?

A: Only if the access mode supports it (e.g., ReadWriteMany).

Q4: Why StatefulSet for databases?

A: StatefulSets provide stable identity, persistent storage, and ordered deployment required by databases.

📌 Official Documentation

ConfigMaps
https://kubernetes.io/docs/concepts/configuration/configmap/

Secrets
https://kubernetes.io/docs/concepts/configuration/secret/

Persistent Volumes
https://kubernetes.io/docs/concepts/storage/persistent-volumes/

Storage Classes
https://kubernetes.io/docs/concepts/storage/storage-classes/

StatefulSets
https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
