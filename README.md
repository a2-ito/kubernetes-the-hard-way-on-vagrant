# kubernetes-the-hard-way-on-vagrant
This document is a version for vagrant environment of the Kubernetes the Hard Way. It follows Kelsey Hightower's tutorial https://github.com/kelseyhightower/kubernetes-the-hard-way, and attempts to make improvements and explanations where needed. So here we go.

## Prerequisites

- Install CFSSL
- Install kubectl

## Software Version

| Server        | Type                    | Version            | 
|:------------- | :---------------------- |:------------------ | 
| Master,Worker | OS                      | Ubuntu 18.04.3 LTS |
| Master        | etcd                    | v3.4.0             |
| Master        | kube-apiserver          | v1.15.0            |
| Master        | kube-controller-manager | v1.15.3            |
| Master        | kube-scheduler          | v1.15.3            |
| Master,Worker | kubectl                 | v1.15.3            |
| Worker        | kubelet                 | v1.15.3            |
| Worker        | kube-proxy              | v1.15.3            |
| Worker        | containerd              | 1.2.9              |
| Worker        | runc                    | v1.0.0-rc8         |
| Worker        | crictl                  | v1.15.0            |
| Worker        | cni-plugins             | v0.8.2             |
| LB            | nginx (lb)              | nginx/1.14.0       |
