<img width="1366" height="768" alt="1c546a67-4343-49b9-ae7d-3847a9a85c4a" src="https://github.com/user-attachments/assets/07bfab92-f25d-4cd8-b039-1cc6b62efc9f" />
# Lab 19 - Securing Kubernetes with RBAC and Service Accounts

## Objective

Secure Kubernetes access using a ServiceAccount, Role, and RoleBinding.

The goal is to allow a Jenkins service account to create and update Deployments only inside the `ivolve` namespace.

## Service Account

A ServiceAccount named `jenkinssa` was created in the `ivolve` namespace.

```bash
kubectl create serviceaccount jenkinssa -n ivolve
```

A token was generated using:

```bash
kubectl create token jenkinssa -n ivolve
```

## Role

A Role named `ivolvedeployrole` was created with permissions to:

- Create Deployments
- Update Deployments

The permissions are restricted to the `ivolve` namespace.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: ivolvedeployrole
  namespace: ivolve
rules:
  - apiGroups:
      - apps
    resources:
      - deployments
    verbs:
      - create
      - update
```

## RoleBinding

A RoleBinding named `ivolvedeployrolebinding` was created to bind the Role to the `jenkinssa` ServiceAccount.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ivolvedeployrolebinding
  namespace: ivolve
subjects:
  - kind: ServiceAccount
    name: jenkinssa
    namespace: ivolve
roleRef:
  kind: Role
  name: ivolvedeployrole
  apiGroup: rbac.authorization.k8s.io
```

## Verification

Verify that the ServiceAccount can create Deployments:

```bash
kubectl auth can-i create deployments \
  --as=system:serviceaccount:ivolve:jenkinssa \
  -n ivolve
```

Expected result:

```text
yes
```

Verify update permission:

```bash
kubectl auth can-i update deployments \
  --as=system:serviceaccount:ivolve:jenkinssa \
  -n ivolve
```

Expected result:

```text
yes
```

Verify that delete permission is not allowed:

```bash
kubectl auth can-i delete deployments \
  --as=system:serviceaccount:ivolve:jenkinssa \
  -n ivolve
```

Expected result:

```text
no
```

Verify that the ServiceAccount cannot create Deployments in another namespace:

```bash
kubectl auth can-i create deployments \
  --as=system:serviceaccount:ivolve:jenkinssa \
  -n default
```

Expected result:

```text
no
```

## Result

The Jenkins ServiceAccount was successfully restricted using Kubernetes RBAC.

It can create and update Deployments only inside the `ivolve` namespace and does not have delete permissions or Deployment access in other namespaces.
