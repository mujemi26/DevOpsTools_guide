# ☸️ Kubernetes Installation Guide

## 📋 Introduction

This comprehensive guide walks you through the complete installation and setup of a Kubernetes cluster using kubeadm. You'll learn how to set up a multi-node Kubernetes cluster with proper networking using Calico.

---

## 🔧 Prerequisites

Before starting, ensure you have:
- 🐧 **Linux environment** (Ubuntu/Debian recommended)
- 🔧 **Multiple nodes** (master + worker nodes)
- 📦 **sudo privileges** on all nodes
- 🌐 **Network connectivity** between nodes
- 💾 **Minimum 2GB RAM** per node

---

## 🚀 Step-by-Step Installation

### Step 1: 🔄 Turn off Swap Area

Disabling swap is required by kubelet to ensure proper memory management ([source](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/?utm_source=chatgpt.com))

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

### Step 2: 🏷️ Set up Hostname in both Devices

```bash
sudo hostnamectl set-hostname
```

**Configure hostnames for your nodes:**

- **Master Node**: Run `sudo hostnamectl set-hostname "master-node"`
- **Worker Node**: Run `sudo hostnamectl set-hostname "worker-node1"`
- **Refresh session**: Run `exec bash` to apply the new hostname

### Step 3: 📝 Update the /etc/hosts File for Hostname Resolution

Setting up hostnames is not enough. We have to map hostnames to their IP addresses as well. You should update the `/etc/hosts` file of all nodes (or at least of the master node), as shown below. (Remember that you have to use the IP addresses of your nodes. I have only given holder values.) You can open the host's file for editing with the command `sudo nano /etc/hosts`

### Step 4: 🌐 Configure the IPV4 Bridge on All Nodes

Execute the following commands on each node:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots:

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

### Step 5: 📦 Install kubelet, kubeadm, and kubectl on Each Node

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl
```

**Create directory for Kubernetes package verification:**
```bash
sudo mkdir /etc/apt/keyrings
```

**Fetch the public key for the Kubernetes repository:**
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

**Add Kubernetes repository:**
```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

**Update package index:**
```bash
sudo apt-get update
```

**Install Kubernetes components:**
```bash
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Step 6: 🐳 Install Docker

```bash
sudo apt install docker.io
```

**Configure containerd for Kubernetes compatibility:**

1. **Create configuration directory:**
```bash
sudo mkdir /etc/containerd
```

2. **Generate default configuration:**
```bash
sudo sh -c "containerd config default > /etc/containerd/config.toml"
```

3. **Enable SystemdCgroup:**
```bash
sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml
```

4. **Restart services:**
```bash
sudo systemctl restart containerd.service
sudo systemctl restart kubelet.service
```

5. **Enable kubelet service:**
```bash
sudo systemctl enable kubelet.service
```

### Step 7: 🎯 Initialize the Kubernetes Cluster on the Master Node

**Pull required container images:**
```bash
sudo kubeadm config images pull
```

**Initialize the master node:**
```bash
sudo kubeadm init --pod-network-cidr=10.10.0.0/16
```

**Configure kubectl for cluster management:**
```bash
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Step 8: 🌐 Configure kubectl and Calico

In Kubernetes, the default for networking traffic to/from pods is default-allow. If you do not lock down network connectivity using network policy, then all pods can communicate freely with other pods.

Calico consists of **networking** to secure network communication, and advanced **network policy** to secure cloud-native microservices/applications at scale.

**Deploy Calico operator:**
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.5/manifests/tigera-operator.yaml
```

**Download Calico custom resources:**
```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.31.5/manifests/custom-resources.yaml -O
```

**Modify CIDR to match your pod network:**
```bash
sed -i 's/cidr: 192\.168\.0\.0\/16/cidr: 10.10.0.0\/16/g' custom-resources.yaml
```

**Apply Calico configuration:**
```bash
kubectl create -f custom-resources.yaml
```

---

## 🌐 Alternative: Installing Cilium CNI

Cilium is a powerful CNI that provides advanced networking, security, and observability features. Here are multiple installation methods:

### 🔧 **Method 1: Install Cilium using Helm (Recommended)**

**Add the Cilium Helm repository:**
```bash
helm repo add cilium https://helm.cilium.io/
helm repo update
```

**Install Cilium with default configuration:**
```bash
helm install cilium cilium/cilium \
  --namespace kube-system \
  --set kubeProxyReplacement=strict \
  --set k8sServiceHost=API_SERVER_IP \
  --set k8sServicePort=API_SERVER_PORT
```

**For kind clusters, use this configuration:**
```bash
helm install cilium cilium/cilium \
  --namespace kube-system \
  --set kubeProxyReplacement=strict \
  --set k8sServiceHost=kind-control-plane \
  --set k8sServicePort=6443 \
  --set tunnel=vxlan \
  --set ipam.mode=kubernetes
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

### 🔧 **Cilium Configuration Options**

**Enable Hubble (observability):**
```bash
helm install cilium cilium/cilium \
  --namespace kube-system \
  --set hubble.enabled=true \
  --set hubble.relay.enabled=true \
  --set hubble.ui.enabled=true
```

**Enable eBPF host routing:**
```bash
helm install cilium cilium/cilium \
  --namespace kube-system \
  --set kubeProxyReplacement=strict \
  --set enableCnpMode=true
```

**Enable transparent encryption:**
```bash
helm install cilium cilium/cilium \
  --namespace kube-system \
  --set encryption.enabled=true \
  --set encryption.type=wireguard
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

### 📊 **Cilium Features**

Cilium provides advanced features including:
- 🔒 **Network Policies** - L3/L4/L7 security policies
- 📊 **Observability** - Hubble for network visibility
- 🚀 **Performance** - eBPF-based networking
- 🛡️ **Security** - Identity-based security
- 🔄 **Load Balancing** - Advanced load balancing
- 🌐 **Transparent Encryption** - WireGuard integration

---

### Step 9: ➕ Add Worker Nodes to the Cluster

Once you have configured the master node, you can add worker nodes to the cluster. When initializing Kubeadm on the master node, you will receive a token that you can use to add worker nodes.

**Add worker nodes using kubeadm join:**
```bash
sudo kubeadm join <MASTER_NODE_IP>:<API_SERVER_PORT> --token <TOKEN> --discovery-token-ca-cert-hash <CERTIFICATE_HASH>
```

### Step 10: ✅ Verify the Cluster and Test

**Check cluster nodes:**
```bash
kubectl get nodes
```

**Verify all pods are running:**
```bash
kubectl get pods --all-namespaces
```

---

## 💡 Tips and Best Practices

### 🔒 **Security Best Practices**
- 🔐 **Use strong passwords** for all user accounts
- 🛡️ **Enable RBAC** for fine-grained access control
- 🔑 **Rotate certificates** regularly
- 🚪 **Configure network policies** to restrict pod-to-pod communication

### 📊 **Performance Optimization**
- 💾 **Monitor resource usage** regularly
- ⚡ **Use resource limits** and requests for pods
- 🔄 **Implement horizontal pod autoscaling**
- 📈 **Set up monitoring** with Prometheus and Grafana

### 🛠️ **Maintenance Tips**
- 🔄 **Keep Kubernetes updated** to the latest stable version
- 📋 **Regular backups** of etcd data
- 🧹 **Clean up unused resources** periodically
- 📝 **Document your cluster configuration**

### 🌐 **Networking Best Practices**
- 🎯 **Use specific CIDR ranges** for different environments
- 🔒 **Implement network policies** for security
- 📡 **Configure proper DNS** resolution
- 🌍 **Set up ingress controllers** for external access

### 📚 **Troubleshooting**
- 🔍 **Check logs** using `kubectl logs`
- 📊 **Monitor events** with `kubectl get events`
- 🐛 **Use kubectl describe** for detailed resource information
- 🔧 **Verify network connectivity** between nodes

### 🚀 **Production Readiness**
- 📦 **Use persistent volumes** for data storage
- 🔄 **Implement rolling updates** for zero-downtime deployments
- 🛡️ **Set up proper backup and disaster recovery**
- 📊 **Monitor cluster health** with comprehensive logging

---

## 🎯 **Next Steps**

After successful installation, consider:
- 📊 **Setting up monitoring** (Prometheus, Grafana)
- 🔄 **Configuring CI/CD pipelines**
- 🛡️ **Implementing security policies**
- 📚 **Learning Kubernetes concepts** and best practices

---

**Happy Kubernetes-ing! ☸️**