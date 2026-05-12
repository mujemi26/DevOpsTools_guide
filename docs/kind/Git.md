## 🐙 Step 1: Install and Configure Git

> **What is Git?** Git is a distributed version control system used to track changes in source code. In a GitOps workflow, it serves as the single source of truth for your infrastructure and application configurations.

Run:
```bash
sudo apt update
sudo apt install git -y
git --version
```

### Configure the git user information by running:
```bash
git config --global user.name "your name"
git config --global user.email "your@email.com"
git config --global core.editor "vim" # execute
```
