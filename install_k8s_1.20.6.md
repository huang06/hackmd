---
tags: Kubernetes
---

# Kubernetes v1.20.6 Installation

|Resource|Version|
|---|---|
|Host OS|Ubuntu MATE Desktop 20.04.2 LTS|
|Kubernetes|v1.20.6|
|Deployment Tool|kubeadm|
|CRI|containerd 1.4.4|
|cgroup driver|systemd|
|CNI|flannel 0.13.0|

[TOC]

## Install Essential Packages

```bash
apt-get update && apt-get upgrade -y
apt-get install -y vim htop net-tools build-essential openssh-server axel tmux
```

## Install containerd

<https://docs.docker.com/engine/install/ubuntu/>

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

```vim
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true  # add this
```

### Restart containerd

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

### Letting iptables see bridged traffic

<https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/>

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

### Install Kubernetes Packages

```bash
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF | tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update

K_VER="1.20.6-00"
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
K_VER="v1.20.6"
```

```bash
$ kubeadm config images pull \
--image-repository="k8s.gcr.io" \
--kubernetes-version=${K_VER}

[config/images] Pulled k8s.gcr.io/kube-apiserver:v1.20.6
[config/images] Pulled k8s.gcr.io/kube-controller-manager:v1.20.6
[config/images] Pulled k8s.gcr.io/kube-scheduler:v1.20.6
[config/images] Pulled k8s.gcr.io/kube-proxy:v1.20.6
[config/images] Pulled k8s.gcr.io/pause:3.2
[config/images] Pulled k8s.gcr.io/etcd:3.4.13-0
[config/images] Pulled k8s.gcr.io/coredns:1.7.0
```

#### Initialize K8s with config file

```bash
cat > kubeadm-config.yaml << EOF
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: ${K_VER}
imageRepository: k8s.gcr.io
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
controlPlaneEndpoint: $(hostname)
apiServer:
  extraArgs:
    advertise-address: 0.0.0.0
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
EOF
```

```bash
$ kubeadm init \
--cri-socket=/run/containerd/containerd.sock \
--config=kubeadm-config.yaml
```

```vim
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join vm-dev:6443 --token xzwdql.gx1gimeib84n7mpl \
    --discovery-token-ca-cert-hash sha256:e8acacb6d8368c61fe685abd10495d475a6ab54c65d06b9f8b37b9e62bd2d428 \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join vm-dev:6443 --token xzwdql.gx1gimeib84n7mpl \
    --discovery-token-ca-cert-hash sha256:e8acacb6d8368c61fe685abd10495d475a6ab54c65d06b9f8b37b9e62bd2d428 
```

```bash
echo -e "\nalias k=kubectl" >> /root/.bashrc
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ${HOME}/.bashrc
source /root/.bashrc
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

NAME                             READY   STATUS    RESTARTS   AGE
coredns-74ff55c5b-csp4v          1/1     Running   0          78s
coredns-74ff55c5b-frnzh          1/1     Running   0          78s
etcd-vm-dev                      1/1     Running   0          94s
kube-apiserver-vm-dev            1/1     Running   0          94s
kube-controller-manager-vm-dev   1/1     Running   0          94s
kube-flannel-ds-9nm8m            1/1     Running   0          37s
kube-proxy-lj5lw                 1/1     Running   0          79s
kube-scheduler-vm-dev            1/1     Running   0          94s
```

### Untaint the control-plan Node

```bash
$ kubectl taint nodes --all node-role.kubernetes.io/master-

node/vm-dev untainted
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

TBD