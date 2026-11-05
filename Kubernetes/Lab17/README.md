# Lab 17 - Control Pod-to-Pod Traffic via Network Policy

## Objective

Create a Kubernetes NetworkPolicy that restricts access to the MySQL pod so that only the application pods can connect to it on port `3306`.

## Configuration

The NetworkPolicy uses:

- Name: `ivolvenp`
- Namespace: `ivolve`
- Target Pod Selector: `app=ivolvedb`
- Policy Type: `Ingress`
- Allowed Source: Pods with label `app=ivolveapp`
- Allowed Port: `3306/TCP`

## NetworkPolicy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ivolvenp
  namespace: ivolve
spec:
  podSelector:
    matchLabels:
      app: ivolvedb
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: ivolveapp
      ports:
        - protocol: TCP
          port: 3306
```

## Apply

```bash
kubectl apply -f lab17-networkpolicy.yaml
```

## Verification

Check the NetworkPolicy:

```bash
kubectl get networkpolicy -n ivolve
```

Describe the policy:

```bash
kubectl describe networkpolicy ivolvenp -n ivolve
```

The policy allows inbound traffic to the MySQL pod only from application pods labeled:

```text
app=ivolveapp
```

and only on:

```text
3306/TCP
```

## Allowed Traffic Test

A test pod with the application label was used to verify that access to MySQL is allowed.

```bash
kubectl exec -it ivolveapp-test -n ivolve -- \
mysql -h ivolvedbsvc -uroot -proot123 -e "SELECT 1;"
```

## Blocked Traffic Test

A pod without the `app=ivolveapp` label was used to verify that unauthorized traffic is blocked.

```bash
kubectl exec -it blocked-test -n ivolve -- \
mysql -h ivolvedbsvc -uroot -proot123 --connect-timeout=5 -e "SELECT 1;"
```

The unauthorized pod was unable to connect to MySQL.

## Result

The NetworkPolicy successfully restricted MySQL ingress traffic to application pods only and limited access to port `3306`.
