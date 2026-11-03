<img width="1366" height="768" alt="eeaeaf79-f457-4ee8-bc4c-db564208f26a" src="https://github.com/user-attachments/assets/3ae8f22b-aaef-48e4-82d8-059fabf52b1c" />
# Lab 9 - Kubernetes Cluster using kubeadm

## Objective
Deploy a Kubernetes cluster using kubeadm and configure the required Kubernetes components.

## Environment
- Ubuntu 24.04 LTS
- containerd
- Kubernetes v1.37.x
- Calico CNI
- Single control-plane node using Killercoda

## Steps Performed

1. Disabled swap.
2. Enabled required kernel modules:
   - overlay
   - br_netfilter
3. Configured required sysctl networking settings.
4. Installed and configured containerd.
5. Enabled `SystemdCgroup`.
6. Installed:
   - kubeadm
   - kubelet
   - kubectl
7. Initialized the Kubernetes control plane using:

```bash
kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --ignore-preflight-errors=NumCPU
```

8. Configured kubectl using `/etc/kubernetes/admin.conf`.
9. Installed Calico CNI.
10. Verified Kubernetes system components and node status.

## Verification

```bash
kubectl get nodes
kubectl get pods -A
systemctl is-active containerd
```

The control-plane node reached the `Ready` state and Kubernetes system components were running successfully.

## Limitation

The Killercoda environment used for this lab provided a single node only. Therefore, the worker-node `kubeadm join` step was not executed in this environment.

The `kubeadm join` command was generated during cluster initialization and can be executed on a separate worker node in a multi-node environment.
