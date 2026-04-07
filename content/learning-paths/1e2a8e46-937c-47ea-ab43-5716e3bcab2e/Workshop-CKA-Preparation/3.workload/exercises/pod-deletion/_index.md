---
title: Pod deletion
---

## Exercise

1. Create a Pod named *debug* with one container based on *alpine:3.15* executing the command `sleep 10000`. Make sure the Pod is running fine

2. Measure the time needed to delete the Pod. How to do explain that ?

3. Which command could be used to delete the Pod right away ?

## Documentation

[https://kubernetes.io/docs/reference/kubectl/generated/kubectl_delete/](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_delete/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create a Pod named *debug* with one container based on *alpine:3.15* executing the command `sleep 10000`. Make sure the Pod is running fine

Creation of the Pod:

```
k run debug --image=alpine:3.15 --command sleep 10000
```

Making sure the Pod is running:

```
k get po debug
NAME    READY   STATUS    RESTARTS   AGE
debug   1/1     Running   0          4s
```

2. Measure the time needed to delete the Pod. How to do explain that ?

It takes around 30 seconds for the pod to be deleted:

```
time k delete po debug
pod "debug" deleted

real	0m30.682s  <- you've been waiting more than 30 seconds for the pod to the deleted
user	0m0.096s
sys	  0m0.037s
```

There are some cases where the sigterm signal is not forwarded to the container. In that case, a sigkill is sent after 30 seconds.

3. Which command could be used to delete the Pod right away ?

You can force the Pod deletion with the following options:

```
k delete po debug --force --grace-period=0
```

</details>

