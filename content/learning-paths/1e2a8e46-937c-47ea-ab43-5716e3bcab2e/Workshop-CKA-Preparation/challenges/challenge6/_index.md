---
weight: 6
title: "Challenge 6"
---

Use the following command to break something in the cluster

```bash
curl -s https://learn.exoscale.dev/cka/ch/set6.sh  | bash
```

It does not seem possible to create a simple pod in the cluster. Understand the problem is and fix it.

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Test

Try to create a pod:

```bash
kubectl run ghost --image=ghost:4
```

Verify it is running:

```bash
$ kubectl get po
NAME    READY   STATUS    RESTARTS   AGE
ghost   0/1     Pending   0          10s
```

There is definitely something wrong as the pod remains in pending status.

```bash
$ kubectl describe po ghost
...
Events:
  Type     Reason            Age               From               Message
  ----     ------            ----              ----               -------
  Warning  FailedScheduling  4s (x2 over 72s)  default-scheduler  0/3 nodes are available: 3 node(s) had taint {app: secure}, that the pod didn't tolerate.
```

It seems the 3 nodes have a taint that prevents the pod from being scheduled. We can see there are 2 taints on the master:
- app=secure:NoSchedule
- node-role.kubernetes.io/master:NoSchedule (default one when using kubeadm)

```bash
$ kubectl get no master -o jsonpath={.spec.taints} | jq
[
  {
    "effect": "NoSchedule",
    "key": "app",
    "value": "secure"
  },
  {
    "effect": "NoSchedule",
    "key": "node-role.kubernetes.io/master"
  }
]
```

Only the *app=secure:NoSchedule* taint exists on both worker nodes:

```bash
# worker1
$ kubectl get no worker1 -o jsonpath={.spec.taints} | jq
[
  {
    "effect": "NoSchedule",
    "key": "app",
    "value": "secure"
  }
]

$ kubectl get no worker2 -o jsonpath={.spec.taints} | jq
# worker2
[
  {
    "effect": "NoSchedule",
    "key": "app",
    "value": "secure"
  }
]
```

2. Fix it

Let's delete the ghost pod and create a new one with a toleration for the *app=secure:NoSchedule* taint

```bash
kubectl delete po/ghost
```

Creation of a pod specification

```bash
kubectl run ghost --image=ghost:4 --dry-run=client -o yaml > pod.yaml
```

Adding the toleration:

```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: ghost
  name: ghost
spec:
  tolerations:
  - key: app
    value: secure
    effect: NoSchedule
  containers:
  - image: ghost:4
    name: ghost
```

Creation of the new pod:

```bash
kubectl apply -f pod.yaml
```

The pod is now running fine:

```bash
$ kubectl get po
NAME    READY   STATUS    RESTARTS   AGE
ghost   1/1     Running   0          2s
```

</details>