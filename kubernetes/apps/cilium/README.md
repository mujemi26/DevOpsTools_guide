## 🕸️ Step 5: Install Cilium (The CNI)

> **What is Cilium?** Cilium is an advanced networking, observability, and security solution for container workloads. It uses eBPF technology to provide high-performance networking and fine-grained network policies.

### We must install the network engine first so the nodes can reach a Ready state. We will use the Docker DNS bypass to ensure stability across reboots.

### 🔧 **Method 1: Install Cilium using Helm**

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

### 🔧 **Cilium Configuration Options**

**Enable Hubble (observability):**

```bash
helm install cilium cilium/cilium \
  --namespace kube-system \
  --set hubble.enabled=true \
  --set hubble.relay.enabled=true \
  --set hubble.ui.enabled=true
```

### 📦 **Method 2: Install Cilium using Cilium CLI**

**Install Cilium CLI:**

```bash
curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-amd64.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin
rm cilium-linux-amd64.tar.gz cilium-linux-amd64.tar.gz.sha256sum
```

**Install Cilium in the cluster:**

```bash
cilium install
```

**For kind clusters:**

```bash
cilium install --helm-set kubeProxyReplacement=strict \
  --helm-set k8sServiceHost=kind-control-plane \
  --helm-set k8sServicePort=6443 \
  --helm-set tunnel=vxlan \
  --helm-set ipam.mode=kubernetes
```

### 🚀 **Method 3: Install Cilium using kubectl**

**Download the Cilium manifest:**

```bash
curl -LO https://github.com/cilium/cilium/releases/latest/download/cilium.yaml
```

**Apply the manifest:**

```bash
kubectl apply -f cilium.yaml
```

### ✅ **Verify Cilium Installation**

**Check Cilium status:**

```bash
cilium status
```

**Verify all Cilium pods are running:**

```bash
kubectl get pods -n kube-system -l k8s-app=cilium
```

**Check Cilium connectivity:**

```bash
cilium connectivity test
```

### 🛠️ **Cilium CLI Commands**

**Check Cilium version:**

```bash
cilium version
```

**Upgrade Cilium:**

```bash
cilium upgrade
```

**Uninstall Cilium:**

```bash
cilium uninstall
```

**Check Cilium connectivity:**

```bash
cilium connectivity test
```

**View Cilium metrics:**

```bash
cilium metrics list
```

### Install Cilium as Pure CNI only with specfic version

```bash
cilium install --list-versions

cilium install --version 1.17.5 \
  --helm-set ipam.operator.clusterPoolIPv4PodCIDRList='{172.16.0.0/16}' \ # flag to move the pods away from your node IPs range
  --helm-set kubeProxyReplacement=false \ # turn off kube-proxy
  --helm-set routingMode=tunnel \ # use tunnel mode
  --helm-set tunnelProtocol=vxlan \ # use vxlan or geneve
  --helm-set bpf.masquerade=false \ # turn off masquerade
  --helm-set bpf.hostRouting=false \ # turn off host routing
  --helm-set operator.replicas=1 # number of operator replicas
```

### **Check that the node CIDRs are set:**

```bash
kubectl -n kube-system get configmap cilium-config -o yaml | grep cluster-pool-ipv4-cidr
```

## Common Trap When installing Cilium

```text
Most cloud providers (Azure, AWS, GCP) default to 10.x.x.x for their VPCs/VNets. Most CNIs (Cilium, Flannel, Calico) also default to 10.x.x.x for Pods. In a small home lab or a standard Azure setup, they eventually crash into each other. Rule of thumb: Always set your Pod CIDR to something distinct like 172.16.0.0/16 or 192.168.0.0/16 if your nodes are on 10.x
```
