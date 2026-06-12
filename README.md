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
