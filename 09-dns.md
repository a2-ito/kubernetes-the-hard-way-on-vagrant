# Deploying the DNS Cluster Add-on

```
export KUBECONFIG=admin.kubeconfig
```

### The DNS Cluster Add-on
Deploy the ```coredns``` cluster add-on:

```
kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml \
  --kubeconfig admin.kubeconfig
```

```
kubectl get pods --kubeconfig admin.kubeconfig \
  -l k8s-app=kube-dns -n kube-system
```
```
kubectl run --kubeconfig admin.kubeconfig \
  --generator=run-pod/v1 busybox --image=busybox:1.28.4 --command -- sleep 3600
```

```
kubectl exec --kubeconfig admin.kubeconfig -ti busybox -- nslookup kubernetes
```


