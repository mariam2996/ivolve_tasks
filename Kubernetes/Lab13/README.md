# Lab 13 - PersistentVolume and PersistentVolumeClaim

## Objective

Create persistent storage using a PersistentVolume and PersistentVolumeClaim.

## PersistentVolume

The PersistentVolume was configured with:

- Capacity: `1Gi`
- Access Mode: `ReadWriteMany`
- Reclaim Policy: `Retain`
- Host Path: `/mnt/app-logs`
- No StorageClass

## PersistentVolumeClaim

The PVC requests:

- Capacity: `1Gi`
- Access Mode: `ReadWriteMany`
- No StorageClass

## pv-pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ivolvepv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""
  hostPath:
    path: /mnt/app-logs
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ivolvepvc
  namespace: ivolve
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: ""
```

Apply:

```bash
kubectl apply -f pv-pvc.yaml
```

## Verification

```bash
kubectl get pv
kubectl get pvc -n ivolve
```

The PV and PVC reached the `Bound` state.

Additional verification:

```bash
kubectl describe pv ivolvepv
kubectl describe pvc ivolvepvc -n ivolve
```

## Result

The PersistentVolumeClaim was successfully bound to the PersistentVolume.<img width="1366" height="768" alt="2824853b-41d6-4210-b61e-9379284762f3" src="https://github.com/user-attachments/assets/eebe47f9-ba76-4b42-9bc4-2c8b55514414" />
