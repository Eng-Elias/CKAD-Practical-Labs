# 4.3 Installing Minikube on Ubuntu

## Prerequisites
- Ubuntu virtual machine (or physical machine)
- Minimum 2 vCPUs
- At least 4GB RAM recommended
- 20GB+ free disk space
- sudo privileges

## Installation Steps

### 1. Update System Packages
```bash
sudo apt update
sudo apt upgrade -y
```

### 2. Install Required Tools
```bash
sudo apt install -y git vim curl wget
```

### 3. Install Docker
```bash
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

### 4. Install kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### 5. Install Minikube
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### 6. Start Minikube
```bash
minikube start --driver=docker
```

### 7. Verify Installation
```bash
kubectl version --client
minikube version
kubectl get nodes
```

## Post-Installation

### Enable kubectl Autocompletion
```bash
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc
```

### Common Minikube Commands
- Start Minikube: `minikube start`
- Stop Minikube: `minikube stop`
- Delete Minikube VM: `minikube delete`
- Access Kubernetes Dashboard: `minikube dashboard`
- SSH into Minikube VM: `minikube ssh`

## Troubleshooting

### If Minikube Fails to Start
1. Check Docker status: `systemctl status docker`
2. Ensure virtualization is enabled in BIOS
3. Verify sufficient system resources
4. Check logs: `minikube logs`

### Resetting Minikube
```bash
minikube delete
minikube start
```

## Next Steps
- Verify your installation by deploying a test application
- Explore Minikube add-ons: `minikube addons list`
- Proceed with the CKAD course exercises
