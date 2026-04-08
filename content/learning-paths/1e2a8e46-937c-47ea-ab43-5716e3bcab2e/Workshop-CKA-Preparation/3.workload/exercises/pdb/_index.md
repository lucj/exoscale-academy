---
title: PDB
---

## Exercise

1. Create a deployment named *ghost* with 3 replicas based on the ghost:4 image. Make sure the deployment has the *app: ghost* label

2. Check where the pods are running

3. Create a PodDisruptionBudget named *ghost-pdb* to make sure at least 2 replicas of the ghost deployment are available during disruptions

Note: https://kubernetes.io/docs/tasks/run-application/configure-pdb/

4. Drain the node where 2 pods are running. What do you observe ? Why is the draining possible ?

Note: you can ignore the daemonset pods with the flag *--ignore-daemonsets*

5. Uncordon the node so new Pods can be scheduled on that one

6. Delete the deployment and the PDB

## Documentation

[https://kubernetes.io/docs/tasks/run-application/configure-pdb/](https://kubernetes.io/docs/tasks/run-application/configure-pdb/)

## Solution

1. Create a deployment named *ghost* with 4 replicas based on the ghost:4 image. Make sure the deployment has the *app: ghost* label

```
k create deployment ghost --image=ghost:4 --replicas=3
```

Note: the *app: ghost* label is automatically set using this imperative command

2. Check where the pods are running

```
$ k get po -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE      NOMINATED NODE   READINESS GATES
ghost-5d77b859d5-64cr9   1/1     Running   0          13s   10.32.128.2   worker2   <none>           <none>
ghost-5d77b859d5-b24qp   1/1     Running   0          13s   10.32.0.3     worker1   <none>           <none>
ghost-5d77b859d5-rqvvl   1/1     Running   0          13s   10.32.0.2     worker1   <none>           <none>
```

Out of the 3 pods, 2 should run on the same node

3. Create a PodDisruptionBudget to make sure at least 2 replicas of the ghost deployment are available during disruptions

```
k create pdb ghost-pdb --selector=app=ghost --min-available=2
```

4. Drain the node where 2 pods are running. What do you observe ? Why is the draining possible ?

When draining the node (= asking Kubernetes to move the pods to the other node) you will first get an error as this would violate the policy defined in the PodDisruptionBudget. 

```
$ k drain worker1 --ignore-daemonsets
node/worker1 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-7ktdc, kube-system/weave-net-ps4tc
evicting pod default/ghost-5d77b859d5-rqvvl
evicting pod default/ghost-5d77b859d5-b24qp
error when evicting pods/"ghost-5d77b859d5-rqvvl" -n "default" (will retry after 5s): Cannot evict pod as it would violate the pod's disruption budget.
pod/ghost-5d77b859d5-b24qp evicted
evicting pod default/ghost-5d77b859d5-rqvvl
pod/ghost-5d77b859d5-rqvvl evicted
node/worker1 drained
```

A couple of seconds later the draining will be possible once the first pod has been terminated and a new one has been scheduled on the other worker. In that case the minimum number of replicas is repected.

5. Uncordon the node so new Pods can be scheduled on that one

Draining a node also *cordon* it so new Pod cannot be scheduled on that one anymore:

```
$ k get no
NAME      STATUS                     ROLES           AGE     VERSION
master    Ready                      control-plane   6h1m    v1.25.7
worker1   Ready,SchedulingDisabled   <none>          5h54m   v1.25.7
worker2   Ready                      <none>          5h52m   v1.25.7
```

By "uncordoning" the node we can make it "schedulable" again:

```
$ k uncordon worker1
node/worker1 uncordoned
```

The status of worker1 does not show *SchedulingDisabled* anymore:

```
root@luc:~/workload# k get node
NAME      STATUS   ROLES           AGE     VERSION
master    Ready    control-plane   6h2m    v1.25.7
worker1   Ready    <none>          5h54m   v1.25.7
worker2   Ready    <none>          5h53m   v1.25.7
```

6. Delete the deployment and the PDB

```
k delete deploy/ghost pdb/ghost-pdb
```


