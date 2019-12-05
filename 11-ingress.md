# Ingress

## Deploy Traefik
```
kubectl apply -f yamls/traefik-clusterrole.yaml
kubectl apply -f yamls/traefik-clusterrolebinding.yaml
kubectl apply -f yamls/traefik-deployment.yaml
```
## Attach External-IP to LoadBalancer
```
kubectl patch svc traefik-ingress-service -n kube-system \
  -p '{"spec": {"type": "LoadBalancer", "externalIPs":["192.168.33.111"]}}'
```
```
kubectl patch svc traefik-ingress-service -n kube-system \
  -p '{"spec": {"type": "LoadBalancer", "externalIPs":["10.0.2.15"]}}'
```

## Expose services
```
kubectl apply -f yamls/traefik-ingress-webui.yaml
```

```
kubectl apply -f yamls/traefik-ingress-prometheus.yaml
kubectl apply -f yamls/traefik-ingress-alertmanager.yaml
kubectl apply -f yamls/traefik-ingress-grafana.yaml
```

```
kubectl apply -f yamls/svc-nodeport-prometheus.yaml
kubectl apply -f yamls/svc-nodeport-alertmanager.yaml
kubectl apply -f yamls/svc-nodeport-grafana.yaml
```
