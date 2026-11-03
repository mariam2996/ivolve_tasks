<img width="1366" height="768" alt="35dd1fed-69c0-4aed-95e3-c47606ae45f5" src="https://github.com/user-attachments/assets/7fd57eaa-a829-4838-ba3d-588bb932c9cf" />
# Lab 10 - Node Taints and Labels

## Objective
Configure a Kubernetes worker node using a taint and a label to control workload scheduling.

## Environment
- Kubernetes Cluster
- Control Plane: `controlplane`
- Worker Node: `node01`

## Tasks Performed

### 1. Add a Taint to the Worker Node

```bash
kubectl taint nodes node01 workload=ivolveApp:NoSchedule
```

This prevents pods without the required toleration from being scheduled on `node01`.

### 2. Add a Label to the Worker Node

```bash
kubectl label nodes node01 project=ivolve
```

The label can later be used with `nodeSelector` to schedule specific workloads on this node.

## Verification

Verify the taint:

```bash
kubectl describe node node01 | grep -i taint
```

Expected result:

```text
Taints: workload=ivolveApp:NoSchedule
```

Verify the label:

```bash
kubectl get node node01 --show-labels
```

The node labels include:

```text
project=ivolve
```

## Result
The worker node was successfully configured with:
- Taint: `workload=ivolveApp:NoSchedule`
- Label: `project=ivolve`
