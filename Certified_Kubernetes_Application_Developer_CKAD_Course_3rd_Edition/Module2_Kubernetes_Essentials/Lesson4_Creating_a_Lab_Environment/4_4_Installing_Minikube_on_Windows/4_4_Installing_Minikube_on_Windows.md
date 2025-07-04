# 4.4 Installing Minikube on Windows

## Prerequisites
- Windows 10/11 64-bit (Pro, Enterprise, or Education)
- Virtualization enabled in BIOS
- Administrator access
- Minimum 4GB RAM (8GB recommended)
- 20GB free disk space

## Installation Steps

### 1. Download Minikube Installer
1. Visit: https://minikube.sigs.k8s.io/docs/start/
2. Download the Windows installer (.exe)
3. Run the installer with administrator privileges

### 2. Download kubectl
1. Download kubectl from: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
2. Add the kubectl binary to your system PATH

### 3. Start Minikube
1. Open PowerShell / Command Prompt
2. Run:
   ```powershell
   minikube start
   ```

## Common Issues and Solutions

### 1. Hyper-V Not Available
- Ensure your Windows edition supports Hyper-V
- Enable virtualization in BIOS/UEFI settings
- Run `systeminfo` in CMD to check Hyper-V requirements

### 2. Minikube Fails to Start
- Try different drivers: `minikube start --driver=docker`
- Increase memory: `minikube config set memory 4096`
- Check logs: `minikube logs`

### 3. Network Issues
- Disable any VPN or proxy temporarily
- Configure Docker/Windows to use system proxy if needed

## Useful Commands
- Start Minikube: `minikube start`
- Open Dashboard: `minikube dashboard`
- SSH into node: `minikube ssh`
- Stop Minikube: `minikube stop`
- Delete cluster: `minikube delete`

## Next Steps
- Verify your installation by running a test deployment
- Explore Minikube add-ons: `minikube addons list`
- Proceed with CKAD course exercises
