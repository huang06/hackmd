---
tags: Kubernetes, CRI
---

# cri-tools Notes

|Resource|Version|
|---|---|
|K8s|v1.20.6|
|Container runtime|containerd 1.4.4|
|crictl|v1.13.0|

<https://kubernetes.io/docs/tasks/debug-application-cluster/crictl/>

[TOC]

## crictl

### Configure config file

```bash
cat > crictl.yaml << EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 10
debug: true
EOF
```

### Display runtime version information

```bash
$ crictl --config=crictl.yaml version

Version:  0.1.0
RuntimeName:  containerd
RuntimeVersion:  1.4.4
RuntimeApiVersion:  v1alpha2
```

### Display information of the container runtime

```bash
$ crictl --config=crictl.yaml info -o yaml

cniconfig:
  Networks:
  - Config:
      CNIVersion: 0.3.1
      Name: cni-loopback
      Plugins:
      - Network:
          dns: {}
          ipam: {}
          type: loopback
        Source: '{"type":"loopback"}'
      Source: |-
        {
        "cniVersion": "0.3.1",
        "name": "cni-loopback",
        "plugins": [{
          "type": "loopback"
        }]
        }
    IFName: lo
  - Config:
      CNIVersion: 0.3.1
      Name: cbr0
      Plugins:
      - Network:
          dns: {}
          ipam: {}
          type: flannel
        Source: '{"delegate":{"hairpinMode":true,"isDefaultGateway":true},"type":"flannel"}'
      - Network:
          capabilities:
            portMappings: true
          dns: {}
          ipam: {}
          type: portmap
        Source: '{"capabilities":{"portMappings":true},"type":"portmap"}'
      Source: |
        {
          "name": "cbr0",
          "cniVersion": "0.3.1",
          "plugins": [
            {
              "type": "flannel",
              "delegate": {
                "hairpinMode": true,
                "isDefaultGateway": true
              }
            },
            {
              "type": "portmap",
              "capabilities": {
                "portMappings": true
              }
            }
          ]
        }
    IFName: eth0
  PluginConfDir: /etc/cni/net.d
  PluginDirs:
  - /opt/cni/bin
  PluginMaxConfNum: 1
  Prefix: eth
config:
  cni:
    binDir: /opt/cni/bin
    confDir: /etc/cni/net.d
    confTemplate: ""
    maxConfNum: 1
  containerd:
    defaultRuntime:
      ContainerAnnotations: null
      PodAnnotations: null
      baseRuntimeSpec: ""
      options: null
      privileged_without_host_devices: false
      runtimeEngine: ""
      runtimeRoot: ""
      runtimeType: ""
    defaultRuntimeName: runc
    disableSnapshotAnnotations: true
    discardUnpackedLayers: false
    noPivot: false
    runtimes:
      runc:
        ContainerAnnotations: null
        PodAnnotations: null
        baseRuntimeSpec: ""
        options: {}
        privileged_without_host_devices: false
        runtimeEngine: ""
        runtimeRoot: ""
        runtimeType: io.containerd.runc.v2
    snapshotter: overlayfs
    untrustedWorkloadRuntime:
      ContainerAnnotations: null
      PodAnnotations: null
      baseRuntimeSpec: ""
      options: null
      privileged_without_host_devices: false
      runtimeEngine: ""
      runtimeRoot: ""
      runtimeType: ""
  containerdEndpoint: /run/containerd/containerd.sock
  containerdRootDir: /var/lib/containerd
  disableApparmor: false
  disableCgroup: false
  disableHugetlbController: true
  disableProcMount: false
  disableTCPService: true
  enableSelinux: false
  enableTLSStreaming: false
  ignoreImageDefinedVolumes: false
  imageDecryption:
    keyModel: ""
  maxConcurrentDownloads: 3
  maxContainerLogSize: 16384
  registry:
    auths: null
    configs: null
    headers: null
    mirrors:
      docker.io:
        endpoint:
        - https://registry-1.docker.io
  restrictOOMScoreAdj: false
  rootDir: /var/lib/containerd/io.containerd.grpc.v1.cri
  sandboxImage: k8s.gcr.io/pause:3.2
  selinuxCategoryRange: 1024
  stateDir: /run/containerd/io.containerd.grpc.v1.cri
  statsCollectPeriod: 10
  streamIdleTimeout: 4h0m0s
  streamServerAddress: 127.0.0.1
  streamServerPort: "0"
  systemdCgroup: false
  tolerateMissingHugetlbController: true
  unsetSeccompProfile: ""
  x509KeyPairStreaming:
    tlsCertFile: ""
    tlsKeyFile: ""
golang: go1.13.15
lastCNILoadStatus: OK
status:
  conditions:
  - message: ""
    reason: ""
    status: true
    type: RuntimeReady
  - message: ""
    reason: ""
    status: true
    type: NetworkReady
```

### List images

```bash
$ crictl --config=crictl.yaml images

IMAGE                              TAG                 IMAGE ID            SIZE
gcr.io/google-samples/node-hello   1.0                 9ef4b4c241fc2       253MB
k8s.gcr.io/kube-proxy              v1.20.6             9a1ebfd8124d7       49.5MB
k8s.gcr.io/pause                   3.2                 80d28bedfe5de       300kB
quay.io/coreos/flannel             v0.13.0             e708f4bb69e31       19.4MB
```

### List containers

```bash
$ crictl --config=crictl.yaml ps -a

CONTAINER ID        IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID
9be2de438e4fc       bfe3a36ebd252       3 hours ago         Running             coredns                   1                   3f9690aa39ede
d7ed91fb8cb00       bfe3a36ebd252       3 hours ago         Running             coredns                   1                   aad707ed2dd09
aebfbda1021f0       e708f4bb69e31       3 hours ago         Running             kube-flannel              1                   d75fe55941136
feba557821f7e       e708f4bb69e31       3 hours ago         Exited              install-cni               1                   d75fe55941136
97b916211c3c3       9a1ebfd8124d7       3 hours ago         Running             kube-proxy                1                   6511d532b9eb5
d66537710c0f6       b93ab2ec44754       3 hours ago         Running             kube-scheduler            1                   38a2b6b6e4ec8
e1e38ad61cdce       560dd11d4550f       3 hours ago         Running             kube-controller-manager   1                   75610e3428918
ca0c4206f0569       b05d611c1af90       3 hours ago         Running             kube-apiserver            1                   c9f5df9841c7e
2581395efe30e       0369cf4303ffd       3 hours ago         Running             etcd                      1                   4d6187719d34b
588b261348541       bfe3a36ebd252       12 hours ago        Exited              coredns                   0                   79284abacd358
fe38444ed436d       bfe3a36ebd252       12 hours ago        Exited              coredns                   0                   de9b570498435
ea4a4a2f25587       e708f4bb69e31       12 hours ago        Exited              kube-flannel              0                   1b72c8b6b99f0
22e173ed0e477       9a1ebfd8124d7       12 hours ago        Exited              kube-proxy                0                   18adc24c15094
25fbc60cbedbd       560dd11d4550f       12 hours ago        Exited              kube-controller-manager   0                   b9baf0427109d
9e26fb627945d       b05d611c1af90       12 hours ago        Exited              kube-apiserver            0                   576cfc2fe35d8
5c7fe5d42a635       0369cf4303ffd       12 hours ago        Exited              etcd                      0                   cf26995d1b4ef
b4d3902182561       b93ab2ec44754       12 hours ago        Exited              kube-scheduler            0                   2dccd51625111
```

```bash
$ crictl --config=crictl.yaml pods

POD ID              CREATED             STATE               NAME                             NAMESPACE           ATTEMPT
3f9690aa39ede       3 hours ago         Ready               coredns-74ff55c5b-csp4v          kube-system         1
aad707ed2dd09       3 hours ago         Ready               coredns-74ff55c5b-frnzh          kube-system         1
d75fe55941136       3 hours ago         Ready               kube-flannel-ds-9nm8m            kube-system         1
6511d532b9eb5       3 hours ago         Ready               kube-proxy-lj5lw                 kube-system         1
38a2b6b6e4ec8       3 hours ago         Ready               kube-scheduler-vm-dev            kube-system         1
75610e3428918       3 hours ago         Ready               kube-controller-manager-vm-dev   kube-system         1
c9f5df9841c7e       3 hours ago         Ready               kube-apiserver-vm-dev            kube-system         1
4d6187719d34b       3 hours ago         Ready               etcd-vm-dev                      kube-system         1
```