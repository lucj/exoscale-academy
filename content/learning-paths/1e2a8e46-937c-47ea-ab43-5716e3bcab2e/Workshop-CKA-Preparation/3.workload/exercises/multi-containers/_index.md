---
title: multi-containers Pod
---

## Exercise

1. Create the specification of a Pod named *www* with one container based on *nginx:1.20-alpine*

2. Add a container based on the image *httpd:2.4* and run the Pod

3. Why this Pod will not reach the *Running* status ?

4. Delete the Pod

## Documentation

[https://kubernetes.io/docs/concepts/workloads/pods/](https://kubernetes.io/docs/concepts/workloads/pods/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create the specification of a Pod with one container based on *nginx:1.20-alpine*

```
k run www --image=nginx:1.20-alpine --dry-run=client -o yaml > pod.yaml
```

2. Add a container based on the image *httpd:2.4* and run the Pod

Adding a second container in the list:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: www
  name: www
spec:
  containers:
  - image: nginx:1.20-alpine
    name: nginx
  - image: httpd:2.4
    name: apache
```

Creation of the Pod

```
k apply -f pod.yaml
```

Get the status of the Pod

```
k get po -l run=www
NAME   READY   STATUS             RESTARTS     AGE
www    1/2     CrashLoopBackOff   1 (5s ago)   11s
```

3. Why this Pod will not reach the *Running* status ?

Containers of a same pod cannot listen on the same port (80 in this exemple)

The logs of the nginx container indicates it started correctly:

```
k logs www -c nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/03/28 14:49:03 [notice] 1#1: using the "epoll" event method
2022/03/28 14:49:03 [notice] 1#1: nginx/1.20.2
2022/03/28 14:49:03 [notice] 1#1: built by gcc 10.3.1 20210424 (Alpine 10.3.1_git20210424)
2022/03/28 14:49:03 [notice] 1#1: OS: Linux 5.4.0-105-generic
2022/03/28 14:49:03 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/03/28 14:49:03 [notice] 1#1: start worker processes
2022/03/28 14:49:03 [notice] 1#1: start worker process 33
2022/03/28 14:49:03 [notice] 1#1: start worker process 34
```

The logs of the apache container indicates it cannot start as the port 80 is already used:

```
k logs www -c  apache
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.32.0.6. Set the 'ServerName' directive globally to suppress this message
(98)Address already in use: AH00072: make_sock: could not bind to address [::]:80
(98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:80
no listening sockets available, shutting down
AH00015: Unable to open logs
```

4. Delete the Pod

```
k delete po/www
```

</details>

