# Production Troubleshooting Scenarios

This document contains scenario-based production issues and troubleshooting examples for Docker, Kubernetes, and ArgoCD. The format matches the structure used in the rest of this repository, with clear headings, numbered sections, and clean markdown presentation.

## Docker Production Scenarios

### 1. **Container works locally but fails in Kubernetes**

**Scenario:**  
A Node.js application runs successfully with `docker run` on a developer laptop, but when deployed to Kubernetes, the pod goes into `CrashLoopBackOff` with `MODULE_NOT_FOUND` errors. The Dockerfile uses `COPY . .` and `npm install` without a proper `.dockerignore`. In CI, extra files such as local test artifacts and multiple lock files are copied into the image, which changes the dependency tree.

**How to troubleshoot:**
- Check pod logs using `kubectl logs <pod-name>`
- Inspect image contents using `kubectl exec -it <pod-name> -- ls -l`
- Review the Dockerfile and `.dockerignore`
- Rebuild the image in the same CI environment to reproduce the issue

**Root cause:**  
The image built in CI is different from the image tested locally because unnecessary files were included in the build context.

**Fix:**  
Create a proper `.dockerignore`, use `npm ci` instead of `npm install`, and ensure the image build is deterministic across local and CI environments.

### 2. **High CPU usage after Docker Engine upgrade**

**Scenario:**  
After a Docker Engine upgrade on production worker nodes, Java-based applications begin consuming significantly more CPU even though traffic remains normal. Logs show frequent garbage collection cycles and thread contention.

**How to troubleshoot:**
- Check container resource usage with `docker stats`
- Verify JVM container-awareness settings
- Compare Docker Engine and cgroup behavior before and after the upgrade
- Inspect GC logs and thread dumps

**Root cause:**  
The JVM was not fully aligned with container CPU limits and assumed host-level CPU availability, which caused inefficient garbage collection and thread usage.

**Fix:**  
Configure JVM options such as `-XX:+UseContainerSupport`, `-XX:MaxRAMPercentage`, and `-XX:ActiveProcessorCount` to match container resource limits.

### 3. **Image pull failures during production deployment**

**Scenario:**  
A new release is deployed successfully in staging but fails in production with `ImagePullBackOff`. The Kubernetes manifests are correct, and the image exists in the container registry.

**How to troubleshoot:**
- Run `kubectl describe pod <pod-name>` and inspect image pull events
- Verify image tag spelling and registry URL
- Check whether the imagePullSecret exists in the namespace
- Confirm registry permissions for the production cluster

**Root cause:**  
The production namespace is missing the required image pull secret, or the service account is not configured to use it.

**Fix:**  
Create the correct `imagePullSecret`, attach it to the service account or deployment, and standardize registry access configuration across environments.

## Kubernetes Production Scenarios

### 4. **Intermittent 5xx errors due to readiness probe issues**

**Scenario:**  
A stateless API starts returning intermittent 502 and 503 errors during rolling deployments. Resource usage looks healthy, and pods eventually become stable, but there is a clear error spike during rollout.

**How to troubleshoot:**
- Check pod events with `kubectl describe pod <pod-name>`
- Review readiness and liveness probe configuration
- Compare application startup time against probe timings
- Verify whether the readiness endpoint depends on external services

**Root cause:**  
The readiness probe is too aggressive or relies on downstream systems, so pods are marked ready before they can reliably handle traffic.

**Fix:**  
Use a lightweight internal readiness endpoint, tune `initialDelaySeconds`, `timeoutSeconds`, and `failureThreshold`, and ensure readiness reflects actual serving capability.

### 5. **Pod evictions caused by node memory pressure**

**Scenario:**  
During traffic spikes, some production pods are evicted even though average cluster CPU utilization is low. Teams notice `Evicted` pods with messages indicating low memory conditions on nodes.

**How to troubleshoot:**
- Check node conditions using `kubectl describe node <node-name>`
- Review pod memory `requests` and `limits`
- Identify overcommitted nodes and noisy neighbor workloads
- Inspect kubelet eviction thresholds

**Root cause:**  
Several workloads have no memory limits or unrealistic requests, causing poor pod placement and node-level memory exhaustion during peak traffic.

**Fix:**  
Set realistic memory requests and limits, enforce policies with `LimitRange` and `ResourceQuota`, and rebalance workloads across nodes.

### 6. **Internal traffic breaks after NetworkPolicy changes**

**Scenario:**  
After security hardening, services in the same cluster begin timing out when calling each other. No application code changed, but a new set of `NetworkPolicy` manifests was recently applied.

**How to troubleshoot:**
- List all policies with `kubectl get networkpolicy -A`
- Review namespace and pod selectors carefully
- Check whether a default-deny policy was introduced
- Validate whether DNS, ingress, and monitoring traffic are still allowed

**Root cause:**  
A default-deny policy was applied without complete allow rules for all required internal communication paths.

**Fix:**  
Add explicit allow policies for required service-to-service traffic, DNS, ingress controllers, and observability components. Test policy behavior in lower environments before production rollout.

### 7. **CrashLoopBackOff after ConfigMap update**

**Scenario:**  
A deployment that was stable for months suddenly starts crashing right after a ConfigMap update. Logs show configuration parsing failures or missing required parameters.

**How to troubleshoot:**
- Compare the new ConfigMap with the previous version from Git
- Inspect pod logs for parsing or validation errors
- Validate the config file syntax before deployment
- Confirm whether the application requires a restart to consume new configuration

**Root cause:**  
The updated ConfigMap contains invalid YAML, JSON, or application-level configuration values that cause startup validation to fail.

**Fix:**  
Revert to the last known good configuration, add validation checks in CI, and roll out configuration changes gradually instead of updating all pods at once.

## ArgoCD Production Scenarios

### 8. **Application shows OutOfSync even when resources look correct**

**Scenario:**  
ArgoCD reports an application as `OutOfSync`, but the workloads are healthy and traffic is normal. Running sync fixes it temporarily, but the application soon returns to `OutOfSync`.

**How to troubleshoot:**
- Open the diff view in ArgoCD
- Compare desired and live manifests
- Check for auto-injected annotations, labels, or sidecars
- Review mutating webhooks and service mesh behavior

**Root cause:**  
A mutating admission controller modifies the live resource after deployment, so ArgoCD keeps detecting drift between Git and the cluster.

**Fix:**  
Use ArgoCD `ignoreDifferences` or resource customizations for expected mutated fields, or adjust the mutating webhook behavior if appropriate.

### 9. **Sync failure due to missing RBAC permissions**

**Scenario:**  
A newly onboarded ArgoCD application fails to sync with errors showing that the ArgoCD application controller service account is forbidden from creating or updating resources in the target namespace.

**How to troubleshoot:**
- Review ArgoCD application sync error details
- Check ClusterRole, Role, RoleBinding, and ClusterRoleBinding configuration
- Use `kubectl auth can-i` to validate access for the ArgoCD service account
- Verify namespace-scoped versus cluster-scoped permissions

**Root cause:**  
The ArgoCD controller service account does not have the required permissions for the resource types or namespaces defined in the application.

**Fix:**  
Update Role or ClusterRole definitions and bindings to grant the required permissions while maintaining least privilege.

### 10. **Git commit reverted automatically after manual hotfix**

**Scenario:**  
During a Sev-1 incident, an engineer applies a manual hotfix directly in production using `kubectl apply` to restore service quickly. A few minutes later, ArgoCD reverts the change and the original issue returns.

**How to troubleshoot:**
- Check whether automated sync is enabled in ArgoCD
- Review ArgoCD audit logs and sync history
- Compare the manual cluster change against the Git repository state
- Inspect prune and self-heal settings

**Root cause:**  
ArgoCD treats direct cluster changes as configuration drift and automatically reconciles the cluster back to the state stored in Git.

**Fix:**  
Pause auto-sync during emergency changes, or apply the hotfix through Git so ArgoCD reconciles to the intended updated state.

### 11. **Application stuck in Progressing due to health checks**

**Scenario:**  
An ArgoCD application remains in `Progressing` state for a long time even though Kubernetes shows all pods as ready and the application is serving traffic correctly.

**How to troubleshoot:**
- Inspect resource-level health status in ArgoCD
- Identify whether a child resource is marked `Unknown` or `Degraded`
- Check whether the application uses a Custom Resource Definition
- Review ArgoCD custom health check settings

**Root cause:**  
ArgoCD does not know how to evaluate health for a custom resource, so the application never transitions fully to `Healthy`.

**Fix:**  
Add a custom health check for the CRD in ArgoCD resource customizations, or adjust the resource configuration so its health can be evaluated correctly.

## How to Use This File

This file is intended for interview preparation, production support learning, and real-world troubleshooting practice. Each scenario is written in a structured format so it can be quickly reviewed and expanded later with commands, YAML examples, and environment-specific notes.
