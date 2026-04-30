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
kubectl edit ingress argocd-server -n argocd
```

- Navigate to https://github.com/argoproj/argo-cd/releases/latest to get the Argo CD CLI.
- Scroll down till you find the downloadable files.
- Copy the link to the Linux amd64 file.
- Install Argo CD CLI by running the following commands:

```bash
wget https://github.com/argoproj/argo-cd/releases/download/<latest_version>/argocd-linux-amd64
sudo mv argocd-linux-amd64 /usr/local/bin/argocd
sudo chmod +x /usr/local/bin/argocd
argocd version --client
```

```bash
# Login to ArgoCD
argocd login localhost # use your domain you set in the ingress and /etc/hosts

# Add your repository
argocd repo add "https://github.com/mujemi26/local-gitops.git" \
  --username "mujemi26" \
  --password "[PASSWORD]" \
  --insecure-skip-server-verification \
  --grpc-web
```
