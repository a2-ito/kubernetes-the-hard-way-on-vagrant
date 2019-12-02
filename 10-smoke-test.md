# Smoke Test
## Deployment

```
kubectl create deployment nginx --image=nginx --kubeconfig admin.kubeconfig
```

### Verification from inside
```
POD_IP=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].status.podIP}")
kubectl run --image=giantswarm/tiny-tools --rm --restart=Never -i testpod -- curl -v --head http://${POD_IP}
kubectl run --image=giantswarm/tiny-tools --rm --restart=Never -i testpod -- sh -c "ip a; curl -v --head http://${POD_IP}"
```

### Verification from outside - via kubectl proxy

```
POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:80
```
```
curl --head localhost:8080
```

## Services

```
kubectl expose deployment nginx --port 80 --type NodePort
```
### Verification
```
kubectl get svc -o wide
kubectl exec -it busybox -- nslookup nginx
```
```
curl 10.200.1.4:80
curl 192.168.33.111:30845
curl 192.168.33.112:30845
curl 10.0.2.15:30845
```

