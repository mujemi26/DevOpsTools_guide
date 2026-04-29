## 🕸️ Step 5: Install Cilium (The CNI)
### We must install the network engine first so the nodes can reach a Ready state. We will use the Docker DNS bypass to ensure stability across reboots.

```bash 
# 1. Add and update the repository
helm repo add cilium https://helm.cilium.io/
helm repo update
```

```yaml
# 2 cilium-values.yaml
k8sServiceHost: "kind-control-plane"
k8sServicePort: "6443"
kubeProxyReplacement: true
k8sClientRateLimit:
  qps: 50
  burst: 100
l2announcements:
  enabled: true
hubble:
  enabled: false
```
```bash
# 3. Install Cilium
helm install cilium cilium/cilium --version 1.17.5 \
  --namespace kube-system \
  -f cilium-values.yaml
```
