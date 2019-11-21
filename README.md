# kubernetes-the-hard-way-on-vagrant
This document is a version for vagrant environment of the Kubernetes the Hard Way. It follows Kelsey Hightower's tutorial https://github.com/kelseyhightower/kubernetes-the-hard-way, and attempts to make improvements and explanations where needed. So here we go.

## Confirm the connectivity
<details>
<summary>Click to expand!</summary>

```
for instance in 1 2 3; do
  ssh master${instance} "hostname"
done
```
```
for instance in 1 2; do
  ssh worker${instance} "hostname"
done
```
</details>

## Certificate Authority
```
export KUBERNETES_PUBLIC_IP_ADDRESS='192.168.33.100'
```

<details>

```
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```
</details>

### The Kubernetes API Server Certificate
<details>

```
cat > kubernetes-csr.json <<EOF
{
  "CN": "*.example.com",
  "hosts": [
    "10.32.0.1",
    "master1",
    "master2",
    "master3",
    "192.168.33.101",
    "192.168.33.102",
    "192.168.33.103",
    "kubernetes",
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.svc.cluster.local",
    "kubernetes.example.com",
    "10.14.20.127",
    "localhost",
    "127.0.0.1"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes
```

The following thigs will be generated:
```
kubernetes-key.pem
kubernetes.pem
kubernetes-csr.json
```
</details>

### The Kubelet Client Certificates
<details>

```
for instance in 1 2; do
cat > worker${instance}-csr.json <<EOF
{
  "CN": "system:node:worker${instance}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
INTERNAL_IP=192.168.33.11${instance}
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=worker${instance},${INTERNAL_IP} \
  -profile=kubernetes \
  worker${instance}-csr.json | cfssljson -bare worker${instance}
done
```
</details>

### The Kube Proxy Client Certificate
<details>

```
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy
```
</details>

### The Controller Manager Client Certificate 
<details>

```
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```
</details>

### The Kubernetes Scheduler Client Certificate 
<details>

```
cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler
```
</details>

### The Admin Client Certificate
<details>

```
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```
</details>

### The Service Account Key Pair
<details>

```
cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
```
</details>

### Copy Certificates
```
for instance in master1 master2 master3
do
  scp ca.pem kubernetes-key.pem kubernetes.pem service-account.pem ${instance}:~
done
```
```
for instance in worker1 worker2
do
  scp ${instance}.pem ${instance}-key.pem ${instance}:~
done
```

## The Encryption Config File
<details>

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```
</details>

```
for instance in master1 master2 master3
do
  scp encryption-config.yaml ${instance}:~
done
```

## Kubernetes Configuration File

### The kube-controller-manager Kubernetes Configuration File 
<details>

```
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://127.0.0.1:6443 \
  --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-credentials system:kube-controller-manager \
  --client-certificate=kube-controller-manager.pem \
  --client-key=kube-controller-manager-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-context default \
  --cluster=kubernetes-the-hard-way \
  --user=system:kube-controller-manager \
  --kubeconfig=kube-controller-manager.kubeconfig

kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
```
</details>

### The kube-scheduler Kubernetes Configuration File
<details>

```
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://127.0.0.1:6443 \
  --kubeconfig=kube-scheduler.kubeconfig

kubectl config set-credentials system:kube-scheduler \
  --client-certificate=kube-scheduler.pem \
  --client-key=kube-scheduler-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-scheduler.kubeconfig

kubectl config set-context default \
  --cluster=kubernetes-the-hard-way \
  --user=system:kube-scheduler \
  --kubeconfig=kube-scheduler.kubeconfig

kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
```
</details>

### The kubelet Kubernetes Configuration File
<details>

```
for instance in worker1 worker2; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://192.168.33.101:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```
</details>

### The kube-proxy Kubernetes Configuration File
<details>

```
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://192.168.33.101:6443 \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config set-credentials system:kube-proxy \
  --client-certificate=kube-proxy.pem \
  --client-key=kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config set-context default \
  --cluster=kubernetes-the-hard-way \
  --user=system:kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```
</details>

### Copy Configuration File
```
for instance in master1 master2 master3
do
  scp kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${instance}:~
done
```
```
for instance in worker1 worker2
do
  scp ${instance}.kubeconfig kube-proxy.kubeconfig ${instance}:~
done
```

## etcd
<details>

```
for instance in master1 master2 master3
do
  ssh ${instance} "\
  sudo mkdir -p /etc/etcd/
  ls /etc/etcd/
  sudo mv ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
  curl -L https://github.com/coreos/etcd/releases/download/v3.4.0/etcd-v3.4.0-linux-amd64.tar.gz \
    -o etcd-v3.4.0-linux-amd64.tar.gz
  tar xzvf etcd-v3.4.0-linux-amd64.tar.gz 
  sudo cp etcd-v3.4.0-linux-amd64/etcd* /usr/bin/
  sudo mkdir -p /var/lib/etcd
  "
done
```

### Configure the etcd Server
```
for instance in 1 2 3
do
cat > etcd.service <<"EOF"
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/bin/etcd --name ETCD_NAME \
  --cert-file=/etc/etcd/kubernetes.pem \
  --key-file=/etc/etcd/kubernetes-key.pem \
  --peer-cert-file=/etc/etcd/kubernetes.pem \
  --peer-key-file=/etc/etcd/kubernetes-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ca.pem \
  --initial-advertise-peer-urls https://INTERNAL_IP:2380 \
  --listen-peer-urls https://INTERNAL_IP:2380 \
  --listen-client-urls https://INTERNAL_IP:2379,http://127.0.0.1:2379 \
  --advertise-client-urls https://INTERNAL_IP:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster master1=https://192.168.33.101:2380,master2=https://192.168.33.102:2380,master3=https://192.168.33.103:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
  export INTERNAL_IP=192.168.33.10${instance}
  export ETCD_NAME=master${instance}
  sed -i s/INTERNAL_IP/${INTERNAL_IP}/g etcd.service
  sed -i s/ETCD_NAME/${ETCD_NAME}/g etcd.service
  scp etcd.service master${instance}:~
  rm etcd.service
  ssh master${instance} "sudo mv etcd.service /etc/systemd/system/"
done
```
</details>

<details>

```
for instance in 1 2 3
do
  ssh master${instance} "\
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
  "
done
```
</details>

### Verification
```
for instance in 1 2 3
do
  ssh master${instance} "\
  etcdctl --ca-file=/etc/etcd/ca.pem cluster-health
  "
done
```

## Kubernetes Controller
### Download and Install the Kubernetes Controller Binaries
<details>

```
for instance in master1 master2 master3
do
  scp ca.pem kubernetes-key.pem kubernetes.pem service-account.pem ${instance}:~
  ssh ${instance} "\
  sudo mkdir -p /var/lib/kubernetes
  sudo mkdir -p /etc/kubernetes/config
  sudo mv ca.pem kubernetes-key.pem kubernetes.pem service-account.pem /var/lib/kubernetes/
  sudo mv encryption-config.yaml /var/lib/kubernetes/
  wget -nv https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-apiserver
  wget -nv https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-controller-manager
  wget -nv https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-scheduler
  wget -nv https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
  sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
  "
done
```
</details>

### Configure the Kubernetes API Server
<details>

```
for instance in 1 2 3
do
cat > kube-apiserver.service <<"EOF"
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \
  --advertise-address=INTERNAL_IP \
  --allow-privileged=true \
  --apiserver-count=3 \
  --audit-log-maxage=30 \
  --audit-log-maxbackup=3 \
  --audit-log-maxsize=100 \
  --audit-log-path=/var/log/audit.log \
  --authorization-mode=Node,RBAC \
  --bind-address=0.0.0.0 \
  --client-ca-file=/var/lib/kubernetes/ca.pem \
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \
  --etcd-cafile=/var/lib/kubernetes/ca.pem \
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \
  --etcd-servers=https://192.168.33.101:2379,https://192.168.33.102:2379,https://192.168.33.103:2379 \
  --event-ttl=1h \
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \
  --kubelet-https=true \
  --runtime-config=api/all \
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --service-node-port-range=30000-32767 \
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
  export INTERNAL_IP=192.168.33.10${instance}
  sed -i s/INTERNAL_IP/${INTERNAL_IP}/g kube-apiserver.service
  scp kube-apiserver.service master${instance}:~
  rm kube-apiserver.service
  ssh master${instance} "sudo mv kube-apiserver.service /etc/systemd/system/"
done
```
```
for instance in 1 2 3
do
  ssh master${instance} "\
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver
  sudo systemctl start kube-apiserver
  "
done
```
</details>

```
for instance in 1 2 3
do
  ssh master${instance} "\
  sudo systemctl status kube-apiserver -l
  "
done
```
### Configure the Kubernetes Controller Manager
<details>

```
for instance in 1 2 3
do
cat > kube-controller-manager.service <<"EOF"
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
  --address=0.0.0.0 \
  --cluster-cidr=10.200.0.0/16 \
  --cluster-name=kubernetes \
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \
  --root-ca-file=/var/lib/kubernetes/ca.pem \
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --use-service-account-credentials=true \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
  export INTERNAL_IP=192.168.33.10${instance}
  sed -i s/INTERNAL_IP/${INTERNAL_IP}/g kube-controller-manager.service
  scp kube-controller-manager.service master${instance}:~
  rm kube-controller-manager.service
  ssh master${instance} "\
  sudo mv kube-controller-manager.service /etc/systemd/system/
  sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
  "
done
```

```
for instance in 1 2 3
do
  ssh master${instance} "\
  sudo systemctl daemon-reload
  sudo systemctl enable kube-controller-manager
  sudo systemctl start kube-controller-manager
  "
done
```
</details>

```
for instance in 1 2 3
do
  ssh master${instance} "\
  sudo systemctl status kube-controller-manager
  "
done
```

### Configure the Kubernetes Scheduler
<details>

```
cat > kube-scheduler.yaml <<"EOF"
apiVersion: kubescheduler.config.k8s.io/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
cat > kube-scheduler.service <<"EOF"
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
  --leader-elect=true \
  --config=/etc/kubernetes/config/kube-scheduler.yaml \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
for instance in 1 2 3
do
  scp kube-scheduler.yaml kube-scheduler.service master${instance}:~
  ssh master${instance} "\
  sudo mv kube-scheduler.service /etc/systemd/system/
  sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/
  sudo mv kube-scheduler.yaml /etc/kubernetes/config/
  "
done
rm kube-scheduler.yaml kube-scheduler.service
```
```
for instance in 1 2 3
do
  ssh master${instance} "\
  sudo systemctl daemon-reload
  sudo systemctl enable kube-scheduler
  sudo systemctl start kube-scheduler
  "
done
```
</details>

```
for instance in 1 2 3
do
  ssh master${instance} "\
  sudo systemctl status kube-scheduler
  "
done
```
### Health check
```
for instance in 1 2 3
do
  ssh master${instance} "\
  kubectl get componentstatuses
  "
done
```
## Bootstrapping Kubernetes Workers
<details>

```
for instance in 1 2
do
  ssh worker${instance} "\
  sudo yum install -y socat conntrack ipset
  sudo swapoff -a
  "
done
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
</details>

### Configure CNI Networking
<details>

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
</details>

### Configure containerd
<details>

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
</details>

### Configure the Kubelet
```
for instance in 1 2
do
  scp ca.pem worker${instance}-key.pem worker${instance}.pem worker${instance}.kubeconfig worker${instance}:~
  ssh worker${instance} "\
  sudo mv worker${instance}-key.pem worker${instance}.pem /var/lib/kubelet/
  sudo mv worker${instance}.kubeconfig /var/lib/kubelet/kubeconfig
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
 | sudo tee /etc/systemd/system/kubelet.service
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

### Create temporary kubeconfig file:
```
cat > kubeconfig <<"EOF"
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /var/lib/kubernetes/ca.pem
    server: https://192.168.33.101:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubelet
  name: kubelet
current-context: kubelet
users:
- name: kubelet
  user:
    token: chAng3m3
EOF
for instance in 1 2
do
  scp kubeconfig worker${instance}:~
  ssh worker${instance} "\
  sudo mv kubeconfig /var/lib/kubelet/
  "
done
rm kubeconfig
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
```
for instance in 1 2
do
  ssh worker${instance} "\
  sudo systemctl status kubelet
  "
done
```

## Configure the Kube-proxy
<details>
<summary>Click to expand!</summary>

```
cat > kube-proxy-config.yaml << "EOF"
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kubelet/kubeconfig"
mode: "iptables"
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
  sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfing
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
</details>

### Verification
<details>

```
for instance in 1 2 3
do
  ssh master${instance} "kubectl get node --kubeconfig=admin.kubeconfig"
done
```
</details>

## The Admin Kubernetes configuration File
```
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://10.14.20.127:6443 \
  --kubeconfig=admin.kubeconfig

kubectl config set-credentials admin \
  --client-certificate=admin.pem \
  --client-key=admin-key.pem \
  --kubeconfig=admin.kubeconfig

kubectl config set-context kubernetes-the-hard-way \
  --cluster=kubernetes-the-hard-way \
  --user=admin \
  --kubeconfig=admin.kubeconfig

kubectl config use-context kubernetes-the-hard-way \
  --kubeconfig=admin.kubeconfig
```

```
KUBECONFIG=admin.kubeconfig kubectl get node
```


