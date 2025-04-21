# Upgrading K8s with kubeadm

- [Upgrading K8s with kubeadm](#upgrading-k8s-with-kubeadm)
  - [Summary](#summary)
  - [Software Components and Versions](#software-components-and-versions)
  - [Determine which version to upgrade to](#determine-which-version-to-upgrade-to)
  - [Upgrading control plane nodes](#upgrading-control-plane-nodes)
    - [Call "kubeadm upgrade"](#call-kubeadm-upgrade)
    - [Upgrade kubelet and kubectl](#upgrade-kubelet-and-kubectl)
  - [Upgrade worker nodes](#upgrade-worker-nodes)
    - [Upgrade kubeadm](#upgrade-kubeadm)
    - [Call "kubeadm upgrade node"](#call-kubeadm-upgrade-node)
    - [Upgrade kubelet and kubectl](#upgrade-kubelet-and-kubectl-1)

## Summary

This tutorial demonstrates how to upgrade a Kubernetes cluster originally installed using `kubeadm` in a Linux machine. The steps and commands listed here are based on the Kubernetes documentation [Upgrading kubeadm clusters](https://v1-31.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

🟧 This tutorial upgrades the K8s cluster to version `1.31.x`.

## Software Components and Versions

| Component                      | Version     | Source / Notes                                                                 |
|-------------------------------|-------------|--------------------------------------------------------------------------------|
| **Operating System**          | Ubuntu 20.04 LTS | Host OS for both `control plane` and `worker` nodes                             |
| **Kubernetes (kubeadm, kubelet, kubectl)** | v1.31.x     | Installed via Kubernetes official APT repository                              |
| **containerd**                | v2.0.4       | Installed manually from GitHub releases                                       |
| **runc**                      | v1.2.6       | Installed manually from [opencontainers/runc](https://github.com/opencontainers/runc) |
| **CNI plugins**               | v1.6.2       | From [containernetworking/plugins](https://github.com/containernetworking/plugins) |
| **Calico CNI plugin**         | v3.29.3      | Applied from [Calico's Kubernetes manifest](https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/calico.yaml)                                     |
| **systemd**                   | Native       | Used to manage `containerd` as a system service                               |
| **wget, curl, tar, gpg**      | OS defaults | Standard Linux utilities used during installation                             |

## Determine which version to upgrade to

Find the latest patch release for Kubernetes 1.31 using the OS package manager:

```bash
# Find the latest 1.31 version in the list.
# It should look like 1.31.x-*, where x is the latest patch.
sudo apt update
sudo apt-cache madison kubeadm
```

## Upgrading control plane nodes

The upgrade procedure on control plane nodes should be executed one node at a time. Pick a control plane node that you wish to upgrade first. It must have the `/etc/kubernetes/admin.conf` file.

### Call "kubeadm upgrade"

For the first control plane node

1. Upgrade kubeadm:

   ```bash
   # replace x in 1.31.x-* with the latest patch version
   sudo apt-mark unhold kubeadm && \
   sudo apt-get update && sudo apt-get install -y kubeadm='1.31.x-*' && \
   sudo apt-mark hold kubeadm
   ```

2. Verify that the download works and has the expected version:

   ```bash
   kubeadm version
   ```

3. Verify the upgrade plan:

   ```bash
   sudo kubeadm upgrade plan
   ```

   This command checks that your cluster can be upgraded, and fetches the versions you can upgrade to. It also shows a table with the component config version states.

4. Choose a version to upgrade to, and run the appropriate command. For example:

   ```bash
   # replace x with the patch version you picked for this upgrade
   sudo kubeadm upgrade apply v1.31.x
   ```

   Once the command finishes you should see:

   ```bash
   [upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.31.x". Enjoy!

   [upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
   ```

### Upgrade kubelet and kubectl

1. Upgrade the kubelet and kubectl:

   ```bash
   # replace x in 1.31.x-* with the latest patch version
   sudo apt-mark unhold kubelet kubectl && \
   sudo apt-get update && sudo apt-get install -y kubelet='1.31.x-*' kubectl='1.31.x-*' && \
   sudo apt-mark hold kubelet kubectl
   ```

2. Restart the kubelet:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

## Upgrade worker nodes

The upgrade procedure on worker nodes should be executed one node at a time or few nodes at a time, without compromising the minimum required capacity for running your workloads.

### Upgrade kubeadm

Upgrade kubeadm:

1. Upgrade kubeadm:

   ```bash
   # replace x in 1.31.x-* with the latest patch version
   sudo apt-mark unhold kubeadm && \
   sudo apt-get update && sudo apt-get install -y kubeadm='1.31.x-*' && \
   sudo apt-mark hold kubeadm
   ```

2. Verify that the download works and has the expected version:

   ```bash
   kubeadm version
   ```

### Call "kubeadm upgrade node"

For worker nodes this upgrades the local kubelet configuration:

```bash
sudo kubeadm upgrade node
```

### Upgrade kubelet and kubectl

1. Upgrade the kubelet and kubectl:

   ```bash
   # replace x in 1.31.x-* with the latest patch version
   sudo apt-mark unhold kubelet kubectl && \
   sudo apt-get update && sudo apt-get install -y kubelet='1.31.x-*' kubectl='1.31.x-*' && \
   sudo apt-mark hold kubelet kubectl
   ```

2. Restart the kubelet:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```
