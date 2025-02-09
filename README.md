# K3s High Availability (HA) Setup

## Introduction

This project involves deploying a Kubernetes cluster using `K3s` on a set of 5 virtual machines (VMs). The VMs are
running on Proxmox hosted on an old x86 PC with 16Gb of memory. The setup is designed to provide a lightweight,
efficient and scalable Kubernetes environment for testing and development.  
The cluster consists of 3 control plane nodes and 2 worker nodes, providing a solid foundation for deploying and
managing containerized applications. The use of `K3s` ensures minimal resource overhead, making it ideal for
environments with limited hardware resources.

Project gracioulsy provided by [Wim van 
## VM overview

| VM Name       | MAC Address       | IP Address   | Memory (Gb) | Disk Size (Gb) | CPU Cores |
|---------------|-------------------|--------------|-------------|----------------|-----------|
| k3s-server-01 | BC:24:11:66:6F:07 | 172.16.12.22 | 8           | 32             | 2         |
| k3s-server-02 | BC:24:11:02:57:C8 | 172.16.12.23 | 8           | 32             | 2         |
| k3s-server-03 | BC:24:11:4F:A3:86 | 172.16.12.24 | 8           | 32             | 2         |
| k3s-worker-01 | BC:24:11:12:B1:D2 | 172.16.12.25 | 16          | 32             | 4         |
| k3s-worker-02 | BC:24:11:07:BA:1A | 172.16.12.26 | 16          | 32             | 4         |
| k3s-worker-03 | BC:24:11:07:BA:1A | 172.16.12.27 | 16          | 32             | 4         |

## Create VMs in Proxmox

[Proxmox Cloud-Init Support](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#qm_cloud_init)  
[Ubuntu Cloud Images](https://cloud-images.ubuntu.com)

Create template based on cloud-init image:

```bash
#!/bin/bash

qm create 9000 --name ubuntu-cloud-init --memory 2048 --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-pci
qm set 9000 --scsi0 data:0,import-from=/var/lib/vz/template/iso/noble-server-cloudimg-amd64.img
qm set 9000 --ide1 data:cloudinit
qm set 9000 --boot order=scsi0
qm set 9000 --serial0 socket --vga serial0
qm template 9000
```

Create VMs based on template and start them:

```bash
#!/bin/bash

qm clone 9000 218 --name k3s-lb-01
qm resize 218 scsi0 32G
qm set 218 --net0 virtio,bridge=vmbr0,tag=12
qm set 218 --cpu host
qm set 218 --memory 8192
qm set 218 --cores 2
qm set 218 --ostype l26
qm set 218 --tags k3s,prd,proxy
qm set 218 --localtime 1
qm set 218 --onboot 1
qm set 218 --ciuser jurre
qm set 218 --sshkey ~/.ssh/id_rsa.pub
qm set 218 --ciupgrade 1
qm set 218 --ipconfig0 ip=172.16.12.20/24,gw=172.16.12.1

qm clone 9000 219 --name k3s-lb-02
qm resize 219 scsi0 32G
qm set 219 --net0 virtio,bridge=vmbr0,tag=12
qm set 219 --cpu host
qm set 219 --memory 8192
qm set 219 --cores 2
qm set 219 --ostype l26
qm set 219 --tags k3s,prd,proxy
qm set 219 --localtime 1
qm set 219 --onboot 1
qm set 219 --ciuser jurre
qm set 219 --sshkey ~/.ssh/id_rsa.pub
qm set 219 --ciupgrade 1
qm set 219 --ipconfig0 ip=172.16.12.21/24,gw=172.16.12.1

qm clone 9000 220 --name k3s-server-01
qm resize 220 scsi0 32G
qm set 220 --net0 virtio,bridge=vmbr0,tag=12
qm set 220 --cpu host
qm set 220 --memory 8192
qm set 220 --cores 2
qm set 220 --ostype l26
qm set 220 --tags k3s,prd,server
qm set 220 --localtime 1
qm set 220 --onboot 1
qm set 220 --ciuser jurre
qm set 220 --sshkey ~/.ssh/id_rsa.pub
qm set 220 --ciupgrade 1
qm set 220 --ipconfig0 ip=172.16.12.22/24,gw=172.16.12.1

qm clone 9000 221 --name k3s-server-02
qm resize 221 scsi0 32G
qm set 221 --net0 virtio,bridge=vmbr0,tag=12
qm set 221 --cpu host
qm set 221 --memory 8192
qm set 221 --cores 2
qm set 221 --ostype l26
qm set 221 --tags k3s,prd,server
qm set 221 --localtime 1
qm set 221 --onboot 1
qm set 221 --ciuser jurre
qm set 221 --sshkey ~/.ssh/id_rsa.pub
qm set 221 --ciupgrade 1
qm set 221 --ipconfig0 ip=172.16.12.23/24,gw=172.16.12.1

qm clone 9000 222 --name k3s-server-03
qm resize 222 scsi0 32G
qm set 222 --net0 virtio,bridge=vmbr0,tag=12
qm set 222 --cpu host
qm set 222 --memory 8192
qm set 222 --cores 2
qm set 221 --ostype l26
qm set 222 --tags k3s,prd,server
qm set 222 --localtime 1
qm set 222 --onboot 1
qm set 222 --ciuser jurre
qm set 222 --sshkey ~/.ssh/id_rsa.pub
qm set 222 --ciupgrade 1
qm set 222 --ipconfig0 ip=172.16.12.24/24,gw=172.16.12.1

qm clone 9000 223 --name k3s-worker-01
qm resize 223 scsi0 32G
qm set 223 --net0 virtio,bridge=vmbr0,tag=12
qm set 223 --cpu host
qm set 223 --memory 16384
qm set 223 --cores 4
qm set 223 --ostype l26
qm set 223 --tags k3s,prd,worker
qm set 223 --localtime 1
qm set 223 --onboot 1
qm set 223 --ciuser jurre
qm set 223 --sshkey ~/.ssh/id_rsa.pub
qm set 223 --ciupgrade 1
qm set 223 --ipconfig0 ip=172.16.12.25/24,gw=172.16.12.1

qm clone 9000 224 --name k3s-worker-02
qm resize 224 scsi0 32G
qm set 224 --net0 virtio,bridge=vmbr0,tag=12
qm set 224 --cpu host
qm set 224 --memory 16384
qm set 224 --cores 4
qm set 224 --ostype l26
qm set 224 --tags k3s,prd,worker
qm set 224 --localtime 1
qm set 224 --onboot 1
qm set 224 --ciuser jurre
qm set 224 --sshkey ~/.ssh/id_rsa.pub
qm set 224 --ciupgrade 1
qm set 224 --ipconfig0 ip=172.16.12.26/24,gw=172.16.12.1

qm clone 9000 225 --name k3s-worker-03
qm resize 225 scsi0 32G
qm set 225 --net0 virtio,bridge=vmbr0,tag=12
qm set 225 --cpu host
qm set 225 --memory 16384
qm set 225 --cores 4
qm set 225 --ostype l26
qm set 225 --tags k3s,prd,worker
qm set 225 --localtime 1
qm set 225 --onboot 1
qm set 225 --ciuser jurre
qm set 225 --sshkey ~/.ssh/id_rsa.pub
qm set 225 --ciupgrade 1
qm set 225 --ipconfig0 ip=172.16.12.27/24,gw=172.16.12.1

qm start 218
qm start 219
qm start 220
qm start 221
qm start 222
qm start 223
qm start 224
qm start 225
```

## Prepare VMS for use in homelab environment
A few agents (qemu-guest-agent and elastic agent (for now)) need to be installed before setting up the `K3s` cluster. This is done on all VMs. 
```bash
# install qemu-guest-agent
sudo apt-get install qemu-guest-agent

# enable and start the service
sudo systemctl enable qemu-guest-agent

# install elastic agent for monitoring
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.17.1-linux-x86_64.tar.gz 
tar xzvf elastic-agent-8.17.1-linux-x86_64.tar.gz
cd elastic-agent-8.17.1-linux-x86_64
sudo ./elastic-agent install --url=https://172.16.12.16:8220 --enrollment-token=<token> --insecure

#after successful installation remove the .tar.gz file.
rm elastic-agent-8.17.1-linux-x86_64.tar.gz
```

## Install first server (control plane node)

Use the `--cluster-init` flag to create the first server in the cluster and initialize the embedded `etcd` datastore for
high availability (HA).  
To install a specific version of `K3s`, set the `INSTALL_K3S_VERSION` environment variable before running the
installation
script.  
All `K3s` versions can be found here: [k3s release](https://github.com/k3s-io/k3s/releases)

```bash
# Use a specific version of K3s.
export INSTALL_K3S_VERSION=v1.31.5+k3s1

# Installs K3s, initializes the first server in HA mode and disables Traefik and the ServiceLB load balancer.
sudo curl -sfL https://get.k3s.io | sh -s - server --cluster-init --disable="traefik" --disable="servicelb"

# Displays the kubeconfig file for accessing the K3s cluster.
sudo cat /etc/rancher/k3s/k3s.yaml

# Displays the token used for joining nodes to the K3s cluster.
sudo cat /var/lib/rancher/k3s/server/token
```

## Install additional servers (control plane nodes)

```bash
export INSTALL_K3S_VERSION=v1.31.5+k3s1
export K3S_URL=https://172.16.12.22:6443
export K3S_TOKEN=<token from first server>

sudo curl -sfL https://get.k3s.io |sh -s - server --disable="traefik" --disable="servicelb"
```

## Install a cluster load balancer (HAProxy) on the two additional servers
Make sure all server (control plane nodes) contain the following config file.
```bash
# /etc/rancher/k3s/config.yaml
tls-san: 172.16.12.19
```

### Configuring the HAProxy
Install HAProxy and KeepAlived
```bash
sudo apt-get install haproxy keepalived
```

Add the following to the /etc/haproxy/haproxy.cfg file
```bash
frontend k3s-frontend
    bind *:6443
    mode tcp
    option tcplog
    default_backend k3s-backend

backend k3s-backend
    mode tcp
    option tcp-check
    balance roundrobin
    default-server inter 10s downinter 5s
    server server-1 172.16.12.22:6443 check
    server server-2 172.16.12.23:6443 check
    server server-3 172.16.12.24:6443 check
```

Add the following to /etc/keepalived/keepalived.conf
```bash
global_defs {
  enable_script_security
  script_user root
}

vrrp_script chk_haproxy {
    script 'killall -0 haproxy' # faster than pidof
    interval 2
}

vrrp_instance haproxy-vip {
    interface eth0
    state <STATE> # MASTER on lb-1, BACKUP on lb-2
    priority <PRIORITY> # 200 on lb-1, 100 on lb-2

    virtual_router_id 51

    virtual_ipaddress {
        10.10.10.100/24
    }

    track_script {
        chk_haproxy
    }
}
```

Restart HAProxy and KeepAlived
```bash
sudo systemctl restart haproxy
sudo systemctl restart keepalived
```

## Install agents (worker nodes)

```bash
export INSTALL_K3S_VERSION=v1.31.5+k3s1
export K3S_URL=https://172.16.12.19:6443 # Virtual IP address of the load balancer
export K3S_TOKEN=<token from first server>

sudo curl -sfL https://get.k3s.io | sh -s - agent
```

## Set up kubectl to use the K3s kubeconfig file

From one of the servers copy the `k3s.yaml` file to your local machine (example to $HOME/.kube/k3s.yaml).  
Edit the `k3s.yaml` file and correct the server address with the virtual IP address of load balancer.   
The `KUBECONFIG` environment variable can now be used to specify the `k3s.yaml` file and use it with `kubectl`.

```bash
export KUBECONFIG=$HOME/.kube/k3s.yaml

kubectl get nodes
NAME            STATUS   ROLES                       AGE     VERSION
k3s-server-01   Ready    control-plane,etcd,master   3h12m   v1.31.5+k3s1
k3s-server-02   Ready    control-plane,etcd,master   149m    v1.31.5+k3s1
k3s-server-03   Ready    control-plane,etcd,master   117m    v1.31.5+k3s1
k3s-worker-01   Ready    <none>                      4m2s    v1.31.5+k3s1
k3s-worker-02   Ready    <none>                      3m31s   v1.31.5+k3s1
k3s-worker-03   Ready    <none>                      3m33s   v1.31.5+k3s1

kubectl get nodes -o wide
NAME            STATUS   ROLES                       AGE     VERSION        INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
k3s-server-01   Ready    control-plane,etcd,master   3h12m   v1.31.5+k3s1   172.16.12.22   <none>        Ubuntu 24.04.1 LTS   6.8.0-52-generic   containerd://1.7.23-k3s2
k3s-server-02   Ready    control-plane,etcd,master   150m    v1.31.5+k3s1   172.16.12.23   <none>        Ubuntu 24.04.1 LTS   6.8.0-52-generic   containerd://1.7.23-k3s2
k3s-server-03   Ready    control-plane,etcd,master   117m    v1.31.5+k3s1   172.16.12.24   <none>        Ubuntu 24.04.1 LTS   6.8.0-52-generic   containerd://1.7.23-k3s2
k3s-worker-01   Ready    <none>                      4m40s   v1.31.5+k3s1   172.16.12.25   <none>        Ubuntu 24.04.1 LTS   6.8.0-52-generic   containerd://1.7.23-k3s2
k3s-worker-02   Ready    <none>                      4m9s    v1.31.5+k3s1   172.16.12.26   <none>        Ubuntu 24.04.1 LTS   6.8.0-52-generic   containerd://1.7.23-k3s2
k3s-worker-03   Ready    <none>                      4m11s   v1.31.5+k3s1   172.16.12.27   <none>        Ubuntu 24.04.1 LTS   6.8.0-52-generic   containerd://1.7.23-k3s2
```

## Tainting Control Plane Nodes

To prevent workloads from running on the control plane nodes, you can taint them.  
This ensures that only specific pods with the corresponding tolerations can be scheduled on these nodes.  
Run the following `kubectl` commands to taint your control plane nodes:

```
kubectl taint nodes k3s-server-01 node-role.kubernetes.io/control-plane:NoSchedule
kubectl taint nodes k3s-server-02 node-role.kubernetes.io/control-plane:NoSchedule
kubectl taint nodes k3s-server-03 node-role.kubernetes.io/control-plane:NoSchedule
```

This will prevent workloads from being scheduled on the `k3s-server-*` nodes unless they have a matching toleration.