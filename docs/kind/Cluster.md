## 🚀 Step 3: Create the Kind Cluster

> **What are we doing here?** This step initializes your local Kubernetes cluster using Kind. We configure the cluster to support custom networking (by disabling the default CNI) and map host ports to allow Ingress traffic to reach our services.


### This configuration disables the default CNI and maps ports 80/443 to your host.
```yaml
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
- role: worker
```

### 🚀 Create a cluster by running the following command:
```bash
kind create cluster --config=kind-config.yaml
kubectl cluster-info --context kind-kind
```
