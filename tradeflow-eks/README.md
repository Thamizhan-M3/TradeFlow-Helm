# TradeFlow EKS Demo Overlay

This folder contains the isolated EKS/GitOps overlay for running the old TradeFlow application in the existing M3 Music EKS cluster.

## Scope

- Namespace: `tradeflow`
- Helm release: `tradeflow-eks`
- Argo CD Application: `tradeflow-eks-demo`
- Deployment model: standard Kubernetes `Deployment`
- Services: `ClusterIP`
- MongoDB: in-cluster, single replica, EBS-backed PVC through the cluster default storage class

## Exclusions

This demo does not use:

- M3 Music Terraform changes
- M3 Music Helm changes
- namespace `dev-m3`
- NFS
- Kyverno
- old HAProxy
- NodePort
- hardcoded worker IP routing
- Argo Rollouts
- Gateway API or HTTPRoute
- cluster-scoped TradeFlow resources

## GitOps Flow

1. Commit this folder and the TradeFlow chart changes.
2. Configure the placeholder `repoURL` and `targetRevision` in `argocd-application.yaml`.
3. Let Argo CD sync `tradeflow-eks-demo`.
4. Argo CD creates namespace `tradeflow` and installs Helm release `tradeflow-eks`.

No `kubectl apply`, `helm install`, or live manual deployment step is required for the final path.

## Rollback

Disable or delete Argo CD Application `tradeflow-eks-demo` and let Argo CD prune its managed resources. Delete namespace `tradeflow` only after confirming it contains only TradeFlow demo resources.
