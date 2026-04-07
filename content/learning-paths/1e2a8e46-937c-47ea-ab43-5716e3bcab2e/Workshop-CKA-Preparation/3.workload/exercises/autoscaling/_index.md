---
title: Autoscaling
---

## Exercise

1. Install metrics server with the *--kubelet-insecure-tls* flag and wait for it to work fine.

Note: it can be installed from [https://luc.run/metrics-server.yaml](https://luc.run/metrics-server.yaml)

2. Create the specification of a Deployment named *www* with a single replica based on nginx

3. Modify the specification adding 100Mi memory and 50m cpu request and create the Deployment.

4. Expose the Pod with a Service named *www*

5. Create a HorizontalPodAutoscaller with a target CPU of 50% and with a minimum of 3 and a maximum of 10 replicas

6. Run a stress Pod (command below) and verify the number of replicas are increased

```
k run ab -ti --rm --restart='Never' --image=lucj/ab -- -n 100000 -c 50 http://www/
```

7. Delete the HPA, the Deployment and the Service

## Documentation

[https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Install metrics server with the *--kubelet-insecure-tls* flag and wait for it to work fine.

Installation:

```
k apply -f https://luc.run/metrics-server.yaml
```

Metrics is running fine when it returns nodes / pods metrics:

```
k top node
NAME      CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master    119m         5%     1242Mi          65%
worker1   33m          1%     962Mi           51%
worker2   48m          2%     987Mi           52%
```

```
k top pod
NAME    CPU(cores)   MEMORY(bytes)
mongo   5m           68Mi
nginx   0m           3Mi
```

2. Create the specification of a Deployment named *www* with a single replica based on nginx

```
k create deploy www --image=nginx:1.20 --dry-run=client -o yaml > deploy.yaml
```

3. Modify the specification adding 100Mi memory and 50m cpu request and create the Deployment.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: www
  name: www
spec:
  replicas: 1
  selector:
    matchLabels:
      app: www
  template:
    metadata:
      labels:
        app: www
    spec:
      containers:
      - image: nginx:1.20
        name: nginx
        resources:
          requests:
            memory: 100Mi
            cpu: 50m
```

Creation of the Deployment

```
k apply -f deploy.yaml
```

4. Expose the Pod with a Service named *www*

```
k expose deploy/www --name=www --port=80
```

5. Create a HorizontalPodAutoscaller with a target CPU of 50% and with a minimum of 3 and a maximum of 10 replicas

```
k autoscale deploy www --min=3 --max=10 --cpu-percent=50
```

6. Run a stress Pod and verify the number of replicas are increased

```
k run ab -ti --rm --restart='Never' --image=lucj/ab -- -n 100000 -c 50 http://www/
```

See the evolution of the number of replicas:

```
k get hpa -w
```

7. Delete the HPA, the Deployment and the Service

```
k delete deploy/www svc/www hpa/www
```

</details>

