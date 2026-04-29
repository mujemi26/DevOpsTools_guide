## 🐳 Step 2: Install Docker, Kubectl, and Kind

### 📦 Install Docker by running the following commands:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker ${USER}
newgrp docker
docker version
```

### ⚙️ Install kubectl by running the following command

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo mv kubectl /usr/local/bin
sudo chmod +x /usr/local/bin/kubectl
kubectl version --client
```
### 🔗 Install kind

- Go to https://github.com/kubernetes-sigs/kind/releases

- Scroll down till you find the downloadable files.

- Right click on the Linux AMD64 and copy the link.

#### In the terminal, run the following:

```bash
wget https://github.com/kubernetes-sigs/kind/releases/download/<latest_version>/kind-linux-amd64
sudo mv kind-linux-amd64 /usr/local/bin/kind
sudo chmod +x /usr/local/bin/kind
kind version
```
