# 13-docker-registry

## Create Namespace

```
kubectl create namespace docker-registry
```

## Create Deployment

```
kubectl apply -f yamls/docker-registry-deployment.yaml
```

## Create Certificate - ```registry.laptop2-cluster.local```

```
openssl genrsa -out registry.laptop2-cluster.local-key.pem 4096

openssl req -new -key registry.laptop2-cluster.local-key.pem \
  -out registry.laptop2-cluster.local.csr \
  -subj "/CN=registry.laptop2-cluster.local/O=asggroup"

openssl x509 -req -in registry.laptop2-cluster.local.csr \
  -signkey registry.laptop2-cluster.local-key.pem \
  -out registry.laptop2-cluster.local.pem -days 365 \
  -extfile SAN-laptop2-cluster.local.txt
```

```
sed "s/CRT-docker-registry/`cat registry.laptop2-cluster.local.pem|base64 -w0`/g" yamls/docker-registry-secrets.yaml | \
  sed "s/KEY-docker-registry/`cat registry.laptop2-cluster.local-key.pem|base64 -w0`/g" | \
  kubectl apply -f -
```

## Create Ingress for Docker Registry
```
kubectl apply -f yamls/docker-registry-ingress.yaml
```


