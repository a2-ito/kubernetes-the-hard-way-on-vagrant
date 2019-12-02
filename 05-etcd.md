# Bootstrapping etcd

## Download and Install etcd binaries

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
  --listen-client-urls https://INTERNAL_IP:2379,https://127.0.0.1:2379 \
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

### Verification

```
for instance in 1 2 3
do
  ssh master${instance} "\
  echo ========= master${instance} =========
  sudo ETCDCTL_API=3 etcdctl member list \\
    --endpoints=https://127.0.0.1:2379 \\
    --cacert=/etc/etcd/ca.pem \\
    --cert=/etc/etcd/kubernetes.pem \\
    --key=/etc/etcd/kubernetes-key.pem
  "
done
```
