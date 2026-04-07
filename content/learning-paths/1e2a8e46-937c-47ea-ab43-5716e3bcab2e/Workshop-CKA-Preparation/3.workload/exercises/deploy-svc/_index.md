---
title: Deployment and Service
---

## Exercise

1. Create a Deployment with 5 replicas of Pods based on the *nginx:1.20-alpine* image

2. Scale the Deployment to 3 replicas

3. Expose the Pods using a Service of type NodePort

4. Get the opened port and try to access the application from a node

5. Delete the Deployment and Service

## Documentation

- [https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create a Deployment with 5 replicas of Pods based on the *nginx:1.20-alpine* image

```
kubectl create deployment nginx --image=nginx:1.20-alpine --replicas=5
```

2. Scale the Deployment to 3 replicas

```
kubectl scale deploy/nginx --replicas=3
```

3. Expose the Pods using a Service of type NodePort

```
kubectl expose deploy/nginx --type=NodePort --port 80
```

4. Get the opened port and try to access the application from a node

```
k get svc nginx
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx   NodePort   10.101.92.175   <none>        80:31641/TCP   15s
```

In this example the opened port is 31641.

From a shell on a node:

```
curl http://localhost:31641
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

5. Delete the Deployment

```
k delete deploy/nginx svc/nginx
```

</details>

