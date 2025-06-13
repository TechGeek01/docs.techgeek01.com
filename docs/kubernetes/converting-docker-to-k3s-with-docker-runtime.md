# Prerequisites
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install git tmux curl openssh-server qemu-guest-agent
sudo apt autoremove
```

# Install k3s
```bash
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" sh -s - server --docker
```
