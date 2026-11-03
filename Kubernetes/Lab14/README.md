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

The MySQL pod was successfully scheduled on `node01`.

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
kubectl exec -it ivolvedb-0 -n ivolve -- mysql -uroot -proot123 -e "SELECT VERSION();"
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

The service successfully resolved to the MySQL pod address.

## Result

The MySQL StatefulSet was successfully deployed with persistent storage, node scheduling constraints, configuration from ConfigMap and Secret, and Headless Service DNS resolution.<img width="1366" height="768" alt="384b941d-75db-45af-b388-227c172832ec" src="https://github.com/user-attachments/assets/0d5375b1-4761-49c7-ba9f-072311fa0973" />
<img width="1366" height="768" alt="d3b062eb-b0f2-4da9-b4c8-4a7975a846d9" src="https://github.com/user-attachments/assets/d73f194c-fb96-403d-a3a5-6a49af828c66" />
<img width="1366" height="768" alt="12281829-70c3-4382-b330-14c42a73ae6c" src="https://github.com/user-attachments/assets/a24a8698-f9ea-470c-9e7e-26b64976d3fa" />
