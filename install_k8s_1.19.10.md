---
tags: kubeadm, Kubernetes, Kubeflow
---

# Kubernetes v1.19.10 Installation

|Resource|Version|
|---|---|
|Host OS|Ubuntu Server 20.04.2 LTS|
|docker-ce|20.10.6|
|Kubernetes|v1.19.10|
|Helm|v3.5.3|

[TOC]

## Install packages for development purpose

```bash
apt update && apt upgrade -y
apt install vim htop net-tools build-essential openssh-server axel tmux
```

## Install Docker

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
DOCKER_VER="5:20.10.6~3-0~ubuntu-focal"
CONTAINERD_VER="1.4.4-1"

apt-get install -y docker-ce=${DOCKER_VER} docker-ce-cli=${DOCKER_VER} containerd.io=${CONTAINERD_VER}

docker run hello-world  # verification
```

```bash
apt-mark hold docker-ce docker-ce-cli containerd.io
```

### (Optional) Enable experimental features

To build multi-arch images on this node, add **"experimental": "enabled"** to ~/.docker/config.json

```bash
systemctl restart docker
```

## Install docker-compose

```bash
DC_VER=1.29.1

axel -n 20 -o docker-compose "https://github.com/docker/compose/releases/download/${DC_VER}/docker-compose-Linux-x86_64"
chmod 755 docker-compose
mv docker-compose /usr/bin
```

## Install Kubernetes with kubeadm

### Disable Swap

```bash
swapoff -a
```

To permanently disable Swap, edit /etc/fstab and then update grub.

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

```bash
systemctl daemon-reload
systemctl restart kubelet
```

### Initializes the control-plane node

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
--image-repository="k8s.gcr.io" \
--kubernetes-version=${K_VER} \
--pod-network-cidr="10.244.0.0/16" \
--service-cidr="10.96.0.0/12"
```

```bash
rm -r ${HOME}/.kube
mkdir -p ${HOME}/.kube
cp -i /etc/kubernetes/admin.conf ${HOME}/.kube/config
chown $(id -u):$(id -g) ${HOME}/.kube/config
```

```bash
echo -e "\nalias k=kubectl" >> ${HOME}/.bashrc
echo "export KUBECONFIG=${HOME}/.kube/config" >> ${HOME}/.bashrc
source ${HOME}/.bashrc
```

Wait for the 5 Pods being Running status. There're 2 Pods still in Pending status. It's fine.

Install **Flannel v0.13.0** for Pod network.

```bash
wget "https://raw.githubusercontent.com/flannel-io/flannel/v0.13.0/Documentation/kube-flannel.yml"
```

```bash
kubectl apply -f ./kube-flannel.yml
```

Wait for the 8 Pods being Running status.

```bash
$ kubectl get po -n kube-system

NAME                            READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-4nmqr         1/1     Running   0          4m5s
coredns-f9fd979d6-np8qk         1/1     Running   0          4m5s
etcd-tompc                      1/1     Running   0          4m23s
kube-apiserver-tompc            1/1     Running   0          4m23s
kube-controller-manager-tompc   1/1     Running   0          4m23s
kube-flannel-ds-tvh96           1/1     Running   0          99s
kube-proxy-bw4kq                1/1     Running   0          4m5s
kube-scheduler-tompc            1/1     Running   0          4m23s

```

### Untaint the control-plan Node

```bash
$ kubectl taint nodes --all node-role.kubernetes.io/master-

node/aaeon-boxer-6842m untainted
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
