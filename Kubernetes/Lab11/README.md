![Uploading 10487ea2-c048-432f-bd18-09ed7bd2f2ef.jpg…]()
# Lab 11 - Namespace and ResourceQuota

## Objective
Create a dedicated Kubernetes namespace and restrict the maximum number of pods using a ResourceQuota.

## Namespace

The namespace used in this lab is:

```text
ivolve
```

Create it using:

```bash
kubectl create namespace ivolve
```

## ResourceQuota

A ResourceQuota named `ivolvequota` was created to limit the namespace to a maximum of 3 pods.

### quota.yaml

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ivolvequota
  namespace: ivolve
spec:
  hard:
    pods: "3"
```

Apply the configuration:

```bash
kubectl apply -f quota.yaml
```

## Verification

```bash
kubectl get resourcequota -n ivolve
```

## Testing the Pod Limit

Three pods were created successfully:

```bash
kubectl run pod1 --image=nginx -n ivolve
kubectl run pod2 --image=nginx -n ivolve
kubectl run pod3 --image=nginx -n ivolve
```

Verify:

```bash
kubectl get pods -n ivolve
```

A fourth pod was then attempted:

```bash
kubectl run pod4 --image=nginx -n ivolve
```

Kubernetes rejected the fourth pod because the namespace had reached its quota:

```text
exceeded quota: ivolvequota
requested: pods=1
used: pods=3
limited: pods=3
```

## Result
The ResourceQuota successfully restricted the namespace to a maximum of 3 pods.
