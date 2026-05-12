# Add the Jetstack Helm repository

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm search repo jetstack
```

# Install cert-manager

```bash
helm install cert-manager jetstack/cert-manager \
 --namespace cert-manager \
 --create-namespace \
 --version v1.14.0 \
 --set installCRDs=true
```

## Step 5: Validate the Installation

```bash
kubectl get pods -n cert-manager
```

## Step 6: Configure the Let's Encrypt ClusterIssuer

> ### **"ClusterIssuer" to tell Cert-manager how to talk to Let's Encrypt. Since you are using NGINX Ingress, the HTTP-01 challenge is the easiest way to verify your domain. Create a file named letsencrypt-staging.yaml (always start with staging to avoid Let's Encrypt rate limits!)**

```bash
# letsencrypt-staging.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # Use your real email here
    email: user@example.com
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-staging-key
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
```

```bash
kubectl apply -f letsencrypt-staging.yaml
```
