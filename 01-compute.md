# 01-Compute

## Compute Resources
In my environment, these following servers are deployed on my VirtualBox envirronment via Vagrant.
My laptop has Core i5-5300U (4 vCPU), 16 GB RAM, and 250 GB local storage.
I was concerned about shortage of compute resources before bootstrapping, but there has been no issues relative with 
compute resources so far.

- master1 - 192.168.33.101
- master2 - 192.168.33.102
- master3 - 192.168.33.103
- worker1 - 192.168.33.111
- worker2 - 192.168.33.112
- lb - 192.168.33.100

## Clean up
This will be used to clean up all confiuration files when we want to retry.

```
rm config.toml *.conf *.yaml *.service *.pem *.json *.kubeconfig *.csr
```

## Confirm the connectivity to all nodes
There are assumptions of this document to connect these VMs without password via SSH keys.
Please make sure that the VMs can be connected by SSH keys.

```
for instance in 1 2 3; do
  ssh master${instance} "hostname"
done
```
```
for instance in 1 2; do
  ssh worker${instance} "hostname"
done
ssh lb "hostname"
```
### Edit hosts

```
for instance in 1 2 3; do
  ssh master${instance} "\
  sudo sh -c \"echo 192.168.33.111 worker1 >> /etc/hosts\"
  sudo sh -c \"echo 192.168.33.112 worker2 >> /etc/hosts\"
  "
done
```
```
for instance in 1 2 3; do
  ssh master${instance} "\
    cat /etc/hosts
  "
done
```

