---
title: Eviction
---

## Exercise

1. Run a Deployment named *www* with 6 replicas of Pod based on nginx:1.20

2. Where are the Pods running ?

3. Using a single command, make sure the Pods are evicted from worker2

4. Delete the Deployment and revert your changes back

## Documentation

[https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Run a Deployment with 6 replicas of Pod based on nginx:1.20

```
k create deploy www --image=nginx:1.20 --replicas=6
```

2. Where are the Pods running ?

Pods are running on worker1 and worker2 nodes (as master node as a NoSchedule taint preventing application pods from been scheduled on that node).

```
k get po -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
www-644dfdf68b-2b42t   1/1     Running   0          3s    10.32.0.3   worker1   <none>           <none>
www-644dfdf68b-6vxmz   1/1     Running   0          3s    10.38.0.2   worker2   <none>           <none>
www-644dfdf68b-jrrd7   1/1     Running   0          3s    10.38.0.1   worker2   <none>           <none>
www-644dfdf68b-kgrc6   1/1     Running   0          3s    10.38.0.5   worker2   <none>           <none>
www-644dfdf68b-lx2sp   1/1     Running   0          3s    10.32.0.5   worker1   <none>           <none>
www-644dfdf68b-mvsfv   1/1     Running   0          3s    10.32.0.4   worker1   <none>           <none>
```

3. Using a single command, make sure the Pods are evicted from worker2

Adding a taint with the *NoExecute* effect on a node will evict all the Pods that do not tolerate that taint

```
k taint node worker2 app=blue:NoExecute
```

The Pods now all run on worker1

```
k get po -o wide
www-644dfdf68b-2b42t   1/1     Running   0          98s   10.32.0.3    worker1   <none>           <none>
www-644dfdf68b-5wmfs   1/1     Running   0          15s   10.32.0.8    worker1   <none>           <none>
www-644dfdf68b-fbf2v   1/1     Running   0          15s   10.32.0.10   worker1   <none>           <none>
www-644dfdf68b-lx2sp   1/1     Running   0          98s   10.32.0.5    worker1   <none>           <none>
www-644dfdf68b-mvsfv   1/1     Running   0          98s   10.32.0.4    worker1   <none>           <none>
www-644dfdf68b-p8xg2   1/1     Running   0          15s   10.32.0.9    worker1   <none>           <none>
```

4. Delete the Deployment and revert your changes back

Deletion of the Deployment

```
k delete deploy www
```

Remove the NoExecute taint from worker2

```
k taint node worker2 app-
```
</details>

