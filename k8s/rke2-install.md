# Install rke2

## 1. Prepare the System
```sh
# disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^(.*)$/#\1/g' /etc/fstab

# load kernal modules and config sysctl

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/modules-load.d/rke2.conf
overlay
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/rke2.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system
```

## 2. download resources and install.sh

download resources from [rke2](https://github.com/rancher/rke2)  
list:
  * sha256sum-amd64.txt
  * rke2.linux-amd64.tar.gz
  * rke2-images-calico.linux-amd64.tar.zst
  * rke2-images.linux-amd64.tar.zst
```sh
# create images dir
mkdir -p /var/lib/rancher/rke2/agent/images
mkdir ~/rke2 and ~/rke2
# mv above resources to ~/rke2
curl -sfL https://get.rke2.io --output install.sh

```

## 3. install and start rke2.service

```sh
# set config.yaml
sudo tee /etc/rancher/rke2/config.yaml << 'EOF'
# single node
node-taint:
  - "CriticalAddonsOnly=true:NoExecute"
cni: calico
node-ip: "truely ip"
tls-san:
  - "truely ip"
EOF
sudo INSTALL_RKE2_ARTIFACT_PATH=/home/liky/rke2 sh install.sh
sudo systemctl enable rke2-server.service
systemctl start rke2-server.service

# check logs
journalctl -u rke2-server -f
```

## 4. copy config and set alias
```sh
sudo cp /etc/rancher/rke2/rke2.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config


sudo tee /etc/crictl.yaml << 'EOF'
runtime-endpoint: unix:///run/k3s/containerd/containerd.sock
image-endpoint: unix:///run/k3s/containerd/containerd.sock
timeout: 10
debug: false
EOF

alias crictl='sudo /var/lib/rancher/rke2/bin/crictl --config /etc/crictl.yaml'
alias ctr='sudo /var/lib/rancher/rke2/bin/ctr --address /run/k3s/containerd/containerd.sock -n k8s.io'
```

# Uninstall rke2

```sh
sudo rke2-uninstall.sh
```