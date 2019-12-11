# kube-prometheus

## Download kube-prometheus
```
git clone https://github.com/coreos/kube-prometheus.git
```

## Deploy Ingress Containers
```
kubectl create -f kube-prometheus/manifests/setup
kubectl create -f kube-prometheus/manifests
```

## Patch Eternal-IP
```
kubectl patch svc traefik-web-ui -n kube-system \
  -p '{"spec": {"type": "LoadBalancer", "externalIPs":["10.0.2.15"]}}'
```

## Option: Delete kube-prometheus

```
kubectl delete --ignore-not-found=true -f kube-prometheus/manifests/ -f kube-prometheus/manifests/setup
```

### Configure statefulset
```
kubectl get statefulset alertmanager-main -n monitoring -o yaml > statefulset-alertmanager-main.yaml
```
```
vi statefulset-alertmanager-main.yaml
```

## Workaround
```
# add paused:true   
kubectl edit alertmanagers.monitoring.coreos.com

# delete the old one   
kubectl delete statefulsets.apps alertmanager-main

# remove both liveness and readiness probe (for the time being)
kubectl create -f dump.yaml
```

