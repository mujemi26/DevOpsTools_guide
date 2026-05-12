## 🌐 Step 6: Install Ingress-NGINX

> **What is Ingress-NGINX?** Ingress-NGINX is an Ingress controller for Kubernetes using NGINX as a reverse proxy and load balancer. It manages external access to the services in a cluster, typically handling HTTP/HTTPS traffic routing.

### Next, we deploy the Ingress controller. We use hostNetwork: true to bind to the Kind node ports and ClusterFirstWithHostNet to ensure the controller can still see internal cluster services.

```bash
# 1. Add the repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

```yaml
# 2. Create the values file (nginx-values.yaml)
controller:
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
  service:
    type: ClusterIP
  nodeSelector:
    ingress-ready: "true"
  tolerations:
    - key: "node-role.kubernetes.io/control-plane"
      operator: "Equal"
      effect: "NoSchedule"
    - key: "node-role.kubernetes.io/master"
      operator: "Equal"
      effect: "NoSchedule"
```

```bash
# 3. Install the controller
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  -f nginx-values.yaml
```
