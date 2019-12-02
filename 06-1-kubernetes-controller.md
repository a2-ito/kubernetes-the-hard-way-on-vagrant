# Bootstrapping Kubernetes Controller

### Download and Install the Kubernetes Controller Binaries

```
for instance in master1 master2 master3
do
  scp ca-key.pem ca.pem kubernetes-key.pem kubernetes.pem service-account-key.pem service-account.pem ${instance}:~
  ssh ${instance} "\
  sudo mkdir -p /var/lib/kubernetes
  sudo mkdir -p /etc/kubernetes/config
  sudo mv ca-key.pem ca.pem kubernetes-key.pem kubernetes.pem service-account.pem service-account-key.pem /var/lib/kubernetes/
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

### Configure the Kubernetes API Server

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

### Verification
```
for instance in 1 2 3
do
  ssh master${instance} "\
  sudo systemctl status kube-apiserver -l
  "
done
```
### Configure the Kubernetes Controller Manager

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
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \
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

### Verification

```
for instance in 1 2 3
do
  ssh master${instance} "\
  sudo systemctl status kube-controller-manager
  "
done
```

### Configure the Kubernetes Scheduler

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
### RBAC for Kubelet Authorization

```
ssh master1

cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

### Bind the ```system:kube-apiserver-to-kubelet``` ClusterRole to the ```kubernetes``` user:

``` 
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
``` 
``` 
exit
``` 

### Deploying the Load Balancer

```
ssh lb "\
  sudo apt-get update
  sudo apt-get install -y nginx
"
```

```
ssh lb
```

```
sudo sh -c "echo include /etc/nginx/tcpconf.d/*\; >> /etc/nginx/nginx.conf"
```

```
sudo mkdir -p /etc/nginx/tcpconf.d
sudo su - 
cat > /etc/nginx/tcpconf.d/lb <<"EOF"
stream {
    upstream kubernetes_api_servers {
        # Our web server, listening for SSL traffic
        # Note the web server will expect traffic
        # at this xip.io "domain", just for our
        # example here
        server 192.168.33.101:6443;
        server 192.168.33.102:6443;
        server 192.168.33.103:6443;
    }

    server {
        listen 6443;
        proxy_pass kubernetes_api_servers;
    }
}
EOF
```
```
sudo systemctl restart nginx
```

### Verification
```
kubectl get node --kubeconfig admin.kubeconfig
```

