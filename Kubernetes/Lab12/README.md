# Lab 12 - ConfigMap and Secret

## Objective
Store application configuration in a ConfigMap and sensitive values in a Kubernetes Secret.

## ConfigMap

### configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ivolvecm
  namespace: ivolve
data:
  DB_HOST: ivolvedbsvc
  DB_USER: root
```

Apply:

```bash
kubectl apply -f configmap.yaml
```

Verify:

```bash
kubectl get configmap ivolvecm -n ivolve
kubectl describe configmap ivolvecm -n ivolve
```

## Secret

The passwords were stored using Base64 encoding.

`root123` encoded in Base64:

```text
cm9vdDEyMw==
```

### secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ivolvesecret
  namespace: ivolve
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: cm9vdDEyMw==
  DB_PASSWORD: cm9vdDEyMw==
```

Apply:

```bash
kubectl apply -f secret.yaml
```

Verify:

```bash
kubectl get secret ivolvesecret -n ivolve
kubectl get secret ivolvesecret -n ivolve -o yaml
```

## Result
The database configuration was successfully stored using a ConfigMap and Secret.

---

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

### pv-pvc.yaml

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
The PVC was successfully bound to the PersistentVolume.

---

# Lab 14 - MySQL StatefulSet and Headless Service

## Objective
Deploy a MySQL database using a StatefulSet with persistent storage, ConfigMap and Secret configuration, node scheduling constraints, and a Headless Service.

## Components

The deployment uses:

- StatefulSet: `ivolvedb`
- Headless Service: `ivolvedbsvc`
- Image: `mysql:8.0`
- ConfigMap: `ivolvecm`
- Secret: `ivolvesecret`
- Node Selector: `project=ivolve`
- Toleration: `workload=ivolveApp:NoSchedule`
- Persistent storage mounted at `/var/lib/mysql`

## MySQL PersistentVolume

### mysql-pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ivolvedbpv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""
  hostPath:
    path: /mnt/mysql-data
```

Apply:

```bash
kubectl apply -f mysql-pv.yaml
```

## StatefulSet and Headless Service

### statefulset.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ivolvedbsvc
  namespace: ivolve
spec:
  clusterIP: None
  selector:
    app: ivolvedb
  ports:
    - port: 3306
      targetPort: 3306
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: ivolvedb
  namespace: ivolve
spec:
  serviceName: ivolvedbsvc
  replicas: 1

  selector:
    matchLabels:
      app: ivolvedb

  template:
    metadata:
      labels:
        app: ivolvedb

    spec:
      nodeSelector:
        project: ivolve

      tolerations:
        - key: workload
          operator: Equal
          value: ivolveApp
          effect: NoSchedule

      containers:
        - name: mysql
          image: mysql:8.0

          ports:
            - containerPort: 3306

          env:
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: ivolvecm
                  key: DB_HOST

            - name: DB_USER
              valueFrom:
                configMapKeyRef:
                  name: ivolvecm
                  key: DB_USER

            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ivolvesecret
                  key: MYSQL_ROOT_PASSWORD

            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: ivolvesecret
                  key: DB_PASSWORD

          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql

  volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        accessModes:
          - ReadWriteOnce
        storageClassName: ""
        resources:
          requests:
            storage: 1Gi
```

Apply:

```bash
kubectl apply -f statefulset.yaml
```

## Verification

Check the pod:

```bash
kubectl get pods -n ivolve -o wide
```

The MySQL pod was scheduled on `node01`.

Check the StatefulSet:

```bash
kubectl get statefulset -n ivolve
```

Check the Headless Service:

```bash
kubectl get svc ivolvedbsvc -n ivolve
```

The service has:

```text
CLUSTER-IP: None
```

Check persistent storage:

```bash
kubectl get pvc -n ivolve
```

The StatefulSet PVC was successfully bound to `ivolvedbpv`.

## MySQL Test

```bash
kubectl exec -it ivolvedb-0 -n ivolve -- \
mysql -uroot -proot123 -e "SELECT VERSION();"
```

The MySQL server responded successfully with version:

```text
8.0.46
```

## DNS Test

The Headless Service DNS was verified using:

```bash
kubectl run dns-test \
  --rm -it \
  --restart=Never \
  --image=busybox:1.36 \
  -n ivolve \
  -- nslookup ivolvedbsvc
```

The service resolved successfully to the MySQL pod address.

## Result
The MySQL StatefulSet was successfully deployed with persistent storage, node scheduling constraints, configuration from ConfigMap and Secret, and Headless Service DNS resolution.
