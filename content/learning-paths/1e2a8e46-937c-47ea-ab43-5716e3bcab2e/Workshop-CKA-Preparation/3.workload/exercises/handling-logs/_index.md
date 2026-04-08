---
title: Handling logs
---

## Exercise

1. Create the specification of a Pod named *www* with a single container based on nginx

2. Modify the specification so the nginx container persists its logs in an ephemeral storage 

Hint: nginx logs are located in */var/log/nginx/access.log* and */var/log/nginx/error.log*

3. Add a sidecar container which reads the logs from the volume every 10 seconds and stream them on its standard output.

You can use the *alpine* image for this sidecar container.

4. Run the pod and make sure both container are running fine. Check the logs of the sidecar container, you should get the logs coming from nginx 

5. Delete the Pod

## Documentation

[https://kubernetes.io/docs/concepts/storage/volumes/#emptydir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

## Solution

1. Create the specification of a Pod named *www* with a single container based on nginx

```
k run www --image=nginx:1.20 --dry-run=client -o yaml > pod.yaml
```

2. Modify the specification so the nginx container persists its logs in an ephemeral storage 

We define an *emptyDir* volume and mount it in */var/log/nginx*:

```
apiVersion: v1
kind: Pod
metadata:
  name: www
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
  volumes:
    - name: logs
      emptyDir: {}
```

3. Add a sidecar container which reads the logs from the volume every 10 seconds and stream them on its standard output

We define another container based on alpine and give it access to the *logs* volume as well. This new container defines an endless loop that reads the logs every 10 seconds and print them.

```
apiVersion: v1
kind: Pod
metadata:
  name: www
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
    - name: alpine
      image: alpine
      command:
      - "/bin/sh"
      - "-c"
      - "while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 10; done"
      volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
  volumes:
    - name: logs
      emptyDir: {}
```

4. Run the pod and make sure both container are running fine

```
k apply -f pod.yaml
```

After a couple of seconds, both containers are running fine

```
k get po
NAME   READY   STATUS    RESTARTS   AGE
www    2/2     Running   0          6s
```

Send a request to the nginx container:

```
k exec www -c nginx -- curl localhost 
```

Check the log of the alpine container:

```
k logs www -c alpine
...
2022/03/29 21:18:26 [notice] 1#1: start worker process 31
2022/03/29 21:18:26 [notice] 1#1: start worker process 32
::1 - - [29/Mar/2022:21:20:28 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.74.0" "-"
2022/03/29 21:18:26 [notice] 1#1: using the "epoll" event method
2022/03/29 21:18:26 [notice] 1#1: nginx/1.21.6
2022/03/29 21:18:26 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/03/29 21:18:26 [notice] 1#1: OS: Linux 5.4.0-105-generic
2022/03/29 21:18:26 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/03/29 21:18:26 [notice] 1#1: start worker processes
2022/03/29 21:18:26 [notice] 1#1: start worker process 31
2022/03/29 21:18:26 [notice] 1#1: start worker process 32
```

This container correctly streams the nginx container's log

5. Delete the Pod

```
k delete -f pod.yaml
```


