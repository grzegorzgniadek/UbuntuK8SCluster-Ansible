# UbuntuK8SCluster-Ansible

# Set up a Highly Available Kubernetes Cluster using kubeadm
Follow this documentation to set up a highly available Kubernetes cluster using Ubuntu 20.04

This documentation guides you in setting up a cluster with three master nodes, three worker nodes and a load balancer node using HAProxy.

## Virtual Machines Environment
|Role|FQDN|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Load Balancer|ubuntulb1|192.168.2.8|Ubuntu 20.04 |8G|2|
|Master|ubuntumaster1|192.168.2.2|Ubuntu 20.04|8G|2|
|Master|ubuntumaster2|192.168.2.3|Ubuntu 20.04|8G|2|
|Master|ubuntumaster3|192.168.2.4|Ubuntu 20.04|8G|2|
|Worker|ubuntuworker1|192.168.2.5|Ubuntu 20.04|8G|2|
|Worker|ubuntuworker1|192.168.2.6|Ubuntu 20.04|8G|2|
|Worker|ubuntuworker1|192.168.2.7|Ubuntu 20.04|8G|2|

## Pre-requisites Master
Use bootstrap scripts to set up iptables:
- Open port 2222 on OpenSSH
- Open port 2381 on etcd/healthz
- Open port 6443 on Kube-apiserver
- Open port 6784 on Weave-net/status
- Open port 2379-2380 on etcd server client API
- Open 10250 on Kubelet API
- Open 10251 on Kube-scheduler
- Open 10252 on Kube-controller-Manager
- Open 10257 on Kube-controller-Manger/healthz
- Open 10259 on Kube-scheduler/healthz

## Pre-requisites Worker
- Open port 2222 on OpenSSH
- Open port 10250 on Kubelet API
- Open port 30000-32767 on NodePort Services


## Set up load balancer node
##### Install Haproxy
```
apt update && apt install -y haproxy
```
##### Configure haproxy
Append the below lines to **/etc/haproxy/haproxy.cfg**
```
frontend kubernetes-frontend
    bind 192.168.2.8:6443
    mode tcp
    option tcplog
    default_backend kubernetes-backend

backend kubernetes-backend
    mode tcp
    option tcp-check
    balance roundrobin
    server ubuntumaster1 192.168.2.2:6443 check fall 3 rise 2
    server ubuntumaster2 192.168.2.3:6443 check fall 3 rise 2
    server ubuntumaster3 192.168.2.4:6443 check fall 3 rise 2
```
##### Restart haproxy service
```
systemctl restart haproxy
```

## Set up cluster
##### Run ansible-playbook on Masternode1 with ip 192.168.2.2
```
git clone https://github.com/grzegorzgniadek/UbuntuK8SCluster-Ansible.git
cd UbuntuK8SCluster-Ansible
ansible-playbook -Kk site.yaml
```

##### Verifying the cluster
```
kubectl cluster-info
kubectl get nodes
kubectl get cs
```
