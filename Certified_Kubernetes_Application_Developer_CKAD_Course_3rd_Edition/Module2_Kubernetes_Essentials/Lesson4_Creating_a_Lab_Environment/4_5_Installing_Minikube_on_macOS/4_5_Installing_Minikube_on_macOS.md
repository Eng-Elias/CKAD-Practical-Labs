# 4.5 Installing Minikube on macOS

## Prerequisites
- macOS 10.11 (El Capitan) or later
- Minimum 2 vCPUs
- 4GB RAM (8GB recommended)
- 20GB free disk space
- Administrator access

## Installation Methods

### Method 1: Using Homebrew (Recommended)

#### 1. Install Homebrew (if not already installed)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 2. Install Minikube and kubectl
```bash
brew install minikube
```

#### 3. Install a Hypervisor
Choose one of the following:

**Option A: HyperKit (Recommended)**
```bash
brew install hyperkit
```

**Option B: VirtualBox**
```bash
brew install --cask virtualbox
```

### Method 2: Manual Installation

#### 1. Install kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chown root: /usr/local/bin/kubectl
```

#### 2. Install Minikube
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

## Starting Minikube

### Using HyperKit (Recommended)
```bash
minikube start --driver=hyperkit
```

### Using VirtualBox
```bash
minikube start --driver=virtualbox
```

## Verifying the Installation
```bash
kubectl version --client
minikube version
kubectl get nodes
```

## Post-Installation Setup

### Enable Shell Completion
Add to `~/.zshrc` or `~/.bash_profile`:
```bash
# Minikube
export PATH="$PATH:$HOME/.minikube/bin"

# kubectl autocompletion
source <(kubectl completion zsh)  # for zsh
# source <(kubectl completion bash)  # for bash

# Aliases
alias k=kubectl
complete -F __start_kubectl k
```

### Useful Commands
- Start Minikube: `minikube start`
- Stop Minikube: `minikube stop`
- Delete cluster: `minikube delete`
- Open dashboard: `minikube dashboard`
- SSH into node: `minikube ssh`

## Troubleshooting

### Common Issues
1. **Minikube fails to start**
   - Check virtualization: `sysctl -a | grep -E --color 'machdep.cpu.features|VMX'`
   - Try different driver: `minikube start --driver=docker`
   
2. **Network issues**
   - Disable VPN if using one
   - Check proxy settings

3. **Resource constraints**
   ```bash
   # Allocate more resources
   minikube config set memory 4096
   minikube config set cpus 2
   minikube delete
   minikube start
   ```

## Next Steps
- Verify your installation by deploying a test application
- Explore Minikube add-ons: `minikube addons list`
- Proceed with CKAD course exercises
