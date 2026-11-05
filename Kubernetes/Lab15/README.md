# Lab 15 - Node.js Application Deployment with ClusterIP Service

## Objective

Deploy the Node.js application on Kubernetes using a Deployment with 2 replicas and expose it internally using a ClusterIP Service.

The application uses:

- Docker image: `mariamas32/ivolve-app:v1`
- Namespace: `ivolve`
- ConfigMap: `ivolvecm`
- Secret: `ivolvesecret`
- PersistentVolumeClaim: `ivolvepvc`
- MySQL Service: `ivolvedbsvc`

## Deployment Configuration

The Deployment is configured with:

- 2 replicas
- Node selector: `project=ivolve`
- Toleration:
  - key: `workload`
  - value: `ivolveApp`
  - effect: `NoSchedule`
- Persistent storage mounted at `/app/logs`
- Environment variables loaded from ConfigMap and Secret

An init container using the `mysql:8.0` image is used to:

- Wait until MySQL becomes available
- Create the `ivolve` database
- Create the application database user
- Grant the application user privileges on the `ivolve` database

## Apply

```bash
kubectl apply -f lab15.yaml
```

## Verification

Check the application pods:

```bash
kubectl get pods -n ivolve -o wide
```

Check the Deployment:

```bash
kubectl get deployment ivolveapp -n ivolve
```

Check the ClusterIP Service:

```bash
kubectl get svc ivolvaappsvc -n ivolve
```

## Database Verification

Verify the database was created:

```bash
kubectl exec -it ivolvedb-0 -n ivolve -- \
mysql -uroot -proot123 -e "SHOW DATABASES;"
```

Verify the application user:

```bash
kubectl exec -it ivolvedb-0 -n ivolve -- \
mysql -uroot -proot123 -e "SELECT user,host FROM mysql.user;"
```

Verify the privileges:

```bash
kubectl exec -it ivolvedb-0 -n ivolve -- \
mysql -uroot -proot123 -e "SHOW GRANTS FOR 'ivolveuser'@'%';"
```

The application user was granted privileges on the `ivolve` database.

## Application Logs

```bash
kubectl logs -l app=ivolveapp -n ivolve --prefix=true
```

The application logs confirmed successful connectivity to MySQL:

```text
Connected to MySQL and 'ivolve' DB found.
Server started on http://0.0.0.0:3000
```

## Result

The Node.js application was successfully deployed with 2 replicas, connected to MySQL, used Kubernetes configuration resources, and was exposed internally using a ClusterIP Service.
