# Installing K8s with kubeadm

- [Installing K8s with kubeadm](#installing-k8s-with-kubeadm)
  - [Summary](#summary)
  - [Software Components and Versions](#software-components-and-versions)
  - [Linux Virtual Machine](#linux-virtual-machine)
  - [Preparing the hosts for Kubernetes installation](#preparing-the-hosts-for-kubernetes-installation)
    - [Enable IPv4 packet forwarding](#enable-ipv4-packet-forwarding)
    - [Container Runtime](#container-runtime)
      - [Installing Container Runtime](#installing-container-runtime)
    - [Download and install K8s packages](#download-and-install-k8s-packages)
  - [Creating the K8s cluster with kubeadm](#creating-the-k8s-cluster-with-kubeadm)
    - [Control Plane](#control-plane)
    - [Worker](#worker)

## Summary

This tutorial demonstrates how to provision a Kubernetes cluster using `kubeadm` in a Linux machine. The steps and commands listed here are based on the Kubernetes documentation [Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

🟧 The idea of this tutorial is loosely based on the famous GitHub repository [kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way)

## Software Components and Versions

| Component                      | Version     | Source / Notes                                                                 |
|-------------------------------|-------------|--------------------------------------------------------------------------------|
| **Operating System**          | Ubuntu 20.04 LTS | Host OS for both `control plane` and `worker` nodes                             |
| **Kubernetes (kubeadm, kubelet, kubectl)** | v1.30.x     | Installed via Kubernetes official APT repository                              |
| **containerd**                | v2.0.4       | Installed manually from GitHub releases                                       |
| **runc**                      | v1.2.6       | Installed manually from [opencontainers/runc](https://github.com/opencontainers/runc) |
| **CNI plugins**               | v1.6.2       | From [containernetworking/plugins](https://github.com/containernetworking/plugins) |
| **Calico CNI plugin**         | v3.29.3      | Applied from [Calico's Kubernetes manifest](https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/calico.yaml)                                     |
| **systemd**                   | Native       | Used to manage `containerd` as a system service                               |
| **wget, curl, tar, gpg**      | OS defaults | Standard Linux utilities used during installation                             |

## Linux Virtual Machine

This tutorial requires:

* two (2) virtual machines running Ubuntu
* 2 GiB of RAM per machine
* 2 CPUs
* Full network connectivity among all machines in the cluster

The steps and commands presented here were tested on VMs using the following configuration:

| Role | OS | CPU | Memory|
|---|---|---|---|
| Control Plane | Ubuntu 20.04 Focal Fossa LTS | 2 vCPU | 2 GiB |
| Worker | Ubuntu 20.04 Focal Fossa LTS | 2 vCPU | 2 GiB |

Changing server cli prompt to easily identify the server you're logged into:

## Preparing the hosts for Kubernetes installation

The steps in this section must be performed on both servers.

### Enable IPv4 packet forwarding

```bash
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

Verify that net.ipv4.ip_forward is set to 1 with:

```bash
sysctl net.ipv4.ip_forward
```

### Container Runtime

This tutorial uses `containerd` version `v2.0.4` as the [Container Runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd).

🟧 For full information on how to install `containerd`, please refer to the [Getting started with containerd](https://github.com/containerd/containerd/blob/main/docs/getting-started.md) page.

#### Installing Container Runtime

1. Download `containerd`:

   ```bash
   wget https://github.com/containerd/containerd/releases/download/v2.0.4/containerd-static-2.0.4-linux-amd64.tar.gz
   ```

2. Extract the contents of the file:

   ```bash
   sudo tar Cxzvf /usr/local containerd-static-2.0.4-linux-amd64.tar.gz
   ```

3. To start `containerd` via [systemd](https://en.wikipedia.org/wiki/Systemd), download the `containerd.service`:

   ```bash
   wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
   ```

4. Copy the `containerd.service` to the `systemd` folder and enable it:

   ```bash
   sudo mkdir -p /usr/local/lib/systemd/system/
   sudo mv containerd.service /usr/local/lib/systemd/system/
   sudo systemctl daemon-reload
   sudo systemctl enable --now containerd
   ```

5. Download and install [runc](https://github.com/opencontainers/runc):

   ```bash
   wget https://github.com/opencontainers/runc/releases/download/v1.2.6/runc.amd64
   sudo install -m 755 runc.amd64 /usr/local/sbin/runc
   ```

6. Download and install the CNI plugins

   ```bash
   wget https://github.com/containernetworking/plugins/releases/download/v1.6.2/cni-plugins-linux-amd64-v1.6.2.tgz
   sudo mkdir -p /opt/cni/bin
   sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.6.2.tgz 
   ```

7. Generate the default configuration for `containerd`:

   ```bash
   containerd config default > config.toml
   sudo mkdir -p /etc/containerd
   sudo mv config.toml /etc/containerd/
   sudo systemctl restart containerd
   sudo systemctl status containerd
   ```

### Download and install K8s packages

Download and install `kubeadm`, `kubelet`, and `kubectl` as documented at [Download Kubernetes](https://kubernetes.io/releases/download/):

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p /etc/apt/keyrings/
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

## Creating the K8s cluster with kubeadm

There are two sets of commands to create the K8s cluster. One set for the `control plane` and other for the `worker`

### Control Plane

1. Initialize `kubeadm`:

   ```bash
   sudo kubeadm init --pod-network-cidr=192.168.0.0/16
   ```

2. To make `kubectl` work for your `non-root` user, run these commands, which are also part of the kubeadm init output:

   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

3. Deploy a [Container Network Interface (CNI)](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/):

   ```bash
   curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/calico.yaml -O
   kubectl apply -f calico.yaml
   ```

### Worker

Join the `worker` node to the cluster using the `join` command from the `kubeadm` output:

```bash
sudo kubeadm join <IP ADDRESS>:6443 --token <TOKEN> \
	--discovery-token-ca-cert-hash sha256:<SHA-256>
```

⚠️ Return to the Control Plane and attribute a label to the `worker` and confirm that the nodes are `Ready`:

```bash
kubectl label node <node-name> node-role.kubernetes.io/k8s-worker=
```

```bash
kubectl get nodes
NAME                           STATUS   ROLES           AGE    VERSION
<node-name>                    Ready    control-plane   3h3m   v1.30.11
<node-name>                    Ready    k8s-worker      166m   v1.30.11
```
