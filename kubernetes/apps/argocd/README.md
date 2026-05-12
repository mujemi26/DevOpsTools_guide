## ⚡ Step 7: Install ArgoCD

> **What is ArgoCD?** ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes. It automates the deployment of applications to your cluster by keeping its state in sync with configuration stored in a Git repository.

### Now we install the GitOps engine.

```bash
# 1. Add the repository
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# 2. Install ArgoCD
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace
```

## 🔐 Step 8: Configure ArgoCD and Add Repository

### Now we add your repository to ArgoCD.

```bash
# Get the admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# Upgrade ArgoCD with Ingress enabled
helm upgrade argocd argo/argo-cd \
  --namespace argocd \
  --set configs.params."server\.insecure"=true \
  --set server.ingress.enabled=true \
  --set server.ingress.ingressClassName="nginx"
```

### ⚙️ Configure Ingress Host

Edit the default ingress to set your desired URL:

```bash
kubectl edit ingress argocd-server -n argocd
```

Example configuration:
```yaml
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.ex280.example.local # Change this to your preferred domain
    http:
      paths:
      - backend:
          service:
            name: argocd-server
            port:
              number: 80
        path: /
        pathType: Prefix
```

## 🔧 Argo CD CLI Installation

1. Navigate to the [ArgoCD Releases](https://github.com/argoproj/argo-cd/releases/latest) page.
2. Download the `argocd-linux-amd64` binary.
3. Install it:

```bash
wget https://github.com/argoproj/argo-cd/releases/download/<latest_version>/argocd-linux-amd64
sudo mv argocd-linux-amd64 /usr/local/bin/argocd
sudo chmod +x /usr/local/bin/argocd
argocd version --client
```

## 🚀 Login and Add Repository

```bash
# Login to ArgoCD (use the domain set in ingress or localhost with port-forward)
argocd login localhost 

# Add your repository
argocd repo add "https://github.com/mujemi26/local-gitops.git" \
  --username "mujemi26" \
  --password "[PASSWORD]" \
  --insecure-skip-server-verification \
  --grpc-web
```
