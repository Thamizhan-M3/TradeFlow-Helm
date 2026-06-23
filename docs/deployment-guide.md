# TradeFlow Deployment Guide

This guide details how to deploy the TradeFlow system using the Helm charts provided in this repository.

## Prerequisites
- A running Kubernetes cluster.
- `helm` CLI installed.
- `kubectl` CLI installed and configured.
- NFS Server setup (see `nfs/README.md`) and NFS CSI Driver installed in the cluster.
- Kyverno installed for admission control (optional but recommended to enforce security policies).

## Step 1: Cluster Setup

First, apply the baseline Kubernetes manifests:

```bash
kubectl apply -f k8s/namespaces.yaml
kubectl apply -f k8s/storageclass-nfs.yaml
kubectl apply -f k8s/admission-controller.yaml
```

## Step 2: Deploy to Development Environment

The repository utilizes a parent chart `tradeflow` which manages all child microservices.

```bash
helm upgrade --install tradeflow ./helm/tradeflow \
  --namespace tradeflow-dev \
  -f ./helm/tradeflow/values.yaml \
  -f ./helm/tradeflow/values-dev.yaml
```

## Step 3: Deploy to Production Environment

For production deployment, use the `values-prod.yaml` override:

```bash
helm upgrade --install tradeflow ./helm/tradeflow \
  --namespace tradeflow-prod \
  -f ./helm/tradeflow/values.yaml \
  -f ./helm/tradeflow/values-prod.yaml
```

## Validation

Verify all pods are running and probes are passing:
```bash
kubectl get pods -n tradeflow-dev
kubectl get hpa -n tradeflow-dev
kubectl get statefulset -n tradeflow-dev
```

To access the frontend, locate the HAProxy LoadBalancer IP/Port:
```bash
kubectl get svc -n tradeflow-dev
```
