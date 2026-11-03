Lab 13 - PersistentVolume and PersistentVolumeClaim

Objective

Create persistent storage using a PersistentVolume and PersistentVolumeClaim.

PersistentVolume

The PersistentVolume was configured with:

Capacity: 1Gi

Access Mode: ReadWriteMany

Reclaim Policy: Retain

Host Path: /mnt/app-logs

No StorageClass

PersistentVolumeClaim

The PVC requests:

Capacity: 1Gi

Access Mode: ReadWriteMany

No StorageClass

pv-pvc.yaml

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

Apply:

kubectl apply -f pv-pvc.yaml

Verification

kubectl get pv
kubectl get pvc -n ivolve

The PV and PVC reached the Bound state.

Additional verification:

kubectl describe pv ivolvepv
kubectl describe pvc ivolvepvc -n ivolve

Result

The PVC was successfully bound to the PersistentVolume.
