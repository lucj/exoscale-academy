---
title: Deployment Strategy
---

## Exercise

1. Create the specification of a deployment named *nginx* with 5 replicas of pods based on the nginx:1.16 image

2. Create the deployment and wait for the pods to be up and running

3. Change the specification so the image is nginx:1.18. Update the resource and observe how the pods are updated using *kubectl get pod -w*

4. Change the deployment strategy to *Recreate* and update the resource

5. Change the specification so the image is nginx:1.20. Update the resource and observe how the pods are updated.

6. Delete the deployment

## Documentation

[https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create the specification of a deployment with 5 replicas of pods based on the nginx:1.16 image

```
k create deployment nginx --image=nginx:1.16 --replicas=5 --dry-run=client -o yaml > deploy.yaml
```

2. Create the deployment and wait for the pods to be up and running

Creation of the deployment:

```
k apply -f deploy.yaml
```

Waiting for the pods to be up and running:

```
k get po -l app=nginx -w
```

3. Change the specification so the image is nginx:1.18. Update the resource and observe how the pods are updated.

Update the specification:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.18
        name: nginx
```

Apply the changes:

```
k apply -f deploy.yaml
```

Observe how the pods are replaced:

```
k get po -l app=nginx -w
```

You should notice the pods are replaced one after the other following a rolling update strategy (default strategy)

4. Change the deployment strategy to *Recreate* and update the resource

You can find information on the deployment rollout strategy with the *explain* command:

```
k explain deploy.spec.strategy
```

Changing the strategy from *RollingUpdate* (default) to *Recreate*

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  strategy:
    type: Recreate
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.18
        name: nginx
```

Apply the changes (this will not trigger the replacement of pods as the pod specification was not modified)

```
k apply -f deploy.yaml
```

5. Change the specification so the image is nginx:1.20. Update the resource and observe how the pods are updated.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  strategy:
    type: Recreate
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.20
        name: nginx
```

Apply the changes:

```
k apply -f deploy.yaml
```

Observe how the pods are replaced:

```
k get po -l app=nginx -w
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6d777db949-9fmnt   1/1     Running   0          10s
nginx-6d777db949-9xdwz   1/1     Running   0          10s
nginx-6d777db949-gnfsg   1/1     Running   0          10s
nginx-6d777db949-qs6k8   1/1     Running   0          10s
nginx-6d777db949-s9zgx   1/1     Running   0          10s
```

You should notice all the pods are terminated at the same time and new ones are created. This is the behavior of the *Recreate* strategy.

6. Delete the deployment

```
k delete deploy nginx
```

</details>

