---
title: Application
---

## Exercise

This exercise covers common application failure scenarios.

### ImagePullBackOff Issues

1. Create a Pod with an invalid image name, check the Pod status and describe it.

```bash
kubectl run broken-image --image=nginx:nonexistent-tag
```

2. Fix the image reference using an existing image

### CrashLoopBackOff Issues

3. Create a Pod that exits immediately

```bash
kubectl run crash-pod --image=busybox --command -- /bin/sh -c "exit 1"
```

4. Examine the logs and fix the Pod to keep it running

## Documentation

- [https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)
- [https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)

## Solution

### ImagePullBackOff Issues

1. Create a Pod with an invalid image name, check the Pod status and describe it.

- creating the Pod

```bash
kubectl run broken-image --image=nginx:nonexistent-tag
```

- checking the Pods status shows ImagePullBackOff

```bash
kubectl get pod broken-image
```

- describing the Pod gives the origin of the error (image cannot be pulled)

```bash
kubectl describe pod broken-image
```

2. Fix the image reference using an existing image

```bash
kubectl delete pod broken-image
kubectl run broken-image --image=nginx:latest
```

### CrashLoopBackOff Issues

3. Create a Pod that exits immediately

Command provided in the instructions

4. Examine the logs and fix the Pod to keep it running

- getting the Pod shows an CrashLoopBackOff error

```bash
kubectl get pods crash-pod
```

- the logs don't provide any information

```bash
kubectl logs crash-pod
```

- describing the Pod shows the exit status 1

```bash
kubectl describe pod crash-pod
```

- fixing the specification using another command

```bash
kubectl delete pod crash-pod
kubectl run crash-pod --image=busybox --command -- /bin/sh -c "while true; do sleep 30; done"
```
