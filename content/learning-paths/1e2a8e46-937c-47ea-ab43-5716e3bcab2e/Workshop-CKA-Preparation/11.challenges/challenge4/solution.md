---
title: Solution
---

1. Test

List the resources in *app2*

```bash
$ kubectl get po,svc -n app2
NAME        READY   STATUS    RESTARTS   AGE
pod/nginx   1/1     Running   0          8s

NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/nginx   NodePort   10.106.164.238   <none>        80:30000/TCP   8s
```

From a worker node, try to reach the nginx Pod

```bash
$ curl localhost:30000
curl: (7) Failed to connect to localhost port 30000: Connection refused
```

2. Check the service

```bash
kubectl describe svc nginx -n app2
Name:                     nginx
Namespace:                app2
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.106.164.238
IPs:                      10.106.164.238
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30000/TCP
Endpoints:                <none>
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

The list of *Endpoints* is empty, that is surely a problem. The Service's selector is *app: nginx*, let's see the content of the Pod's labels.

```bash
kubectl get po nginx -o jsonpath={.metadata.labels}
{"run":"nginx"}
```

The Pod's label does not match the Service selector.

3. Fix it

One way to fix it is to change the Service's selector to *run: nginx* (the other way is to change the Pod's label)

```bash
kubectl edit service/nginx -n app2
```

4. Verification

Calling the service from a node through the NodePort should now work:

```bash
$ curl localhost:30000
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
