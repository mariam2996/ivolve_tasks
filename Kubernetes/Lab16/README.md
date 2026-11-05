# Lab 16 - Pod Resource Management with CPU and Memory Requests and Limits

## Objective

Update the existing `ivolveapp` Deployment to configure CPU and memory requests and limits.

## Resource Requests

- CPU: `1`
- Memory: `1Gi`

## Resource Limits

- CPU: `2`
- Memory: `2Gi`

## Configuration

The Deployment was updated using:

```bash
kubectl set resources deployment ivolveapp \
  -n ivolve \
  --containers=ivolveapp \
  --requests=cpu=1,memory=1Gi \
  --limits=cpu=2,memory=2Gi
```

## Verification

The configured resources can be verified using:

```bash
kubectl get deployment ivolveapp -n ivolve -o yaml
```

or:

```bash
kubectl describe pod <pod-name> -n ivolve
```

The expected values are:

```text
Requests:
  cpu:     1
  memory:  1Gi

Limits:
  cpu:     2
  memory:  2Gi
```

## Real-Time Monitoring

Real-time resource usage can be checked using:

```bash
kubectl top pod -n ivolve
```

This command requires Metrics Server to be available in the Kubernetes cluster.

## Environment Limitation

The resource configuration was applied successfully, but the Killercoda environment did not have enough available CPU to schedule all updated replicas.

The scheduler reported:

```text
Insufficient cpu
```

The configured resource values were kept unchanged because they are required by the lab specifications.

## Result

CPU and memory requests and limits were successfully configured on the `ivolveapp` Deployment. Full scheduling of all replicas was limited by the available CPU capacity of the Killercoda environment.
