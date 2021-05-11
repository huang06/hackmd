---
tags: kubeadm, Kubernetes, Kubeflow
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

## Install Helm 3

```bash
HELM_VER="v3.5.3"

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/${HELM_VER}/scripts/get-helm-3 \
&& chmod 700 get_helm.sh \
&& ./get_helm.sh
```

## Install Dynamic NFS Provisioner

### Set up NFS Server on host

```bash
apt-get install -y nfs-common nfs-kernel-server
```

```bash
EXTERNAL_NFS_IP="127.0.0.1"
EXTERNAL_NFS_DIR="/svc/nfsdata"

mkdir -p ${EXTERNAL_NFS_DIR}

# echo "${EXTERNAL_NFS_DIR} *(rw,no_root_squash,no_subtree_check)" | tee -a /etc/fstab > /dev/null
echo "${EXTERNAL_NFS_DIR} *(rw,no_root_squash,no_subtree_check)" | tee -a /etc/exports > /dev/null
```

```bash
systemctl restart nfs-kernel-server
```

Verify that NFS server is accessible.

```bash
$ showmount -e ${EXTERNAL_NFS_IP}

Export list for 127.0.0.1:
/var/nfsdata *
```

### Install nfs-subdir-external-provisioner

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```

```bash
CHART_VERSION="4.0.4"

helm install nfs-subdir-external-provisioner \
nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--version ${CHART_VERSION} \
--set image.repository=k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner \
--set image.tag=v4.0.0 \
--set nfs.server=${EXTERNAL_NFS_IP} \
--set nfs.path=${EXTERNAL_NFS_DIR} \
--set storageClass.name=nfs \
--set storageClass.defaultClass=true
```

#### Verification

```bash
$ kubectl logs -l app=nfs-subdir-external-provisioner

I0429 17:03:08.859100       1 leaderelection.go:242] attempting to acquire leader lease  default/cluster.local-nfs-subdir-external-provisioner...
I0429 17:03:08.864928       1 leaderelection.go:252] successfully acquired lease default/cluster.local-nfs-subdir-external-provisioner
I0429 17:03:08.865024       1 event.go:278] Event(v1.ObjectReference{Kind:"Endpoints", Namespace:"default", Name:"cluster.local-nfs-subdir-external-provisioner", UID:"86a13d77-ac94-463b-bbae-5e7116371cc1", APIVersion:"v1", ResourceVersion:"9910", FieldPath:""}): type: 'Normal' reason: 'LeaderElection' nfs-subdir-external-provisioner-6bd9449677-8fhwz_c6852cf4-7cbe-475f-a425-e7e734339f72 became leader
I0429 17:03:08.865075       1 controller.go:820] Starting provisioner controller cluster.local/nfs-subdir-external-provisioner_nfs-subdir-external-provisioner-6bd9449677-8fhwz_c6852cf4-7cbe-475f-a425-e7e734339f72!
I0429 17:03:08.965280       1 controller.go:869] Started provisioner controller cluster.local/nfs-subdir-external-provisioner_nfs-subdir-external-provisioner-6bd9449677-8fhwz_c6852cf4-7cbe-475f-a425-e7e734339f72!
```

## Remove K8s Cluster completely

```bash
kubeadm reset -f
```

Harbor

```bash
docker-compose down -v
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

Kubeflow PVC

```bash
cd /svc/nfsdata

rm -rf kubeflow-minio-pv-claim-pvc* \
kubeflow-mysql-pv-claim-pvc* \
kubeflow-katib-mysql-pvc* \
kubeflow-metadata-mysql-pvc*

rm /usr/local/bin/kfctl
```

```bash
reboot
```
