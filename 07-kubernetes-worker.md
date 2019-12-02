# Bootstrapping Kubernetes Workers

### Download and Install Worker Binaries

```
for instance in 1 2
do
  ssh worker${instance} "\
  sudo apt-get install -y socat conntrack ipset
  sudo swapoff -a
  "
done
```

### Option: Disable swap permanently
```
sudo vi /etc/fstab
```
```
for instance in 1 2
do
  ssh worker${instance} "\
  wget -nv https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.15.0/crictl-v1.15.0-linux-amd64.tar.gz 
  wget -nv https://github.com/opencontainers/runc/releases/download/v1.0.0-rc8/runc.amd64 
  wget -nv https://github.com/containernetworking/plugins/releases/download/v0.8.2/cni-plugins-linux-amd64-v0.8.2.tgz
  wget -nv https://github.com/containerd/containerd/releases/download/v1.2.9/containerd-1.2.9.linux-amd64.tar.gz
  wget -nv https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl
  wget -nv https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-proxy
  wget -nv https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubelet
  sudo mkdir -p /etc/cni/net.d
  sudo mkdir -p /opt/cni/bin
  sudo mkdir -p /var/lib/kubelet
  sudo mkdir -p /var/lib/kube-proxy
  sudo mkdir -p /var/lib/kubernetes
  sudo mkdir -p /var/run/kubernetes
  mkdir containerd
  tar -xvf crictl-v1.15.0-linux-amd64.tar.gz
  tar -xvf containerd-1.2.9.linux-amd64.tar.gz -C containerd
  sudo tar -xvf cni-plugins-linux-amd64-v0.8.2.tgz -C /opt/cni/bin/
  sudo mv runc.amd64 runc
  chmod +x crictl kubectl kube-proxy kubelet runc 
  sudo mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
  sudo mv containerd/bin/* /bin/
  "
done
```

### Configure CNI Networking

```
cat > 99-loopback.conf <<EOF
{
    "cniVersion": "0.3.1",
    "name": "lo",
    "type": "loopback"
}
EOF
for instance in 1 2
do
POD_CIDR=10.200.${instance}.0/24
cat > 10-bridge.conf <<EOF
{
  "cniVersion": "0.3.1",
  "name": "bridge",
  "type": "bridge",
  "bridge": "cnio0",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "ranges": [
        [{"subnet": "${POD_CIDR}"}]
    ],
    "routes": [{"dst": "0.0.0.0/0"}]
  }
}
EOF
  scp 10-bridge.conf 99-loopback.conf worker${instance}:~
  ssh worker${instance} "\
  sudo mv 10-bridge.conf /etc/cni/net.d/
  sudo mv 99-loopback.conf /etc/cni/net.d/
  "
done
rm 10-bridge.conf 99-loopback.conf
```

### Configure containerd

```
cat > config.toml <<"EOF"
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
EOF
for instance in 1 2
do
  scp config.toml worker${instance}:~
  ssh worker${instance} "\
  sudo mkdir -p /etc/containerd/
  sudo mv config.toml /etc/containerd/
  "
done
rm config.toml
```

```
cat > containerd.service <<"EOF"
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
for instance in 1 2
do
  scp containerd.service worker${instance}:~
  ssh worker${instance} "\
  sudo mv containerd.service /etc/systemd/system/
  "
done
rm containerd.service
```
```
for instance in 1 2
do
  ssh worker${instance} "\
  sudo systemctl daemon-reload
  sudo systemctl enable containerd
  sudo systemctl start containerd
  "
done
```

### Configure the Kubelet

```
for instance in 1 2
do
  scp ca.pem worker${instance}-key.pem worker${instance}.pem worker${instance}.kubeconfig worker${instance}:~
  ssh worker${instance} "\
  sudo mv worker${instance}-key.pem worker${instance}.pem /var/lib/kubelet/
  sudo mv ca.pem /var/lib/kubernetes/
  "
done
```

```
for instance in 1 2
do
POD_CIDR=10.200.${instance}.0/24
cat > kubelet-config.yaml <<EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/worker${instance}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/worker${instance}-key.pem"
EOF
  scp kubelet-config.yaml worker${instance}:~
  ssh worker${instance} "\
  sudo mv kubelet-config.yaml /var/lib/kubelet/
  "
done
rm kubelet-config.yaml
```

```
for instance in 1 2
do
cat > kubelet.service <<"EOF"
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \
  --config=/var/lib/kubelet/kubelet-config.yaml \
  --container-runtime=remote \
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \
  --image-pull-progress-deadline=2m \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --network-plugin=cni \
  --register-node=true \
  --node-ip=INTERNAL_IP \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
  export INTERNAL_IP=192.168.33.11${instance}
  sed -i s/INTERNAL_IP/${INTERNAL_IP}/g kubelet.service
  scp kubelet.service worker${instance}:~
  ssh worker${instance} "\
  sudo mv kubelet.service /etc/systemd/system/
  sudo mv worker${instance}.kubeconfig /var/lib/kubelet/kubeconfig
  "
done
rm kubelet.service
```

```
for instance in 1 2
do
  ssh worker${instance} "\
  sudo systemctl daemon-reload
  sudo systemctl enable kubelet
  sudo systemctl start kubelet
  "
done
```

### Verification

```
for instance in 1 2
do
  ssh worker${instance} "\
  sudo systemctl status kubelet
  "
done
```

### Configure the Kube-proxy

```
cat > kube-proxy-config.yaml << "EOF"
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
iptables:
  masqueradeAll: true
clusterCIDR: "10.200.0.0/16"
EOF
for instance in 1 2
do
  scp kube-proxy-config.yaml worker${instance}:~
  ssh worker${instance} "\
  sudo mv kube-proxy-config.yaml /var/lib/kube-proxy/
  "
done
rm kube-proxy-config.yaml
```

```
cat > kube-proxy.service <<"EOF"
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
for instance in 1 2
do
  scp kube-proxy.service worker${instance}:~
  ssh worker${instance} "\
  sudo mv kube-proxy.service /etc/systemd/system/
  sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
  "
done
rm kube-proxy.service
```
```
for instance in 1 2
do
  ssh worker${instance} "\
  sudo systemctl daemon-reload
  sudo systemctl enable kube-proxy
  sudo systemctl start kube-proxy
  "
done
```
```
for instance in 1 2
do
  ssh worker${instance} "\
  sudo systemctl status kube-proxy
  "
done
```

### Verification

```
for instance in 1 2 3
do
  ssh master${instance} "kubectl get node --kubeconfig=admin.kubeconfig"
done
```

