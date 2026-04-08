---
title: Solution
---

1. Create a Deployment with 6 replicas of Pods based on the nginx:1.20 image

```
k create deploy www --image=nginx:1.20 --replicas=4
```

2. Where are the pods scheduled ?

The pods are splitted between worker1 and worker2 (because master node has a NoScheduled taint that the pods do not tolerate)

```
k get po -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
www-644dfdf68b-6mk2w   1/1     Running   0          33s   10.38.0.1   worker2   <none>           <none>
www-644dfdf68b-crdjl   1/1     Running   0          33s   10.32.0.6   worker1   <none>           <none>
www-644dfdf68b-nhm7b   1/1     Running   0          33s   10.32.0.2   worker1   <none>           <none>
www-644dfdf68b-tsw84   1/1     Running   0          33s   10.38.0.5   worker2   <none>           <none>
```

3. Drain the worker1 node. What happened ?

```
k drain worker1 --ignore-daemonsets
```

The application pods have been evicted from worker1 and are now all running on worker2

```
k get po -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
www-644dfdf68b-6mk2w   1/1     Running   0          56s   10.38.0.1   worker2   <none>           <none>
www-644dfdf68b-slvk9   1/1     Running   0          18s   10.38.0.6   worker2   <none>           <none>
www-644dfdf68b-tsw84   1/1     Running   0          57m   10.38.0.5   worker2   <none>           <none>
www-644dfdf68b-xrhhm   1/1     Running   0          9ss   10.38.0.2   worker2   <none>           <none>
```

4. Uncordon worker1

Undordoning a node makes it possible for new Pods to be scheduled on that node

```
k uncordon worker1
```

5. What happened for the pods belonging to the www Deployment ?

Nothing changed regarding the pods belonging to the www Deployment:

6. Force the re-creation of the pods

Restarting the Deployment

```
k rollout restart deploy/www
```

Forcing the Deployment to be restarted terminates the running Pods and create new ones

```
k get po -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
www-b44d96857-bpxr8   1/1     Running   0          5s    10.38.0.5   worker2   <none>           <none>
www-b44d96857-lwx65   1/1     Running   0          5s    10.32.0.3   worker1   <none>           <none>
www-b44d96857-rdv69   1/1     Running   0          6s    10.38.0.2   worker2   <none>           <none>
www-b44d96857-xbssr   1/1     Running   0          6s    10.32.0.2   worker1   <none>           <none>
```

Pods are now distributed between the worker1 and worker2 nodes

7. Delete the Deployment

```
k delete deploy www
```