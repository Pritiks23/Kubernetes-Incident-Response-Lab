# Kubernetes-Incident-Response-Lab
Built a Kubernetes troubleshooting lab containing 20 production-inspired outages. Diagnosed and resolved failures involving networking, DNS, ingress, storage, resource limits, configuration management, probes, scheduling, and container lifecycle issues. Authored incident runbooks and root-cause analyses for each scenario.


# Incident 1: Ingress 404 Misrouting

## Summary
Simulated an Ingress routing failure where traffic returned 404 despite a healthy backend service and running pods.

## Initial State (Healthy)
- nginx Deployment running
- ClusterIP Service correctly mapped (port 80 → 80)
- Pod responding on port 80
- Ingress controller installed and functional

## Failure Injected
Modified Ingress path from:
- `/` → `/broken`

## Symptoms
- `/` returned 404 Not Found
- `/broken` returned valid nginx response
- No issues observed at pod or service level

## Root Cause
Incorrect Ingress path configuration in routing rule.

## Detection Method
- kubectl describe ingress
- Direct HTTP testing via debug pod
- Path-based response validation

## Resolution
Restored Ingress rule to:
- `/` → nginx:80

## Outcome
- Full HTTP traffic restored
- End-to-end routing validated (Ingress → Service → Pod)

## Key Learning
Ingress misconfigurations can break routing even when backend services are healthy.


---

# Incident 2: Service Selector Failure (Silent Traffic Outage)

## Summary
Simulated a Kubernetes service outage caused by a Service selector misconfiguration. Pods were healthy, but traffic failed due to missing endpoint resolution.

## Initial State (Healthy)
- nginx Deployment running with label `app=nginx`
- ClusterIP Service correctly selecting `app=nginx`
- Endpoints correctly pointing to Pod IP

## Failure Injected
Modified Service selector:
- from `app=nginx`
- to `app=broken`

## Symptoms
- Service showed zero endpoints
- Internal cluster traffic to `http://nginx/` failed
- Pods remained healthy and running (no CrashLoopBackOff or pod-level errors)

## Root Cause
Service selector mismatch caused Kubernetes to fail endpoint discovery, breaking service-to-pod routing.

## Detection Method
- `kubectl get endpoints nginx` returned empty
- `kubectl describe svc nginx` showed selector mismatch
- HTTP requests from debug pod failed

## Resolution
Restored Service selector:
- `app=broken` → `app=nginx`

## Outcome
- Endpoints restored automatically
- Full service-to-pod traffic flow recovered
- Cluster returned to healthy state

## Key Learning
Service selector misconfigurations can cause silent, non-obvious outages where workloads remain healthy but are completely unreachable.

## 🚨 Incident 3: CrashLoopBackOff (Application Failure)
📌 Summary

This incident simulates a production failure where a Kubernetes Deployment enters a CrashLoopBackOff state due to an invalid container startup command. The issue caused continuous pod restarts and prevented stable application availability.

The root cause was a misconfigured container command in the Deployment spec that forced the container to immediately exit (exit 1).

🧱 Environment
Cluster: kind (Kubernetes v1.36.1)
Deployment: nginx
Service Type: ClusterIP
Namespace: default
Tooling: kubectl, docker, kind
💥 Incident Trigger

The issue was introduced by patching the Deployment with an invalid startup command:

command:
  - sh
  - -c
  - exit 1

This caused all newly created pods to fail immediately after startup.

⚠️ Symptoms Observed
Pod Behavior
Pods repeatedly restarted
Status alternated between:
Error
CrashLoopBackOff
Running (briefly)
Kubernetes Signals
High restart counts
Unstable ReplicaSet rollout
Missing or unstable endpoints
Example:
nginx-xxxxx   0/1   CrashLoopBackOff   Restarting continuously
🔍 Diagnosis Steps
1. Checked pod status
kubectl get pods
2. Inspected failing pod
kubectl describe pod <pod-name>

Key finding:

Exit Code: 1
Reason: Error
3. Checked logs
kubectl logs deployment/nginx

Logs showed no application output due to immediate termination.

4. Verified rollout state
kubectl rollout status deployment/nginx

Observed rollout instability due to failing ReplicaSet.

🧠 Root Cause

The Deployment spec contained an invalid container command:

sh -c exit 1

This forced the container to terminate immediately after startup, triggering Kubernetes restart policies.

Kubernetes behavior:

Detects failure
Restarts container
Enters exponential backoff (CrashLoopBackOff)
🛠️ Resolution
Step 1: Rollback Deployment
kubectl rollout undo deployment/nginx
Step 2: Restart Deployment
kubectl rollout restart deployment/nginx
Step 3: Validate Recovery
kubectl get pods
kubectl get endpoints nginx
✅ Final State
Pods
All pods in Running state
1/1 READY
Endpoints
Service routing restored
Valid pod IPs assigned
Deployment
Stable ReplicaSet active
No restart loops
📊 Evidence Collected
Pod describe output
Deployment YAML (before/after rollback)
Rollout history
Service & endpoint snapshots
Cluster-wide resource state

Stored in:

evidence/incident-3/
📚 Key Learnings
1. CrashLoopBackOff is application-level failure

Kubernetes is functioning correctly; the container itself is failing.

2. Rollback is not enough if spec is still mutated

If Deployment template is incorrect, Kubernetes will continue recreating failures.

3. Debugging requires multiple layers:
Pod lifecycle events (describe)
Deployment rollout state
ReplicaSet reconciliation
Endpoint health
4. Logs may not always be available

Fast-failing containers may not produce usable logs; events become primary signal.
