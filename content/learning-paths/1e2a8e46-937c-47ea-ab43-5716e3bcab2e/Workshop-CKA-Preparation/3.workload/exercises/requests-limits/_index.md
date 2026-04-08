---
title: Resources
---

## Exercise

1. Create the specification of a Pod named *mongo* and based on the *mongo:5.0* image

2. Modify the specification to make sure the container has 64Mi of RAM and 30m cpu allocated

3. Modify the specification to make sure the container cannot use more than 128Mi and 50m cpu

4. Create the Pod

5. Retrieve the content of the Pod's resources with the jsonpath output

6. Delete the Pod

## Documentation

[https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#example-1](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#example-1)

## Solution

1. Create the specification of a Pod named *mongo* and based on the *mongo:5.0* image

```
kubectl run mongo --image=mongo:5.0 --dry-run=client -o yaml > mongo.yaml
```

2. Modify the specification to make sure the container has 64Mi of RAM and 30m cpu allocated

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mongo
  name: mongo
spec:
  containers:
  - image: mongo:5.0
    name: mongo
    resources:
      requests:
        cpu: 30m
        memory: 64Mi
```

3. Modify the specification to make sure the container cannot use more than 128Mi and 50m cpu

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mongo
  name: mongo
spec:
  containers:
  - image: mongo:5.0
    name: mongo
    resources:
      requests:
        cpu: 30m
        memory: 64Mi
      limits:
        cpu: 50m
        memory: 128Mi
```

4. Create the Pod

```
k apply -f mongo.yaml
```

5. Retrieve the content of the Pod's resources with the jsonpath output

```
k get po/mongo -o jsonpath={.spec.containers[0].resources}
{"limits":{"cpu":"50m","memory":"128Mi"},"requests":{"cpu":"30m","memory":"64Mi"}}
```

6. Delete the Pod

```
k delete -f mongo.yaml
```