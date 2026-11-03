Lab 14 - MySQL StatefulSet and Headless Service

Objective

Deploy a MySQL database using a StatefulSet with persistent storage, ConfigMap and Secret configuration, node scheduling constraints, and a Headless Service.

Components

The deployment uses:

StatefulSet: ivolvedb

Headless Service: ivolvedbsvc

Image: mysql:8.0

ConfigMap: ivolvecm

Secret: ivolvesecret

Node Selector: project=ivolve

Toleration: workload=ivolveApp:NoSchedule

Persistent storage mounted at /var/lib/mysql

MySQL PersistentVolume

mysql-pv.yaml

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

Apply:

kubectl apply -f mysql-pv.yaml

StatefulSet and Headless Service

statefulset.yaml

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

Apply:

kubectl apply -f statefulset.yaml

Verification

Check the pod:

kubectl get pods -n ivolve -o wide

The MySQL pod was scheduled on node01.

Check the StatefulSet:

kubectl get statefulset -n ivolve

Check the Headless Service:

kubectl get svc ivolvedbsvc -n ivolve

The service has:

CLUSTER-IP: None

Check persistent storage:

kubectl get pvc -n ivolve

The StatefulSet PVC was successfully bound to ivolvedbpv.

MySQL Test

kubectl exec -it ivolvedb-0 -n ivolve -- \
mysql -uroot -proot123 -e "SELECT VERSION();"

The MySQL server responded successfully with version:

8.0.46

DNS Test

The Headless Service DNS was verified using:

kubectl run dns-test \
  --rm -it \
  --restart=Never \
  --image=busybox:1.36 \
  -n ivolve \
  -- nslookup ivolvedbsvc

The service resolved successfully to the MySQL pod address.

Result

The MySQL StatefulSet was successfully deployed with persistent storage, node scheduling constraints, configuration from ConfigMap and Secret, and Headless Service DNS resolution.
