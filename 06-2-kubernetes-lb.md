# Deploying the Load Balancer

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

