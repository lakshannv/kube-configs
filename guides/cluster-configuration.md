# **Kuberneters Cluster Configuration**
### Login as root user
```
sudo su -
```

### Update packages and their version
```
sudo apt update
sudo apt upgrade
sudo apt-get update && sudo apt-get upgrade -y
```

### Disable Firewall
```
ufw disable
```

### Disable swap
```
swapoff -a
sed -i '/swap/d' /etc/fstab
```

### Update sysctl settings for Kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
# Docker Setup
### Install docker engine
```
apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y docker-ce=5:19.03.12~3-0~ubuntu-focal containerd.io
```

# Kubernetes Setup
### Add Apt repository
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
```

### Install Kubernetes components
```
apt update && apt install -y kubeadm=1.20.7-00 kubelet=1.20.7-00 kubectl=1.20.7-00
```

### apt-mark hold is used so that these components will not be updated/removed automatically
```
sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```

### Enable and start kubelet
```
systemctl enable kubelet && systemctl start kubelet
```

### Reboot VM
```
reboot
```
---

# On Master Node
### Initialize Kubernetes Cluster
```
kubeadm init --apiserver-advertise-address=[Private IP of Master] --pod-network-cidr=192.168.0.0/16 --ignore-preflight-errors=all
```

### To start using your cluster, you need to run the following as a regular user:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Alternatively, if you are the root user, you can run:
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```
  
### Deploy Calico network
```
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```

### Get Cluster join command
```
kubeadm token create --print-join-command
```
---

# On Worker Nodes
### Use the output from kubeadm token create command in previous step from the master server and run here.
```
kubeadm join [IP]:[Port] --token [token] --discovery-token-ca-cert-hash [hash]
```