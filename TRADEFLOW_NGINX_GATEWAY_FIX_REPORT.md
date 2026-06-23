# TradeFlow Nginx Gateway Fix Report

## Files changed

- `TradeFlow-Helm/tradeflow-eks/values-eks-demo.yaml`
- `TradeFlow-Helm/helm/tradeflow/values.yaml`
- `TradeFlow-Helm/helm/tradeflow/templates/nginx-gateway-configmap.yaml`
- `TradeFlow-Helm/helm/tradeflow/templates/nginx-gateway-deployment.yaml`
- `TradeFlow-Helm/helm/tradeflow/templates/nginx-gateway-service.yaml`
- `TRADEFLOW_NGINX_GATEWAY_FIX_REPORT.md`

No `M3Music-Terraform`, `M3Music-Helm`, or M3Music application files were modified.

## Exact changes made

### EKS values overlay

In `TradeFlow-Helm/tradeflow-eks/values-eks-demo.yaml`:

- Changed frontend service exposure from `LoadBalancer` to `ClusterIP`.
- Added `nginxGateway.enabled: true`.
- Configured the nginx gateway image as `nginx:stable-alpine`.
- Configured the nginx gateway service as `LoadBalancer` on port `80`.
- Left user, stock, trading, portfolio, and MongoDB services as `ClusterIP`.
- Left Gateway API, HTTPRoute routing, Rollout, and HAProxy disabled.
- Confirmed MongoDB persistence still uses `storageClass: "gp3"`.

### Parent chart defaults

In `TradeFlow-Helm/helm/tradeflow/values.yaml`:

- Added a default `nginxGateway` block with `enabled: false`.
- This keeps the new gateway opt-in for overlays and avoids changing default chart behavior.

### Nginx gateway templates

Added parent chart templates:

- `nginx-gateway-configmap.yaml`
- `nginx-gateway-deployment.yaml`
- `nginx-gateway-service.yaml`

The rendered gateway names are release-name based. With Argo CD release name `tradeflow-eks`, they render as:

- ConfigMap: `tradeflow-eks-gateway-nginx-config`
- Deployment: `tradeflow-eks-gateway`
- Service: `tradeflow-eks-gateway`

## Routing behavior

The gateway exposes one external LoadBalancer and proxies internally:

```text
/              -> http://tradeflow-eks-frontend:80
/api/users/    -> http://tradeflow-eks-user:4001/api/
/api/stocks/   -> http://tradeflow-eks-stock:4002/api/
/api/trades/   -> http://tradeflow-eks-trading:4003/api/
/api/portfolio/ -> http://tradeflow-eks-portfolio:4004/api/
```

This preserves the old prefix rewrite behavior. Example:

```text
Browser: /api/users/users/register
Nginx:   /api/users/register
Backend: tradeflow-eks-user:4001
```

## Validation commands run

Attempted Helm render:

```powershell
cd TradeFlow-Helm/helm/tradeflow
helm template tradeflow-eks . -n tradeflow -f ../../tradeflow-eks/values-eks-demo.yaml
```

Result:

```text
helm: command not found
```

Escalated PATH check:

```powershell
helm version --short
```

Result:

```text
helm: command not found
```

Static validation run:

```powershell
Select-String -Path TradeFlow-Helm\tradeflow-eks\values-eks-demo.yaml -Pattern 'storageClass'
```

Result:

```text
TradeFlow-Helm\tradeflow-eks\values-eks-demo.yaml:221:    storageClass: "gp3"
```

Checked the EKS overlay for service exposure:

```powershell
rg -n 'type: LoadBalancer|type: ClusterIP|nginxGateway|routing:|rollout:|haproxy:' TradeFlow-Helm\tradeflow-eks\values-eks-demo.yaml
```

Confirmed:

- `nginxGateway.service.type` is `LoadBalancer`.
- Frontend service is `ClusterIP`.
- User service is `ClusterIP`.
- Stock service is `ClusterIP`.
- Trading service is `ClusterIP`.
- Portfolio service is `ClusterIP`.
- MongoDB service is `ClusterIP`.
- Gateway API, HTTPRoute routing, Rollout, and HAProxy remain disabled in the EKS overlay.

Checked new gateway templates:

```powershell
rg -n 'kind: ConfigMap|kind: Deployment|kind: Service|proxy_pass' TradeFlow-Helm\helm\tradeflow\templates
```

Confirmed the new templates include:

- nginx gateway ConfigMap
- nginx gateway Deployment
- nginx gateway Service
- proxy rules for frontend, user, stock, trading, and portfolio services

Checked scoped files for forbidden references:

```powershell
rg -n 'dev-m3|m3music|M3|nfs|Gateway|HTTPRoute|Rollout' TradeFlow-Helm\tradeflow-eks\values-eks-demo.yaml TradeFlow-Helm\helm\tradeflow\templates
```

Confirmed no `dev-m3`, M3Music, NFS, Gateway API, HTTPRoute, or Rollout references exist in the EKS overlay or the new parent gateway templates.

## Validation limits

- Helm template rendering could not be completed because `helm` is not installed or not available on PATH in this environment.
- No `kubectl apply`, `helm install`, or cluster-changing command was run.
- This remains GitOps driven through the existing Argo CD application.

## Commands to run next

Commit and push the TradeFlow repo:

```bash
cd TradeFlow-Helm
git add tradeflow-eks/values-eks-demo.yaml helm/tradeflow/values.yaml helm/tradeflow/templates/nginx-gateway-configmap.yaml helm/tradeflow/templates/nginx-gateway-deployment.yaml helm/tradeflow/templates/nginx-gateway-service.yaml TRADEFLOW_NGINX_GATEWAY_FIX_REPORT.md
git commit -m "Expose TradeFlow through nginx gateway"
git push origin master
```

Force Argo CD refresh:

```bash
kubectl annotate application tradeflow-eks-demo \
  -n argocd \
  argocd.argoproj.io/refresh=hard \
  --overwrite
```

Get the new gateway LoadBalancer URL:

```bash
kubectl get svc tradeflow-eks-gateway -n tradeflow
```

Wait for the external hostname:

```bash
kubectl get svc tradeflow-eks-gateway -n tradeflow -w
```

Verify frontend access:

```bash
GATEWAY_URL="http://<tradeflow-eks-gateway-loadbalancer-hostname>"
curl -I "$GATEWAY_URL/"
```

Verify API routing:

```bash
curl -i "$GATEWAY_URL/api/users/health"
curl -i "$GATEWAY_URL/api/stocks/health"
curl -i "$GATEWAY_URL/api/trades/health"
curl -i "$GATEWAY_URL/api/portfolio/health"
```

Verify Kubernetes objects:

```bash
kubectl get svc -n tradeflow
kubectl get pods -n tradeflow
kubectl describe svc tradeflow-eks-gateway -n tradeflow
```
