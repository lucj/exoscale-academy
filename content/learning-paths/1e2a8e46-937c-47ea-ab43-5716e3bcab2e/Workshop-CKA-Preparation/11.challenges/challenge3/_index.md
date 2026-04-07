---
weight: 3
title: "Challenge 3"
---

Use the following command to break something in the cluster

```bash
curl -s https://learn.exoscale.dev/cka/ch/set3.sh  | bash
```

It seems not possible to reach any services in the cluster. Understand the problem is and fix it.

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Test

Let's create a new Pod and expose it with a ClusterIP Service

```bash
kubectl run nginx --image=nginx:1.20 --port 80 --expose
```

Let's new get an interactive shell in an Alpine based Pod

```bash
kubectl run -ti --rm debug --image=alpine -- sh
```

We cannot reach the nginx Pod from the debug one:

```bash
/ # wget http://nginx
wget: bad address 'nginx'
```

Going one step further we can see the name resolution is not working fine:

```bash
/ # nslookup nginx
;; connection timed out; no servers could be reached
```

2. Checking the cluster's internal DNS

```bash
kubectl get deploy -n kube-system
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
coredns          0/0     0            0           2d17h
```

Seems like there is no replica running for the dns (should be 2 by default)

3. Fixing the thing

Let's increase the number of replicas to 2

```bash
kubectl -n kube-system scale deploy/coredns --replicas=2
```

4. Verification

The name resolution is now working:

```bash
Server:		10.96.0.10
Address:	10.96.0.10:53
Name:	nginx.default.svc.cluster.local
Address: 10.96.188.129
...
```

From a new debug Pod the nginx Pod can be reached:

```bash
/ # wget -O- nginx
Connecting to nginx (10.96.188.129:80)
writing to stdout
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

</details>