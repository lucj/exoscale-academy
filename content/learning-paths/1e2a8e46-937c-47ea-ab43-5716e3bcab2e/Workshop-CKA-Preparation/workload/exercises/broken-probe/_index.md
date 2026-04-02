## Exercise

1. Create the specification of a Deployment named nginx, with a single replica based on the *nginx:1.20-alpine* image

2. Add a liveness probe which checks every 5 seconds the */* endpoint on port 8080

3. Create the Deployment. What do you notice ?

4. Fix the probe so it checks on port 80

5. Delete the Deployment

## Documentation

[https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

<br>

<details>
  <summary markdown="span">Solution</summary>

1. Create the specification of a Deployment with a single replica based on the *nginx:1.20-alpine* image

```
kubectl create deploy nginx --image=nginx:1.20-alpine --dry-run=client -o yaml > deploy.yaml
```

2. Add a liveness probe which checks every 5 seconds the */* endpoint on port 8080

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.20-alpine
        name: nginx
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          periodSeconds: 5
```

3. Create the Deployment. What do you notice ?

The nginx container is restarted many times.

```
k get po
NAME                     READY   STATUS             RESTARTS      AGE
nginx-65bbc7db4d-664xr   0/1     CrashLoopBackOff   6 (27s ago)   4m12s
```

Describing the pod gives additional information which indicates the liveness probe keeps on failing:

```
k describe po nginx-65bbc7db4d-664xr
...
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  26s               default-scheduler  Successfully assigned default/nginx-65bbc7db4d-664xr to worker2
  Normal   Pulling    25s               kubelet            Pulling image "nginx:1.20-alpine"
  Normal   Pulled     21s               kubelet            Successfully pulled image "nginx:1.20-alpine" in 3.80023247s
  Normal   Created    6s (x2 over 21s)  kubelet            Created container nginx
  Normal   Killing    6s                kubelet            Container nginx failed liveness probe, will be restarted
  Normal   Pulled     6s                kubelet            Container image "nginx:1.20-alpine" already present on machine
  Normal   Started    5s (x2 over 21s)  kubelet            Started container nginx
  Warning  Unhealthy  1s (x4 over 16s)  kubelet            Liveness probe failed: Get "http://10.32.128.5:8080/": dial tcp 10.32.128.5:8080: connect: connection refused
```

4. Fix the probe so it checks on port 80

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.20-alpine
        name: nginx
        livenessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 5
```

5. Delete the Deployment

```
k delete deploy/nginx
```

</details>

