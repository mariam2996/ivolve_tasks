<img width="1366" height="768" alt="6ebccb74-95a0-4be7-8d1a-1870a7110b2e" src="https://github.com/user-attachments/assets/2907da72-37bf-4933-b4e7-503b244884fd" />
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

