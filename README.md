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
