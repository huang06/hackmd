---
tags: Kubernetes
---

# Kubernetes v1.19.10 Installation

|Resource|Version|
|---|---|
|Host OS|Ubuntu MATE Desktop 20.04.2 LTS|
|Kubernetes|v1.19.10|
|deployment tool|kubeadm|
|CRI|containerd 1.4.4|
|cgroup driver|systemd|

[TOC]

## Install packages for development purpose

```bash
apt update && apt upgrade -y
apt install vim htop net-tools build-essential openssh-server axel tmux
```

## containerd

```bash
apt-get remove docker docker-engine docker.io containerd runc

apt-get update

apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

apt-key fingerprint 0EBFCD88

add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable"

apt-get update
```

```bash
CONTAINERD_VER="1.4.4-1"

apt-get install -y containerd.io=${CONTAINERD_VER}
```

```bash
apt-mark hold containerd.io
```

### Configure containerd

<https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd>

```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

### Configure cgroup driver

```bash
vi /etc/containerd/config.toml
```

```bash
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true  # add this
```

### Restart and reaload on boot

```bash
sudo systemctl daemon-reload
sudo systemctl restart containerd
```

## Install Kubernetes with kubeadm

### Disable Swap

```bash
swapoff -a
```

To permanently disable Swap, edit /etc/fstab.

### Bridged traffic and iptables

```bash
modprobe br_netfilter
```

```bash
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### Install Kubernetes packages

```bash
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF | tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update

K_VER="1.19.10-00"
apt-get install -y kubelet=${K_VER} kubectl=${K_VER} kubeadm=${K_VER}

apt-mark hold kubelet kubeadm kubectl
```

### Configure kubelet

```bash
cat > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf << EOF
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
# ExecStart=/usr/bin/kubelet    
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"
ExecStart=/usr/bin/kubelet
EOF
```

### Initialize the control-plane node

#### Pull images from k8s.gcr.io

```bash
K_VER="v1.19.10"
```

```bash
$ kubeadm config images pull \
--image-repository="k8s.gcr.io" \
--kubernetes-version=${K_VER}

W0429 15:48:45.321686   10570 configset.go:348] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[config/images] Pulled k8s.gcr.io/kube-apiserver:v1.19.10
[config/images] Pulled k8s.gcr.io/kube-controller-manager:v1.19.10
[config/images] Pulled k8s.gcr.io/kube-scheduler:v1.19.10
[config/images] Pulled k8s.gcr.io/kube-proxy:v1.19.10
[config/images] Pulled k8s.gcr.io/pause:3.2
[config/images] Pulled k8s.gcr.io/etcd:3.4.13-0
[config/images] Pulled k8s.gcr.io/coredns:1.7.0
```

```bash
kubeadm init \
--image-repository=k8s.gcr.io \
--kubernetes-version=${K_VER} \
--pod-network-cidr=10.244.0.0/16 \
--service-cidr=10.96.0.0/12 \
--control-plane-endpoint="$(hostname)" \
--apiserver-advertise-address=0.0.0.0 \
--cri-socket="/run/containerd/containerd.sock"
```

```bash
echo -e "\nalias k=kubectl" >> ${HOME}/.bashrc
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ${HOME}/.bashrc
source ${HOME}/.bashrc
```

### Install CNI Plugin

Install **Flannel v0.13.0** for Pod network.

```bash
wget "https://raw.githubusercontent.com/flannel-io/flannel/v0.13.0/Documentation/kube-flannel.yml"
```

```bash
kubectl apply -f ./kube-flannel.yml
```

```bash
$ kubectl get po -n kube-system

NAME                              READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-v8dgp           1/1     Running   0          29m
coredns-f9fd979d6-wt88m           1/1     Running   0          29m
etcd-tom-k8s                      1/1     Running   0          29m
kube-apiserver-tom-k8s            1/1     Running   0          29m
kube-controller-manager-tom-k8s   1/1     Running   0          29m
kube-flannel-ds-5jqww             1/1     Running   0          26m
kube-proxy-kdxtr                  1/1     Running   0          29m
kube-scheduler-tom-k8s            1/1     Running   0          29m
```

### Untaint the control-plan Node

```bash
$ kubectl taint nodes --all node-role.kubernetes.io/master-

node/tom-k8s untainted
```

### Deploy helloworld sample application

```bash
cat > helloworld.yaml << EOF
apiVersion: v1
kind: Namespace
metadata:
  name: helloworld
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: helloworld
  namespace: helloworld
spec:
  rules:
  - http:
      paths:
      - pathType: Prefix
        path: /helloworld
        backend:
          service:
            name: helloworld
            port:
              number: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld
  namespace: helloworld
spec:
  selector:
    matchLabels:
      run:  helloworld
  replicas: 1
  template:
    metadata:
      labels:
         run:  helloworld
    spec:
      containers:
        - name: helloworld
          image: gcr.io/google-samples/node-hello:1.0
          ports:
            - containerPort: 8080
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: helloworld
  namespace: helloworld
spec:
  ports:
  - nodePort: 31215
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    run: helloworld
  type: NodePort
EOF
```

Access the service.

```bash
kubectl apply -f ./helloworld.yaml

$ curl 0.0.0.0:31215

Hello Kubernetes!
```

Delete app.

```bash
kubectl delete -f ./helloworld.yaml
```

## Remove K8s Cluster completely

```bash
kubeadm reset -f
```

Kubernetes

```bash
rm -rf ${HOME}/.kube

sudo -i
rm -rf /etc/cni /etc/kubernetes /var/lib/dockershim /var/lib/etcd /var/lib/kubelet /var/run/kubernetes
rm -rf ${HOME}/.kube
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
rm -f /etc/cni/net.d/*
```

```bash
reboot
```
