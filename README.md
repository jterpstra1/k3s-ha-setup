# K3s High Availability (HA) Setup

## Introduction

This project involves deploying a Kubernetes cluster using `K3s` on a set of 5 virtual machines (VMs). The VMs are
running on Proxmox hosted on an old x86 PC with 16Gb of memory. The setup is designed to provide a lightweight,
efficient and scalable Kubernetes environment for testing and development.  
The cluster consists of 3 control plane nodes and 2 worker nodes, providing a solid foundation for deploying and
managing containerized applications. The use of `K3s` ensures minimal resource overhead, making it ideal for
environments with limited hardware resources.

Project gracioulsy provided by [Wim VDW](https://github.com/wim-vdw)
## VM overview

| VM Name       | MAC Address       | IP Address    | Memory (Gb) | Disk Size (Gb) | CPU Cores |
|---------------|-------------------|---------------|-------------|----------------|-----------|
| k3s-server-01 | BC:24:11:66:6F:07 | 192.168.1.205 | 4           | 32             | 2         |
| k3s-server-02 | BC:24:11:02:57:C8 | 192.168.1.206 | 4           | 32             | 2         |
| k3s-server-03 | BC:24:11:4F:A3:86 | 192.168.1.207 | 4           | 32             | 2         |
| k3s-worker-01 | BC:24:11:12:B1:D2 | 192.168.1.210 | 8           | 32             | 4         |
| k3s-worker-02 | BC:24:11:07:BA:1A | 192.168.1.211 | 8           | 32             | 4         |
| k3s-worker-03 | BC:24:11:07:BA:1A | 192.168.1.212 | 8           | 32             | 4         |
| k3s-lb-01     | BC:24:11:07:BA:1A | 192.168.1.201 | 2           | 32             | 2         |
| k3s-lb-02     | BC:24:11:07:BA:1A | 192.168.1.202 | 2           | 32             | 2         |

VIP Address: 192.168.1.200

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

qm clone 9000 201 --name k3s-lb-01
qm resize 201 scsi0 32G
qm set 201 --net0 virtio,bridge=vmbr0
qm set 201 --cpu host
qm set 201 --memory 2048
qm set 201 --cores 2
qm set 201 --ostype l26
qm set 201 --tags k3s,prd,lb,192.168.1.201
qm set 201 --localtime 1
qm set 201 --onboot 1
qm set 201 --ciuser jurre
qm set 201 --sshkey ~/.ssh/pubkey.pub
qm set 201 --ciupgrade 1
qm set 201 --ipconfig0 ip=192.168.1.201/24,gw=192.168.1.201

qm clone 9000 202 --name k3s-lb-02
qm resize 202 scsi0 32G
qm set 202 --net0 virtio,bridge=vmbr0
qm set 202 --cpu host
qm set 202 --memory 2048
qm set 202 --cores 2
qm set 202 --ostype l26
qm set 202 --tags k3s,prd,lb,192.168.1.202
qm set 202 --localtime 1
qm set 202 --onboot 1
qm set 202 --ciuser jurre
qm set 202 --sshkey ~/.ssh/pubkey.pub
qm set 202 --ciupgrade 1
qm set 202 --ipconfig0 ip=192.168.1.202/24,gw=192.168.1.1

qm clone 9000 205 --name k3s-server-01
qm resize 205 scsi0 32G
qm set 205 --net0 virtio,bridge=vmbr0
qm set 205 --cpu host
qm set 205 --memory 4096
qm set 205 --cores 2
qm set 205 --ostype l26
qm set 205 --tags k3s,prd,server,192.168.1.205
qm set 205 --localtime 1
qm set 205 --onboot 1
qm set 205 --ciuser jurre
qm set 205 --sshkey ~/.ssh/pubkey.pub
qm set 205 --ciupgrade 1
qm set 205 --ipconfig0 ip=192.168.1.205/24,gw=192.168.1.1

qm clone 9000 206 --name k3s-server-02
qm resize 206 scsi0 32G
qm set 206 --net0 virtio,bridge=vmbr0
qm set 206 --cpu host
qm set 206 --memory 4096
qm set 206 --cores 2
qm set 206 --ostype l26
qm set 206 --tags k3s,prd,server,192.168.1.206
qm set 206 --localtime 1
qm set 206 --onboot 1
qm set 206 --ciuser jurre
qm set 206 --sshkey ~/.ssh/pubkey.pub
qm set 206 --ciupgrade 1
qm set 206 --ipconfig0 ip=192.168.1.206/24,gw=192.168.1.1

qm clone 9000 207 --name k3s-server-03
qm resize 207 scsi0 32G
qm set 207 --net0 virtio,bridge=vmbr0
qm set 207 --cpu host
qm set 207 --memory 4096
qm set 207 --cores 2
qm set 207 --ostype l26
qm set 207 --tags k3s,prd,server,192.168.1.207
qm set 207 --localtime 1
qm set 207 --onboot 1
qm set 207 --ciuser jurre
qm set 207 --sshkey ~/.ssh/pubkey.pub
qm set 207 --ciupgrade 1
qm set 207 --ipconfig0 ip=192.168.1.207/24,gw=192.168.1.1

qm clone 9000 210 --name k3s-worker-01
qm resize 210 scsi0 32G
qm set 210 --net0 virtio,bridge=vmbr0
qm set 210 --cpu host
qm set 210 --memory 8192
qm set 210 --cores 4
qm set 210 --ostype l26
qm set 210 --tags k3s,prd,worker,192.168.1.210
qm set 210 --localtime 1
qm set 210 --onboot 1
qm set 210 --ciuser jurre
qm set 210 --sshkey ~/.ssh/pubkey.pub
qm set 210 --ciupgrade 1
qm set 210 --ipconfig0 ip=192.168.1.210/24,gw=192.168.1.1

qm clone 9000 211 --name k3s-worker-02
qm resize 211 scsi0 32G
qm set 211 --net0 virtio,bridge=vmbr0
qm set 211 --cpu host
qm set 211 --memory 8192
qm set 211 --cores 4
qm set 211 --ostype l26
qm set 211 --tags k3s,prd,worker,192.168.1.211
qm set 211 --localtime 1
qm set 211 --onboot 1
qm set 211 --ciuser jurre
qm set 211 --sshkey ~/.ssh/pubkey.pub
qm set 211 --ciupgrade 1
qm set 211 --ipconfig0 ip=192.168.1.211/24,gw=192.168.1.1

qm clone 9000 212 --name k3s-worker-03
qm resize 212 scsi0 32G
qm set 212 --net0 virtio,bridge=vmbr0
qm set 212 --cpu host
qm set 212 --memory 8192
qm set 212 --cores 4
qm set 212 --ostype l26
qm set 212 --tags k3s,prd,worker,192.168.1.212
qm set 212 --localtime 1
qm set 212 --onboot 1
qm set 212 --ciuser jurre
qm set 212 --sshkey ~/.ssh/pubkey.pub
qm set 212 --ciupgrade 1
qm set 212 --ipconfig0 ip=192.168.1.212/24,gw=192.168.1.1

qm start 201
qm start 202
qm start 205
qm start 206
qm start 222
qm start 210
qm start 211
qm start 212
```

## Prepare VMS for use in homelab environment
A few agents (qemu-guest-agent and elastic agent (for now)) need to be installed before setting up the `K3s` cluster. This is done on all VMs. 
```bash
# install qemu-guest-agent
sudo apt-get install qemu-guest-agent
```

## Install a cluster load balancer (HAProxy) on the two additional servers
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
    server server-1 192.168.1.205:6443 check
    server server-2 192.168.1.206:6443 check
    server server-3 192.168.1.207:6443 check
```

Add the following to /etc/keepalived/keepalived.conf on k3s-lb-01 (MASTER):
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
    state MASTER
    priority 200

    virtual_router_id 51

    virtual_ipaddress {
        192.168.1.220/24
    }

    track_script {
        chk_haproxy
    }
}
```

Add the following to /etc/keepalived/keepalived.conf on k3s-lb-02 (BACKUP):
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
    state BACKUP
    priority 100

    virtual_router_id 51

    virtual_ipaddress {
        192.168.1.220/24
    }

    track_script {
        chk_haproxy
    }
}
```

Restart HAProxy and Keepalived on k3s-lb-01 and k3s-lb-02:
```bash
sudo systemctl restart haproxy
sudo systemctl restart keepalived
```

## Install first server (control plane node)

Use the `--cluster-init` flag to create the first server in the cluster and initialize the embedded `etcd` datastore for
high availability (HA).  
Use the `--tls-san` flag to specify the fixed IP address that will be used by Keepalived as virtual IP to access the cluster.
To install a specific version of `K3s`, set the `INSTALL_K3S_VERSION` environment variable before running the
installation
script.  
All `K3s` versions can be found here: [k3s release](https://github.com/k3s-io/k3s/releases)

```bash
# Use a specific version of K3s.
export INSTALL_K3S_VERSION=v1.33.4+k3s1

# Installs K3s, initializes the first server in HA mode and disables Traefik and the ServiceLB load balancer.
sudo curl -sfL https://get.k3s.io | sh -s - server --cluster-init --disable="traefik" --disable="servicelb" --tls-san=192.168.1.220

# Displays the kubeconfig file for accessing the K3s cluster.
sudo cat /etc/rancher/k3s/k3s.yaml

# Displays the token used for joining nodes to the K3s cluster.
sudo cat /var/lib/rancher/k3s/server/token
```

## Install additional servers (control plane nodes)

```bash
export INSTALL_K3S_VERSION=v1.33.4+k3s1
export K3S_URL=https://192.168.1.205:6443
export K3S_TOKEN=<token>

sudo curl -sfL https://get.k3s.io |sh -s - server --disable="traefik" --disable="servicelb" --tls-san=192.168.1.220
```

## Install agents (worker nodes)

```bash
export INSTALL_K3S_VERSION=v1.33.4+k3s1
export K3S_URL=https://192.168.1.220:6443
export K3S_TOKEN=<token>

sudo curl -sfL https://get.k3s.io | sh -s - agent
```

## Set up kubectl to use the K3s kubeconfig file

From one of the servers copy the `k3s.yaml` file to your local machine (example to $HOME/.kube/k3s.yaml).  
Edit the `k3s.yaml` file and correct the server address with the `Keepalived` virtual IP address 192.168.1.220, this ensures high availability for the Kubernetes API Server.
The `KUBECONFIG` environment variable can now be used to specify the `k3s.yaml` file and use it with `kubectl`.

```bash
export KUBECONFIG=$HOME/.kube/k3s.yaml

❯ kubectl get nodes
NAME            STATUS   ROLES                       AGE     VERSION
k3s-server-01   Ready    control-plane,etcd,master   11m     v1.33.4+k3s1
k3s-server-02   Ready    control-plane,etcd,master   6m56s   v1.33.4+k3s1
k3s-server-03   Ready    control-plane,etcd,master   6m25s   v1.33.4+k3s1
k3s-worker-01   Ready    <none>                      4m16s   v1.33.4+k3s1
k3s-worker-02   Ready    <none>                      3m23s   v1.33.4+k3s1
k3s-worker-03   Ready    <none>                      2m58s   v1.33.4+k3s1

❯ kubectl get nodes -o wide
NAME            STATUS   ROLES                       AGE     VERSION        INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
k3s-server-01   Ready    control-plane,etcd,master   12m     v1.33.4+k3s1   192.168.1.205   <none>        Ubuntu 24.04.3 LTS   6.8.0-79-generic   containerd://2.0.5-k3s2
k3s-server-02   Ready    control-plane,etcd,master   7m22s   v1.33.4+k3s1   192.168.1.206   <none>        Ubuntu 24.04.3 LTS   6.8.0-79-generic   containerd://2.0.5-k3s2
k3s-server-03   Ready    control-plane,etcd,master   6m51s   v1.33.4+k3s1   192.168.1.207   <none>        Ubuntu 24.04.3 LTS   6.8.0-79-generic   containerd://2.0.5-k3s2
k3s-worker-01   Ready    <none>                      4m42s   v1.33.4+k3s1   192.168.1.210   <none>        Ubuntu 24.04.3 LTS   6.8.0-79-generic   containerd://2.0.5-k3s2
k3s-worker-02   Ready    <none>                      3m49s   v1.33.4+k3s1   192.168.1.211   <none>        Ubuntu 24.04.3 LTS   6.8.0-79-generic   containerd://2.0.5-k3s2
k3s-worker-03   Ready    <none>                      3m24s   v1.33.4+k3s1   192.168.1.212   <none>        Ubuntu 24.04.3 LTS   6.8.0-79-generic   containerd://2.0.5-k3s2
```

## Tainting Control Plane Nodes

To prevent workloads from running on the control plane nodes, you can taint them.  
This ensures that only specific pods with the corresponding tolerations can be scheduled on these nodes.  
Run the following `kubectl` commands to taint your control plane nodes:

```
sudo kubectl taint nodes k3s-server-01 node-role.kubernetes.io/control-plane:NoSchedule
sudo kubectl taint nodes k3s-server-02 node-role.kubernetes.io/control-plane:NoSchedule
sudo kubectl taint nodes k3s-server-03 node-role.kubernetes.io/control-plane:NoSchedule
```

This will prevent workloads from being scheduled on the `k3s-server-*` nodes unless they have a matching toleration.

## Bootstrapping with Argo CD and Deploying Workloads
Now that your cluster is set up, you can deploy workloads on it. However, before doing so it is recommended to bootstrap the cluster with Argo CD for easier and more efficient management of your Kubernetes resources.
Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes. By integrating it into your cluster, you can manage deployments from Git repositories and streamline your deployment process.
After bootstrapping with Argo CD, you can deploy your workloads seamlessly, with Argo CD taking care of the application management and lifecycle.

Argo CD setup and application deployment can be done using the following repositories:

- Argo CD installation and configuration: https://github.com/jterpstra1/argocd-setup
- Argo CD application definitions: https://github.com/jterpstra1/argocd-apps
- Kubernetes manifests or Helm charts used by Argo CD applications: https://github.com/jterpstra1/argocd-k8s-resources