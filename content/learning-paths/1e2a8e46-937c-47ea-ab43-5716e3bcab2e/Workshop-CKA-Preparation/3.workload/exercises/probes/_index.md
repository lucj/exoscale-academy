---
title: Probes
---

## Exercise

1. Create the specification of a Pod based on the *stefanprodan/podinfo* image

2. Add a liveness probe checking the /healthz on port 9898 every 10 seconds after an initial delay of 30 seconds

3. Add a readiness probe checking the /readyz on port 9898 every 5 seconds after an initial delay of 30 seconds

4. Delete the Pod

## Documentation

[https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

## Solution

1. Create the specification of a Pod based on the *stefanprodan/podinfo* image

```
kubectl run podinfo --image=stefanprodan/podinfo --dry-run=client -o yaml > podinfo.yaml
```

2. Add a liveness probe checking the /healthz on port 9898 every 10 seconds

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: podinfo
  name: podinfo
spec:
  containers:
  - image: stefanprodan/podinfo
    name: podinfo
    livenessProbe:
      httpGet:
        path: /healthz
        port: 9898
      periodSeconds: 10
      initialDelaySeconds: 30
```

3. Add a readiness probe checking the /readyz on port 9898 every 5 seconds

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: podinfo
  name: podinfo
spec:
  containers:
  - image: stefanprodan/podinfo
    name: podinfo
    livenessProbe:
      httpGet:
        path: /healthz
        port: 9898
      periodSeconds: 10
      initialDelaySeconds: 30
    readinessProbe:
      httpGet:
        path: /readyz
        port: 9898
      periodSeconds: 5
      initialDelaySeconds: 30
```

4. Delete the Pod

```
k delete po podinfo
```


