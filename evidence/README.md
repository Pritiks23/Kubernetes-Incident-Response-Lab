# Kubernetes Incident Response Lab - Snapshot

## Cluster
- Kind cluster running in GitHub Codespaces
- Single-node control plane

## Installed Components
- nginx deployment
- ClusterIP service
- ingress-nginx controller

## Current State
- All services healthy
- No active incidents
- Baseline system validated via direct pod + service + ingress tests

## Verification Methods Used
- kubectl get pods
- kubectl get svc
- kubectl get ingress
- curl/wget via debug pod
- ingress controller logs

## Purpose
This snapshot represents a known-good baseline state before simulated incident injection.
^X

