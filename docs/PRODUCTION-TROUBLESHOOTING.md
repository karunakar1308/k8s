Docker production scenarios
1. Container works locally but fails in Kubernetes
Scenario:
A Node.js app runs fine when you do docker run on your laptop, but in Kubernetes the pod crashes with MODULE_NOT_FOUND for a dependency. The Dockerfile uses COPY . . and npm install without a proper .dockerignore. In CI, extra files (test folders, different lock files) get copied, causing a different dependency tree than your local build.

How to troubleshoot:

Check container image contents in the running pod: kubectl exec -it <pod> -- ls node_modules and compare with local.

Inspect Docker build context size using docker build . output and ensure .dockerignore excludes node_modules, .git, and build artifacts.

Rebuild the image in the same environment as CI to reproduce.

Root cause:
Inconsistent build context and missing .dockerignore meant CI produced a different image than local, leading to missing or mismatched dependencies at runtime.

Fix:
Add a proper .dockerignore, pin dependencies with package-lock.json, and ensure npm ci (or equivalent) runs during image build to make builds deterministic.

2. High CPU usage after Docker upgrade
Scenario:
After upgrading Docker Engine on production nodes, multiple Java containers start consuming high CPU even at low traffic. The application logs show frequent full GCs and thread contention.

How to troubleshoot:

Check container CPU limits and the JVM’s container awareness flags.

Run docker stats and JVM tools (jstat, jcmd) inside the container.

Confirm if the new Docker version changed default CPU quota behavior or cgroup driver.

Root cause:
The JVM was not fully container‑aware and still assumed host CPU count, leading to an aggressive GC thread count and wrong ergonomics once Docker’s cgroup behavior changed.

Fix:
Add JVM options like -XX:+UseContainerSupport, set -XX:MaxRAMPercentage and -XX:ActiveProcessorCount to match container limits, then redeploy.

Kubernetes production scenarios
3. Intermittent 5xx errors due to readiness probes
Scenario:
A stateless API randomly returns 503/502 during deployments under load, even though HPA and resources look fine. Pods restart count is low. Metrics show a spike in failed requests right after new pods come up.

How to troubleshoot:

Check readinessProbe and livenessProbe settings in the Deployment.

Describe pods: kubectl describe pod <pod> and look for probe failures.

Inspect app startup time vs probe initialDelaySeconds and timeoutSeconds.

Root cause:
Readiness probe used a complex health endpoint that depends on external services (DB, third‑party APIs), and timeouts were too aggressive. New pods were marked ready before dependent services were actually usable, then flapped between ready and not‑ready, leading to traffic being routed to unstable pods.

Fix:
Simplify readiness endpoint to only check internal app readiness, increase initialDelaySeconds and timeoutSeconds, and ensure the app signals readiness only after it is truly ready to serve traffic.

4. Node pressure causing random pod evictions
Scenario:
In a production cluster, certain workloads get evicted randomly during traffic spikes, even though cluster CPU utilization is below 60%. Teams see Evicted pods with reasons like Evicted: The node was low on resource: memory.

How to troubleshoot:

Check kubectl describe node <node> for memory pressure and eviction thresholds.

Review pod requests and limits for memory.

Verify if there are “noisy neighbor” pods on affected nodes.

Root cause:
Several pods had no memory limits and some had very low requests, so the scheduler packed them tightly on a few nodes. Under load, these pods exceeded physical memory, triggering kubelet evictions based on node eviction thresholds.

Fix:
Define proper requests and limits for memory for all workloads, use LimitRanges and ResourceQuotas to enforce, then gradually rebalance workloads via a rolling restart or descheduler.

5. Service unavailable due to misconfigured NetworkPolicy
Scenario:
After introducing NetworkPolicies for security hardening, internal calls between microservices start failing with connection timeouts. No deployments changed, only the new NetworkPolicy manifests were applied.

How to troubleshoot:

List policies in the namespace: kubectl get networkpolicy.

Describe the impacted pods and look for matching pod selectors.

Check if there is a default‑deny policy without an allow rule for required traffic.

Root cause:
A default‑deny ingress NetworkPolicy was applied to the namespace, but the allow rules covered only traffic from specific labels and missed system components and some services. As a result, traffic that previously worked got dropped.

Fix:
Add explicit allow policies for required namespaces (e.g., ingress controllers, DNS, monitoring) and for all required app‑to‑app flows, test in a non‑prod environment, then roll out gradually.

6. CrashLoopBackOff after config change
Scenario:
A deployment that was stable for months starts CrashLooping immediately after a config change in ConfigMap. The pod log shows “invalid configuration” or “cannot parse YAML/JSON”.

How to troubleshoot:

Get previous ConfigMap revision from Git/history.

Compare current vs previous config.

Validate the config using the application’s built‑in validation or tools before starting.

Root cause:
ConfigMap was updated with a malformed configuration (e.g., wrong indentation, missing required field), and the application exits on startup validation failure.

Fix:
Revert to the previously working ConfigMap from Git, add linting/validation in CI for config files, and use canary rollouts instead of full rollout for config changes.

ArgoCD production scenarios
7. ArgoCD shows “OutOfSync” but resources look correct
Scenario:
ArgoCD Application status is OutOfSync even though the cluster resources appear correct and the app is serving traffic normally. Syncing again toggles to Synced briefly, then it flips back to OutOfSync without changes in Git.

How to troubleshoot:

Check ArgoCD Application diff in the UI or via CLI.

Look for auto‑generated fields (timestamps, annotations, status) in the live manifest.

Confirm if any controllers (e.g., MutatingWebhook, sidecars) are mutating the manifest.

Root cause:
A mutating admission controller (for example, a sidecar injector, or a service mesh like Istio) added annotations and fields that are not in Git. ArgoCD compares live vs desired and considers those differences as drift.

Fix:
Configure ArgoCD’s resource.customizations or ignoreDifferences to ignore specific paths (annotations, status fields) for those resources, or adjust the mutating webhook behavior if possible.

8. ArgoCD fails to sync due to RBAC in target cluster
Scenario:
New application onboarded via ArgoCD fails with sync errors like “forbidden: User ‘system:serviceaccount:argocd:argocd-application-controller’ cannot create resource X in namespace Y”.

How to troubleshoot:

Check ArgoCD project settings for the app and the service account used.

Inspect RBAC roles and rolebindings in the target cluster for the ArgoCD service account.

Reproduce with kubectl auth can-i --as=system:serviceaccount:argocd:argocd-application-controller create deployment -n <ns>.

Root cause:
ArgoCD’s service account in the target cluster lacks permissions for the new namespaces or resource types (e.g., CRDs like IngressRoute, PrometheusRule). The Application was added before RBAC was updated.

Fix:
Update ClusterRole/Role and bindings to grant required verbs on the necessary API groups and namespaces, following the least privilege principle, then trigger a resync.

9. Git commit reverted automatically after manual hotfix
Scenario:
During a Sev‑1 incident, an engineer applies a manual hotfix using kubectl apply directly in production to quickly unblock traffic. A few minutes later, ArgoCD syncs and silently reverts the hotfix to match Git, reintroducing the issue.

How to troubleshoot:

Check ArgoCD Application sync policy (auto vs manual).

Review the Git history around the time of the incident.

Inspect ArgoCD audit logs to see who/what triggered the sync.

Root cause:
ArgoCD is configured with automated sync and pruning. Manual in‑cluster changes are treated as drift and are overwritten to match Git, so the hotfix got rolled back.

Fix:
Update incident procedures: either pause auto‑sync for affected apps during incidents, or make hotfixes by committing to Git and letting ArgoCD sync them. Optionally use ArgoCD’s “allow empty” or “selfHeal” flags appropriately.

10. ArgoCD Application stuck in Progressing due to health check
Scenario:
ArgoCD shows an Application in Progressing state for a long time, even though the Kubernetes Deployment shows all pods ready and traffic is flowing. Pipelines wait on ArgoCD health status and seem hung.

How to troubleshoot:

Check the health status at resource level in ArgoCD.

See if any child resource (e.g., CustomResource of a service mesh, StatefulSet) is reported as Degraded or Unknown.

Verify health check configuration for that CRD in ArgoCD.

Root cause:
ArgoCD’s default health check for a custom resource either does not exist or is misconfigured, so ArgoCD cannot determine “Healthy”, leaving the overall app status as Progressing.

Fix:
Define a custom health check Lua script in ArgoCD for that CRD (in resource.customizations.health.<group_kind>), or adjust the resource expectations so ArgoCD recognizes when it is healthy.
