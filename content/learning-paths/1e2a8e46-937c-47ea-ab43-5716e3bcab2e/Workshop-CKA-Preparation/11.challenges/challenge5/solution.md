---
title: Solution - Ch5
---

1. Test

The single replica is not running

```bash
kubectl get deploy,po -n app1
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   0/1     1            0           79m

NAME                         READY   STATUS             RESTARTS   AGE
pod/nginx-7b4b5bbf6f-dth6d   0/1     ImagePullBackOff   0          79m
```

The description of the nginx Pod gives additional information

```bash
kubectl describe po pod/nginx-7b4b5bbf6f-dth6d
...
Events:
  Type    Reason   Age                  From     Message
  ----    ------   ----                 ----     -------
  Normal  BackOff  16s (x351 over 80m)  kubelet  Back-off pulling image "ngnix:1.26"
```

2. Fix it

Edit the Deployment and replace the image name *ngnix:1.26* with *nginx:1.26*, a little typo can have big consequences :)

```bash
kubectl edit deploy/nginx -n app1
```

3. Verification

The Pod is now running fine

```bash
$ kubectl get deploy,po -n app1
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           83m

NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-8f58f5d59-hfg5j   1/1     Running   0          5s
```
