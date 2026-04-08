---
title: Labels
---

## Exercise

1. List the labels of all the nodes

2. Add the label *disktype: ssd* on the node worker1

3. Create a Pod named www, with a single container based on the nginx:1.20 image and making sure it is deployed on the node worker1

4. Delete the Pod

5. Delete the *disktype* label from *worker1*

## Documentation

[https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

## Solution

1. List the labels of all the nodes

```
k get nodes --show-labels
```

2. Add the label *disktype: ssd* on the node worker1

```
k label node worker1 disktype=ssd
```

3. Create a Pod named www, with a single container based on the nginx:1.20 image and making sure it is deployed on the node worker1

Create the Pod specification

```
k run nginx --image=nginx:1.20 --dry-run=client -o yaml > pod.yaml
```

Modify the specification to add a scheduling constraint

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx:1.20
    name: nginx
  nodeSelector:
    disktype: ssd
```

Create the Pod

```
k apply -f pod.yaml
```

Verify this Pod has been scheduled on worker1

```
k get po -l run=nginx -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          53s   10.32.0.2   worker1   <none>           <none>
```

4. Delete the Pod

```
k delete po/nginx
```

5. Delete the *disktype* label from *worker1*

```
k label node worker1 disktype-
```


